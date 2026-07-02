# ADR-052: Seed default categories on account creation, with a guarded backfill

**Status:** Accepted
**Date:** 2026-07-01
**Deciders:** Mert Ertugrul

## Context

[ADR-048](048-categories-client-model-and-live-synced-management.md) established categories as
server-owned domain data that **start empty** — a new user opens Manage Categories and the slice
dialog to a blank list. User testing of categories + gap-fill suggestions (BE #335) surfaced this as
a cold start: with nothing to tag slices with on day one, the feature reads as broken rather than
optional. We want every account to begin with a meaningful, fully-editable set of categories.

Two questions shape the decision:

- **Where does the seed run?** A client-side seed would race across a user's devices and reinstalls
  (each fresh client would try to create the same set), and would spread "what the defaults are" across
  every platform. The set should be consistent across devices from the first sync.
- **How do we treat existing accounts?** Accounts created before this change have an empty list. We
  want to backfill them — but only the ones that *never had* categories, not the ones that created some
  and then deliberately **deleted them all**. Nothing in the schema distinguished those two states:
  categories shipped only days earlier ([#325](../), #326) and the `audit_log` does not record category
  mutations, so history cannot tell them apart.

## Decision

Seed a fixed default catalog **server-side, on account creation**, and backfill existing empty accounts
**once**, gated by a new per-user marker. Key choices:

- **One in-code catalog, derived from the front-end gap-fill labels.** `DefaultCategoryCatalog.SEEDS`
  is an ordered list of 15 `(name, #RRGGBB)` pairs taken from the FE `TemplateLibrary` +
  `SuggestionCatalog` ([ADR-045](045-gap-fill-activity-suggestions.md),
  [ADR-051](051-app-curated-in-code-day-templates.md)), first-occurrence-wins. It is the single source
  of truth for both seeding and backfill.
- **Seed inside account provisioning.** `UserService.autoProvisionUser` flushes the new user, then
  calls `DefaultCategorySeedingService.seedFor(userId)` in the same transaction, so a new account and
  its categories commit atomically. Seeding failure rolls back provisioning and the user retries
  cleanly rather than ending up flagged-but-empty.
- **Defaults are ordinary categories.** They are normal `Category` rows (id, name, `#RRGGBB`, timestamps),
  fully renamable / recolorable / deletable, and they emit the usual `CATEGORY_CREATED` STOMP events
  (ADR-048) so any already-open client converges without special-casing.
- **A per-user `default_categories_seeded_at` marker is the guard.** `NULL` means "seeding has never
  run for this user". It is set the moment we seed (or, in backfill, the moment we decide not to). This
  is what tells a never-seeded account apart from one that cleared its list: once the marker is set,
  seeding never runs for that user again.
- **Idempotent by case-insensitive name.** Seeding fetches the user's existing category names and only
  inserts catalog entries that are missing, so re-running never creates duplicates and respects the
  existing name-uniqueness / `409` semantics.
- **One-off backfill via a guarded startup runner.** `DefaultCategoryBackfillRunner` (an
  `ApplicationRunner`, `slicefocus.categories.backfill.enabled`, on by default, off in tests) processes
  every user whose marker is `NULL`, one transaction each so a single failure cannot abort the batch:
  a user with **zero** categories is seeded; a user who **already has** categories is marked seeded
  **without** inserting (they "had some" — we must not repopulate a list they may have curated). After
  one successful pass every user carries the marker, so subsequent boots find no candidates and do
  nothing.

## Alternatives Considered

- **Seed on the client (FE).** Rejected: multiple devices and reinstalls would double-seed or race, and
  the definition of "the defaults" would live in every client instead of one server list. Server seeding
  makes the set consistent across devices and lets the FE stay a pure renderer of what the server returns.
- **Backfill as a pure SQL Flyway migration.** Rejected: it cannot emit the `CATEGORY_CREATED` STOMP
  events open clients rely on to converge, would duplicate the 15-entry catalog (and colours) in SQL as a
  second source of truth, and would need separate handling for H2 (tests) vs the Postgres vendor path. The
  Java runner reuses the one catalog and the normal event path.
- **Infer never-had-vs-deleted-all from history.** Rejected: `audit_log` does not record category
  create/delete, so there is no historical signal. An explicit per-user marker is both simpler and exact
  going forward.
- **Seed lazily on first category list.** Rejected: the list endpoint is read-only, so seeding there
  fights transaction flush semantics and reintroduces the multi-device seed race that server-side,
  provision-time seeding avoids.

## Consequences

- New accounts open Manage Categories to 15 sensible, editable categories instead of a blank list;
  ADR-048's "categories start empty" premise is **amended** — they now start seeded, but remain ordinary,
  deletable categories, so a user can still curate down to none.
- The backfill is effectively idempotent and self-terminating: after the first pass the `NULL`-marker
  query is empty and the runner is a no-op on every boot.
- **One-time backfill caveat:** because the marker did not exist before this change, a pre-existing user
  who deliberately deleted *all* their categories in the few days between the categories launch and this
  backfill is indistinguishable from a never-seeded user and will be re-seeded **once**. After that their
  marker is set and they are never re-seeded. Given the tiny window this is an acceptable one-off; FE #371
  (empty-state "create a category" prompt) remains the fallback for anyone who clears the list afterward.
- The FE gap-fill catalog and the backend seed are two copies of the same 15 labels for now; keeping them
  in lockstep is manual. Making the FE derive from the seeded categories is possible later and out of scope
  here.
- Emoji is intentionally not seeded: the `Category` model has no emoji field, so defaults carry name +
  colour only (revisit if the model gains one).

## Amendment (2026-07-02): user-triggered restore as a non-destructive merge

The backfill's "already has categories → mark seeded without inserting" rule (correct as a default)
left one gap: accounts that predate seeding, and anyone who later deletes defaults, had no way to get
the default set back. BE #337 adds `POST /api/v1/categories/restore-defaults`, a user-triggered
**merge, not a reset**:

- Reuses `DefaultCategorySeedingService.seedFor(userId)` — the same single-catalog, idempotent,
  case-insensitive-by-name insert used at provisioning — so only missing catalog entries are created.
  Existing categories are never modified or deleted, and the `default_categories_seeded_at` marker is
  irrelevant to (and untouched by) the endpoint.
- `seedFor` now returns the created `Category` rows (previously a count) so the endpoint can report
  exactly what was added; the response is the list of created categories, empty when all defaults are
  already present (re-invoking is a no-op). Created rows emit the usual `CATEGORY_CREATED` STOMP events.
- A destructive **reset** (delete-all + reseed) was considered and rejected: it would orphan slice→category
  tags, recreate renamed defaults as duplicates, and require a heavyweight confirmation. The merge variant
  needs only mild FE copy ("adds any missing default categories; your existing ones are kept" — FE #377).

*Related: BE #335, BE #337, FE #377, [ADR-048](048-categories-client-model-and-live-synced-management.md) (amended),
[ADR-045](045-gap-fill-activity-suggestions.md), [ADR-051](051-app-curated-in-code-day-templates.md),
[ADR-007](007-websocket-stomp-realtime.md)*
