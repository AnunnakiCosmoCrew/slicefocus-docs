# ADR-048: Categories — client model, slice tagging, cache & live-synced management

**Status:** Accepted (amended by [ADR-052](052-seed-default-categories-on-account-creation.md))
**Date:** 2026-07-01
**Deciders:** Mert Ertugrul

> **Amended by [ADR-052](052-seed-default-categories-on-account-creation.md) (BE #335):** categories no
> longer start empty — a fixed default set is seeded server-side on account creation (and backfilled for
> existing empty accounts). They remain ordinary, fully-editable, deletable categories, so a user can
> still curate down to none.

## Context

Categories (FE #335, #336, #337) let a user define reusable, coloured labels and tag slices
with them. The feature spans three concerns that each need a deliberate client-side design:

- **How a slice references categories.** A slice can carry several categories (many-to-many), and
  slices created before the feature must keep loading.
- **How the category list is stored and shown.** Users open the manage-categories screen and the
  slice dialog constantly; a blank spinner on every open would feel broken, but a stale list that
  never refreshes would drift from the backend.
- **How edits stay consistent across screens and devices.** Renaming a category in the Account
  screen must be reflected in an open slice dialog, and on the user's other devices, without a
  manual refresh — the same cross-device expectation that motivated real-time session sync
  ([ADR-007](007-websocket-stomp-realtime.md)).

Unlike theme/timer preferences ([ADR-032](032-server-synced-preferences.md)), categories are
**shared, server-owned domain data** with their own identity and uniqueness rules, so they are
fetched and mutated through a REST repository, not folded into the preferences blob.

## Decision

Model categories as pure-Dart domain objects, tag slices by id list, cache the list locally with a
seed-then-refresh policy, and keep every open surface live via a thin STOMP broadcast that triggers a
refetch. Key design choices:

- **Pure-Dart `Category` model with ARGB-int colour.** `Category` (`categories/domain/category.dart`)
  holds `id`, `name`, `colorValue` (int, e.g. `0xFF43AA8B`) and `updatedAt`, with no `dart:ui`
  dependency — mirroring `DaySlice`. The backend exchanges colour as `#RRGGBB`; `hexFromColorValue` /
  `colorValueFromHex` convert at the I/O boundary and fall back to a default colour on malformed input.
- **Tagging by id list on the slice, not a join model.** `DaySlice` gains
  `List<String> categoryIds` (default empty). Parsing is tolerant (`whereType<String>()`), so slices
  authored before the feature — and any malformed payload — load with an empty tag set rather than
  throwing. The relational join lives on the backend; the client keeps a flat id list because that is
  all the UI needs.
- **Single-key cache, seed then refresh.** `SharedPreferencesCategoryCache` stores the whole list as
  JSON under one key (`categories_cache`, [ADR-029](029-shared-preferences-persistence.md)). A warm
  cache renders instantly while a background fetch refreshes it; a cold cache blocks on the network so
  the user sees a spinner rather than a false "no categories" empty state. Corrupt data self-heals to
  an empty list.
- **Client-generated ids + idempotent PUT-upsert.** Online creates use `POST` (server assigns the id),
  but the offline/replay path generates a UUID client-side and writes via `PUT /api/v1/categories/{id}`,
  which the backend treats as an idempotent upsert. One code path — and one replayed action — serves
  create, update, and reconnect without any server-id remapping (see
  [ADR-049](049-typed-envelope-action-queue.md)). A duplicate name returns `409` →
  `CategoryNameConflictException`, surfaced inline to the user.
- **Optimistic mutations with rollback.** `categoriesProvider` is an `AsyncNotifierProvider`
  ([ADR-008](008-riverpod-state-management.md)) that applies create/update/delete to state and cache
  immediately, delegates to the sync service, and rolls state + cache back to the previous snapshot if
  the write ultimately fails.
- **Thin STOMP event, full refetch.** The backend publishes id-only events
  (`CATEGORY_CREATED/UPDATED/DELETED`) to `/topic/user/{userId}/categories`. The client does not patch
  from the payload; it invalidates `categoriesProvider` and refetches, guaranteeing convergence with
  the server — the same "broadcast is a hint, the fetch is the truth" pattern already used for session
  and slice events.
- **Session-scoped shared instances.** The repository, cache, and sync service are created once per
  authenticated session and injected via `ProviderScope` overrides, and pushed routes inherit that
  scope, so an edit on one screen is visible to every other screen backed by the same container.

## Alternatives Considered

- **Store full category objects on each slice.** Denormalized and simple to render, but every rename
  would have to rewrite every tagged slice, and stale copies would diverge. An id list keeps one source
  of truth.
- **A dedicated client-side join model (slice_categories).** Faithful to the relational shape but pure
  overhead on the client, which never queries categories independently of the slice that owns them.
- **Per-category cache keys.** Enables granular invalidation, but the list is small and always shown in
  full; one key is simpler and the whole-list refetch on any change keeps it trivially consistent.
- **Patch UI state from the STOMP event payload.** Avoids a refetch, but forces the event to carry the
  full category and re-implements merge/ordering on the client. An id-only event plus refetch is smaller
  on the wire and cannot drift from the backend.
- **Fold categories into server-synced preferences ([ADR-032](032-server-synced-preferences.md)).**
  Wrong fit: categories are first-class entities with ids and a uniqueness constraint, not scalar
  settings; overloading the preferences row would lose conflict semantics and per-entity events.

## Consequences

- Category tagging is many-to-many and forward/backward compatible: old slices load, and a rename
  updates every tagged slice for free because slices reference ids, not copies.
- Manage-categories and the slice dialog open instantly from cache and self-correct in the background;
  a cold start still shows an honest loading state.
- Offline category writes are durable and idempotent (ADR-049); a lost name-conflict race is dropped
  last-write-wins and surfaced non-blockingly rather than retried forever.
- Edits converge across screens and devices within a STOMP round-trip, at the cost of a refetch per
  event (acceptable for a small, infrequently-changed list).
- The cache JSON shape and the `categoryIds` field become soft local contracts; both degrade to empty
  on malformed data rather than crashing, at the cost of a silent re-fetch/re-tag.

*Related: FE #335, FE #336, FE #337, [ADR-007](007-websocket-stomp-realtime.md), [ADR-008](008-riverpod-state-management.md), [ADR-027](027-offline-action-queue.md), [ADR-029](029-shared-preferences-persistence.md), [ADR-032](032-server-synced-preferences.md), [ADR-049](049-typed-envelope-action-queue.md)*
