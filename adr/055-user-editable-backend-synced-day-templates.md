# ADR-055: User-editable, backend-synced day templates

**Status:** Accepted
**Date:** 2026-07-05
**Deciders:** Mert Ertugrul

## Context

[ADR-051](051-app-curated-in-code-day-templates.md) shipped day templates as a hardcoded, app-curated
library of five `const DayTemplate`s. That solved the cold-start problem but leaves no way for a user
to create their own routine, tweak a starter template, or capture a day they already built. The ask
(docs #9, BE #356, FE #418/#419/#420) is full user templates: create, edit, delete, duplicate a
starter, and "save current day as template" — synced across devices.

Two prior stacks constrain the design: the categories feature
([ADR-048](048-categories-client-model-and-live-synced-management.md)) established the
client-model + cache + live-sync pattern, and the typed-envelope offline queue
([ADR-049](049-typed-envelope-action-queue.md)) established client-UUID + idempotent `PUT` replay.
The suggestion catalog ([ADR-045](045-gap-fill-activity-suggestions.md)) is derived from the curated
template library, so any change here risks disturbing that coupling.

## Decision

Add **user-owned day templates** as a first-class backend resource at `/api/v1/day-templates`,
mirroring the categories stack end to end, while the five curated templates stay exactly as ADR-051
defined them: read-only, in-code, `const`.

- **Duplicate-then-edit, not edit-in-place.** Curated templates cannot be modified or deleted.
  "Editing" one duplicates it into a user template ("Maker Schedule (copy)") that is then fully
  editable. No override/shadowing layer, no "reset to default" machinery, and the curated library
  remains a compile-time constant.
- **Disjoint id spaces instead of an `isCurated` flag.** Curated ids are non-UUID strings
  (`maker_schedule`); user template ids are client-generated UUIDs, and the API path parameter is
  uuid-typed. `TemplateLibrary.isCurated(id)` answers provenance; the `DayTemplate` value object
  stays free of persistence concerns, and collision is impossible by construction.
- **Categories-style sync, verbatim.** Client UUID + idempotent `PUT` upsert (creation happens via
  `PUT`, so offline replay needs no server-id remap), SharedPreferences seed-then-refresh cache,
  optimistic mutations with rollback, ActionQueue entity `day_template`, and an id-only STOMP topic
  `/topic/user/{userId}/day-templates` that triggers a refetch.
- **Parent + child tables, not JSONB.** `day_templates` → `day_template_slices` (label, minute range,
  color, position) → `day_template_slice_categories`. The BE test suite runs Flyway on H2, which JSONB
  would break, and real FKs give integrity plus `ON DELETE CASCADE` cleanup when a category is
  deleted.
- **Template slices carry `categoryIds`; the backend drops unknown ids silently.** Tags are what make
  "save day as template" round-trip focus attribution. Strict 400 validation would poison the offline
  queue: a queued upsert referencing a since-deleted category would fail on every replay, forever.
  Lenient dropping self-heals; the FE additionally sanitizes category ids when *applying* a template,
  because the slice API (unchanged) still validates strictly.
- **No name uniqueness — a deliberate divergence from categories.** Duplicate-then-edit legitimately
  produces colliding "(copy)" names, and dropping the constraint removes the entire 409
  conflict-replay path from queue handling.
- **Slice semantics match `DaySlice`.** Minutes 0–1439, `start != end`, midnight wrap allowed
  (end = 0), circular non-overlap enforced server-side — a template that can never be applied is
  invalid.

## Alternatives Considered

- **Local-only storage (SharedPreferences, like timer preferences).** Far smaller, FE-only — but no
  cross-device sync, and templates are durable user content, not device preferences. Rejected by
  product decision.
- **Edit curated templates in place via a local override layer.** More "direct" UX, but requires
  shadow/merge/reset logic and makes the curated library mutable state. Rejected for complexity.
- **`isCurated` field on `DayTemplate`.** Pollutes the value object with provenance and invites
  invalid states (a synced template claiming to be curated). Disjoint id spaces encode the same fact
  structurally. Rejected.
- **JSONB slices column.** One table, flexible — but not H2-portable (breaking the BE test setup) and
  no FK from slice tags to categories, so deleted-category cleanup becomes manual. Rejected.
- **Strict category validation on template upsert.** Symmetric with the slice API, but poisons offline
  replay (see above). Rejected.

## Consequences

- Users can create, edit, delete, and duplicate templates on any device and see them everywhere;
  "save day as template" captures an existing schedule, tags included.
- The curated starter library keeps its ADR-051 guarantees (const, translatable later, no fetch), and
  the suggestion catalog **remains derived from curated templates only** — user templates do not feed
  gap-fill suggestions in v1. Making the catalog dynamic is a known follow-up.
- The apply-template flow in the FE must sanitize slices (strip ids, filter category ids against the
  live category list) before writing through the slice repository, since the slice API stays strict.
- A second divergence point from categories (no 409) must be remembered when reasoning about queue
  replay: template replay is pure idempotent upsert with no conflict channel.
- Amends [ADR-051](051-app-curated-in-code-day-templates.md) (curated library unchanged, but
  "templates are purely in-code, no backend" no longer holds globally) and extends
  [ADR-048](048-categories-client-model-and-live-synced-management.md)/
  [ADR-049](049-typed-envelope-action-queue.md) to a new entity.
