# ADR-050: EKReminders for slice reminders

**Status:** Accepted
**Date:** 2026-07-01
**Deciders:** Mert Ertugrul

## Context

The Calendar & Reminders epic (FE #360) lets a user be nudged before a planned slice begins by
creating a native reminder in Apple's Reminders app. The app already bridges to EventKit for calendar
export ([ADR-046](046-calendar-export-device-local-id-map.md)) over a hand-rolled platform channel,
so reminders reuse that native surface rather than introducing a new notification mechanism.

The design turns on how much machinery a slice reminder deserves. Two forces pull in opposite
directions:

- Calendar export invested in a device-local id map, a decorator, self-heal, and offline replay so that
  the calendar mirror is *exact and duplicate-free*.
- A pre-start reminder is far lower stakes: it is a convenience nudge, not a record of work. A missed or
  slightly-off reminder costs the user nothing they can't recover, unlike a lost focus-session
  completion ([ADR-027](027-offline-action-queue.md)) or a duplicated calendar event.

We also send notifications through Firebase Cloud Messaging ([ADR-010](010-fcm-push-notifications.md))
for session events, so a decision is needed on why reminders use EKReminders instead.

## Decision

Create an `EKReminder` on slice save through a dedicated EventKit channel, with deliberately minimal
lifecycle machinery. Key design choices:

- **Its own platform channel, shared event store.** `EventKitRemindersRepository` talks over
  `com.slicefocus.reminders` (`createReminder`, `completeReminder`, `listReminders`,
  `requestAccess`, `authorizationStatus`), separate from `com.slicefocus.calendar` but backed by the
  same long-lived `EKEventStore` in the iOS/macOS `AppDelegate`. iOS 17+/macOS 14+ use
  `requestFullAccessToReminders` with a pre-version `requestAccess` fallback; the surface is a no-op on
  platforms without EventKit.
- **Opt-in with a lead time.** A settings toggle (`remindersSettingsProvider`) gates creation and stores
  a lead-time in minutes. `reminderDueForSlice(date, startMinute, leadMinutes)` computes the due instant
  as local midnight + `(startMinute − leadMinutes)`, so a lead larger than the start time rolls into the
  previous day — the intended "remind me before it starts" semantics.
- **Best-effort, no retry queue.** Reminder creation runs fire-and-forget after the slice is saved and
  swallows failures. Unlike calendar export, it does **not** enqueue on the `ActionQueue` — a nudge that
  fails to materialize is not worth the durability cost, and a stale queued reminder could fire for a
  time that has already passed.
- **Weak linkage to the slice (title only).** The reminder carries the slice's title and due time but no
  stored slice id and no entry in a mapping table. Consequences are accepted deliberately: deleting a
  slice does not remove its reminder, and moving a slice after save does not reschedule it. Reminders are
  create-time snapshots, surfaced/completed through a reminders sheet, not a synced projection of the
  slice.

## Alternatives Considered

- **FCM push ([ADR-010](010-fcm-push-notifications.md)) for pre-start nudges.** Would route through the
  backend and require server-side scheduling of a per-slice fire time. Over-engineered for a local,
  time-anchored nudge, and it would put reminders on a network dependency the user's own device doesn't
  need. EventKit fires locally and lands in the Reminders app the user already checks.
- **Local notifications (`flutter_local_notifications`).** Fires an alert but leaves nothing the user can
  see, re-open, or check off later. EKReminders integrate with the native Reminders app, syncing across
  the user's Apple devices for free.
- **Full calendar-grade lifecycle (id map, decorator, self-heal, offline replay).** Symmetric with
  ADR-046 but unjustified: it would keep reminders exactly in sync with slice edits and deletes at a real
  complexity cost, to protect a low-stakes convenience feature. We consciously accept orphaned/stale
  reminders instead.
- **Store a slice→reminder id map for cleanup.** A middle ground that would at least allow deleting a
  slice's reminder. Deferred: the map is only worth it once users ask for edit/delete propagation; until
  then it is unused bookkeeping.

## Consequences

- A pre-start reminder appears in the native Reminders app and syncs across the user's Apple devices with
  no backend involvement.
- The implementation is small and has no durable state to migrate or reconcile.
- Reminders can drift from slices: editing a slice's time does not move its reminder, and deleting a slice
  leaves the reminder behind. This is a known, accepted limitation; propagation (via a slice→reminder id
  map) is future work if users need it.
- Transient create failures are silently dropped — acceptable for a nudge, but it means "I saved a slice
  and got no reminder" has no retry path today.
- Reminders and calendar now share the `EKEventStore` and permission-request pattern but keep independent
  channels and independent authorization, so a user can grant one without the other.

*Related: FE #360, [ADR-010](010-fcm-push-notifications.md), [ADR-022](022-macos-menu-bar-platform-channels.md), [ADR-027](027-offline-action-queue.md), [ADR-046](046-calendar-export-device-local-id-map.md), [ADR-047](047-two-way-calendar-reconciliation-snapshot-echo-guard.md)*
