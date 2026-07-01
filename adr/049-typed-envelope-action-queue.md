# ADR-049: Typed-envelope generalization of the offline action queue

**Status:** Accepted
**Date:** 2026-07-01
**Deciders:** Mert Ertugrul

## Context

The offline `ActionQueue` ([ADR-027](027-offline-action-queue.md)) was built for exactly one kind of
write: a focus-session completion. Each stored entry was a bare `FocusSessionSummary` JSON map, and
the flush loop knew how to replay only that.

Two later features need the same durable, ordered, restart-surviving replay for *different* writes:

- **Categories** (FE #337) must not lose an offline create/rename/delete, and must replay them in order
  on reconnect.
- **Calendar export** ([ADR-046](046-calendar-export-device-local-id-map.md)) already enqueues a
  `calendar_export` action when an EventKit write fails transiently.

Standing up a second and third parallel queue would duplicate the connectivity trigger, the
per-user-keyed SharedPreferences storage, the serialized-flush guard, and the restart recovery — and
give three chances to get cross-account isolation wrong. We need one queue that can carry heterogeneous
actions without breaking the entries already persisted on users' devices by the session-only version.

## Decision

Replace the bare-summary entry with a **typed `QueuedAction` envelope** and dispatch on it, keeping a
single queue instance per session. Key design choices:

- **The envelope.** `QueuedAction { entity, type, id, payload, ts }` where `entity ∈ {session,
  category, calendar_export}` and `type ∈ {complete, upsert, delete}`. `id` is the target resource id
  (sliceId or categoryId), `payload` is the operation body (empty for deletes), and `ts` records
  creation time for last-write-wins ordering.
- **Migration-free legacy read.** `QueuedAction.fromJson` treats any entry *without* an `entity` key as
  a legacy session completion (`entity: session`, `type: complete`, payload = the old map). Entries
  written by the ADR-027 version replay unchanged — no migration step, no data loss on upgrade.
- **Dispatch by entity, fail-safe on the unknown.** `flush` decodes each entry and routes it:
  sessions to `FocusSessionRepository.completeSession`, categories to the `CategoryRepository`
  upsert/delete, calendar-export to the `CalendarExportReflector`. An unrecognized `entity` (e.g. an
  action written by a newer app version, then downgraded) is dropped rather than retained forever so it
  can't poison the queue.
- **Idempotent replay per entity.** Category upserts use a client-generated id + the backend's
  idempotent `PUT /{id}` (see [ADR-048](048-categories-client-model-and-live-synced-management.md)), so
  a replayed create never orphans a duplicate. Calendar-export replay is made idempotent by the
  device-local slice→event id map inside the reflector (ADR-046). Malformed calendar payloads (empty
  title, non-positive window) are dropped on replay instead of creating junk events.
- **Conflict handling is per-entity, not per-queue.** A category name conflict (`409`) during replay is
  a lost last-write-wins race: the offending action is dropped and emitted on a
  `Stream<CategoryNameConflictException>` for a non-blocking UI notice. All other failures are transient
  — the entry is kept for the next flush. Ordered, serialized flushing and per-user keying are unchanged
  from ADR-027.

## Alternatives Considered

- **One queue per feature (session queue + category queue + calendar queue).** No shared envelope, but
  triplicates the connectivity subscription, storage, flush guard, and per-user isolation. More code,
  more surface for account-crossing bugs, and three flush orders to reason about instead of one.
- **A generic "queue any repository call" abstraction.** Serialize an arbitrary method + args and replay
  it. Maximally flexible but effectively unbounded — it makes any call queueable, defeats typed dispatch,
  and turns the persisted format into an opaque RPC log that is impossible to validate or migrate.
- **Version + migrate the stored entries on upgrade.** Rewrite legacy session maps into the new envelope
  at read time via a one-shot migration. Correct but heavier than needed: the absence of an `entity` key
  is already an unambiguous discriminator, so a branch in `fromJson` achieves the same thing with no
  migration bookkeeping.
- **Drop offline support for categories/calendar (online-only writes).** Simplest, but categories and
  calendar export are edited exactly when a user is mobile and flaky; silently losing those writes is the
  same unacceptable UX ADR-027 was created to avoid.

## Consequences

- One durable, ordered, per-user queue now backs session completions, category writes, and calendar
  exports; new offline-capable writes add an `entity`/`type` and a dispatch arm rather than a new queue.
- Upgrades are seamless: entries persisted by the session-only queue replay without migration.
- The `QueuedAction` JSON shape is now a cross-feature contract (ADR-027 already flagged the queue format
  as a contract); adding fields is safe, but renaming/removing `entity`/`type`/`payload` requires care.
- Idempotency is delegated to each entity's replay path (client-id upsert for categories, id map for
  calendar), so correctness of "replay exactly once in effect" is verified per feature, not centrally.
- Unknown-entity entries are dropped by design, so a forward-then-back app downgrade can silently discard
  a queued action written by the newer version — an accepted trade for not wedging the queue.

*Related: FE #337, FE #358, [ADR-027](027-offline-action-queue.md), [ADR-046](046-calendar-export-device-local-id-map.md), [ADR-048](048-categories-client-model-and-live-synced-management.md), [ADR-029](029-shared-preferences-persistence.md)*
