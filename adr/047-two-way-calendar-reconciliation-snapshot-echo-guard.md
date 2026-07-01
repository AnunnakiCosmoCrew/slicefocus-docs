# ADR-047: Two-way calendar reconciliation via snapshot echo-guard

**Status:** Accepted
**Date:** 2026-07-01
**Deciders:** Mert Ertugrul

## Context

The Calendar & Reminders epic's two-way phase (FE #359) completes the loop opened by one-way export ([ADR-046](046-calendar-export-device-local-id-map.md)): when the user edits an exported event **in the system Calendar** — moves it, renames it, or deletes it — the change must be reflected back onto the matching slice via `SliceRepository`, which then propagates cross-device over the existing slice WebSocket.

Three properties make the design non-obvious:

- **Change observation is a native push, not a poll.** EventKit signals mutations through the `EKEventStoreChanged` notification, which carries no diff — it fires (often in bursts) and the app must re-query to learn what changed. Flutter had no stream for this; the app is otherwise entirely method-channel (request/response).
- **We must not echo our own writes.** Exporting a slice to the calendar *also* triggers `EKEventStoreChanged`. Naively reconciling that change back onto the slice would create a ping-pong loop between export and reconcile.
- **Last-write-wins has no timestamps to compare.** The issue specifies LWW comparing slice `updatedAt` vs event last-modified. But `DaySlice` carries no `updatedAt` and the backend `Slice` contract does not expose one; `CalendarEvent` had no last-modified field either. True timestamp-LWW is impossible on the front end alone.

## Decision

Reconcile by **diffing each mapped event against a locally stored snapshot of the last state we wrote**, and treat divergence — not a timestamp comparison — as the signal of a genuine user edit. This snapshot is simultaneously the change detector and the echo guard.

- **Snapshot in the id map.** [ADR-046](046-calendar-export-device-local-id-map.md)'s `slice→eventId` map is widened so each entry also stores the event `title`, start/end (epoch ms), the slice's local day, and EventKit's `lastModified` when known. The legacy bare-string format decodes transparently (a snapshot-less entry). The `CalendarExportReflector` refreshes the snapshot on every successful export write, so immediately after export the snapshot equals the event's state.
- **Native change stream.** A new `EventChannel` `com.slicefocus.calendar/changes` bridges `EKEventStoreChanged` (iOS + macOS) — the codebase's first EventChannel. A companion `fetchEventsByIds` method looks up the exact mapped events by id (regardless of which day they landed on), returning off the main thread; missing ids signal deletion.
- **Reconcile rules.** On a debounced change tick, a `CalendarReconciliationService` fetches the mapped events and, per mapping: **snapshot matches current state ⇒ ignore** (our own echo, or untouched — *this is the loop guard*); **diverges ⇒ `updateSlice`** with the new title/time (re-fetching the current slice to preserve colour + categories, which the calendar cannot carry) and refresh the snapshot; **event missing ⇒ `deleteSlice`** and drop the mapping.
- **Write-back bypasses export.** Reconciliation writes through the *undecorated* `SliceRepository`, never the `CalendarExportSliceRepository`, so reflecting a calendar edit cannot re-export it — a second, belt-and-suspenders guard against loops on top of the snapshot check.
- **Gated on the export toggle.** Two-way is the completion of export, not a separate opt-in: reconciliation runs only while `CalendarExportPreferences.enabled` is on, reusing the same id map and target calendar. No new settings UI.
- **Best-effort, per-mapping isolation.** Change ticks are debounced (bursty `EKEventStoreChanged`) and concurrent cycles coalesce; a failure on one mapping never aborts the batch, matching the offline queue's posture ([ADR-027](027-offline-action-queue.md)).

## Alternatives Considered

- **Add `updatedAt` to `DaySlice` + backend `Slice` and do true timestamp-LWW.** Faithful to the issue's wording, but a cross-stack change gated on a backend contract addition — out of scope for a front-end phase, and unnecessary in practice: the calendar edit that *triggers* reconciliation is by definition the latest write to that event, and the snapshot guard already prevents echoes. Recorded here so a future genuine offline-conflict need can revisit it.
- **A mute window around export writes** (ignore change notifications for N ms after we write). Simpler in spirit but fragile: it races the notification delivery, drops legitimate concurrent user edits inside the window, and tunes a timeout with no correct value. The snapshot compares actual state and has no timing assumptions.
- **Poll the calendar on an interval** instead of observing `EKEventStoreChanged`. Wasteful and laggy; EventKit already offers a precise push. The debounce absorbs the burstiness the push introduces.
- **Fetch by a date window** (reuse `fetchEvents(start,end)`) rather than by id. The map spans arbitrary days; a window either misses events or over-fetches. By-id is exact and detects deletions cleanly.
- **Import unmapped external events as new slices.** Explicitly out of scope for this phase (decided separately); reconciliation only touches events the app itself exported.

## Consequences

- Calendar edits reflect onto slices within one reconcile cycle with no echo loops, and no backend change was required.
- The snapshot is the single basis for both change detection and echo suppression, so the two can't drift apart; the reflector and the reconciliation service must keep writing it on every event mutation.
- LWW is *approximated*, not literal. The known gap: an in-app slice edit and a calendar edit to the same event racing while offline could resolve by which observation lands last. Given export pushes slice edits to the calendar immediately, the practical window is small; a future `updatedAt` would close it.
- One residual edge case: an *arbitrary multi-day* move of a **legacy** (pre-snapshot) mapping on its *first* reconcile can fail to locate the slice (its old day is unknown) and fall back to default colour/categories. Adjacent days are probed to cover the common cross-midnight case; after the first reconcile the snapshot carries the day and it self-corrects. Documented in `CalendarReconciliationService._findSlice`.
- The app now owns a native event stream (`EKEventStoreChanged`) — the first EventChannel — establishing the pattern for future push-style native signals. It is exercised in tests via a mocked Dart change stream, since no EventChannel unit harness exists (consistent with how the method-channel bridge is tested).

*Related: FE #359, [ADR-046](046-calendar-export-device-local-id-map.md), [ADR-027](027-offline-action-queue.md), [ADR-029](029-shared-preferences-persistence.md), [ADR-008](008-riverpod-state-management.md)*
