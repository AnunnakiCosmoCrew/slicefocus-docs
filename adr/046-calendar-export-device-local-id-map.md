# ADR-046: Device-local slice→calendar-event id map for one-way export

**Status:** Accepted
**Date:** 2026-07-01
**Deciders:** Mert Ertugrul

## Context

The Calendar & Reminders epic's export phase (FE #358) mirrors planned slices into the system calendar (Apple EventKit) as create/update/delete operations, opt-in per user. This is one-way (slice → calendar); reading calendar edits back into slices is a later, separate phase.

Two properties make the design non-obvious:

- **EventKit identifiers are per-device.** The `eventIdentifier` returned when creating an event is local to that device's event store — it is meaningless on another device and cannot be reconstructed from slice data. So to update or delete "the event for this slice" later, we must remember the slice→event association *somewhere*, and that somewhere cannot be the backend `Slice` (which is shared across a user's devices).
- **Re-export must not duplicate.** Toggling export off/on, editing a slice, or replaying a queued write must converge to exactly one event per slice, never a second copy.

We also need the export to survive transient failures and app restarts, consistent with the offline-first posture of the focus-session queue ([ADR-027](027-offline-action-queue.md)).

## Decision

Maintain a **device-local slice→event id map** in `SharedPreferences` ([ADR-029](029-shared-preferences-persistence.md)), namespaced per user (`calendar_export_map_<userId>`), and never persist it on the backend `Slice`. Key design choices:

- **Decorator over the repository.** A `CalendarExportSliceRepository` wraps `HttpSliceRepository` and reflects each successful `saveSlice`/`updateSlice`/`deleteSlice` to EventKit. The HTTP transport stays pure, and every mutation path — including the focus-session ±5-minute tolerance updates ([ADR-034](034-flexible-session-start-stop-tolerance.md)) — is covered for free. Virtual "Unplanned" gap slices and unpersisted (null-id) slices are skipped.
- **Idempotency via the map.** A slice with no mapping ⇒ create the event and store its id; a slice with a mapping ⇒ update in place. There is never a second create for the same slice, so re-export is safe.
- **Self-heal on external deletion.** If the user deletes an exported event by hand, the next update surfaces the native `NOT_FOUND` code (`CalendarEventNotFoundException`); the app recreates the event and remaps. The create/update/delete/self-heal logic lives in a single `CalendarExportReflector` shared by the live write path and the offline replay so the rules can't drift.
- **Synchronous best-effort, queue on failure.** EventKit is a local, fast operation, so reflection runs synchronously after the backend write succeeds. Only when a calendar write actually fails is a `calendar_export` action enqueued on the existing `ActionQueue` for replay on reconnect — reusing ADR-027's durable, ordered, restart-surviving log rather than bouncing every local write through it.
- **Device-local settings.** The opt-in toggle and target-calendar id are stored locally and are *not* server-synced (unlike theme/timer prefs, cf. [ADR-032](032-server-synced-preferences.md)), because the target calendar id is itself a per-device EventKit identifier.

## Alternatives Considered

- **Store the event id on the backend `Slice`.** Simplest to reason about, but wrong: EventKit ids are per-device, so a slice synced to a second device would carry a stale id that resolves to nothing (or, worse, a different event). It would also leak a device-specific concern into the shared domain model.
- **Always route export writes through the `ActionQueue`.** Faithful to the issue's wording and uniform, but it forces a persisted round-trip for what is normally an instantaneous local write, and ties EventKit availability to network connectivity (the queue's flush trigger), which are unrelated. We keep the queue strictly as the failure/retry path.
- **Edit `HttpSliceRepository` directly.** Fewer files, but couples HTTP transport to calendar side-effects and complicates testing of both. The decorator keeps concerns separated and matches the repository pattern.
- **Derive events on demand (no id map).** Query the calendar for a matching event instead of remembering its id. Fragile (matching by title/time is ambiguous) and slow; the id map is exact and cheap.

## Consequences

- Export is exact and duplicate-free across toggles, edits, and replays, and self-heals when events are deleted outside the app.
- The id map is device-scoped and cleared on account switch (per-user key), so accounts never cross wires; it is rebuilt lazily as slices are re-exported after a reinstall.
- The map's JSON shape becomes a soft local contract; corrupt data self-heals to empty (ADR-029), at the cost of silently re-exporting on the next write.
- One-way only: events the user edits in their calendar are not read back into slices. Divergence is expected until the two-way phase; re-exporting a slice overwrites its event.
- The `CalendarExportReflector` is the single source of truth for id-mapping and self-heal, shared by the live decorator and queue replay — new export triggers should call it rather than reimplement the rules.

*Related: FE #358, [ADR-027](027-offline-action-queue.md), [ADR-029](029-shared-preferences-persistence.md), [ADR-022](022-macos-menu-bar-platform-channels.md)*
