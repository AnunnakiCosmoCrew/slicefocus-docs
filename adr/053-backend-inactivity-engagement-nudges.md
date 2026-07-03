# ADR-053: Backend inactivity-based engagement nudges via FCM

**Status:** Accepted
**Date:** 2026-07-03
**Deciders:** Mert Ertugrul

## Context

Re-engagement epic (BE #346, FE follow-up): nudge users who have drifted away back into the app
(e.g. *"A few focused minutes go a long way. Open SliceFocus when you're ready."*). The trigger is
**inactivity** — a user who has neither started nor been running a focus session recently.

This is a genuinely different use case from the slice-reminder precedent in
[ADR-050](050-ekreminders-for-slices.md), which deliberately kept pre-start slice reminders on the
device via EventKit and **avoided** the FCM backend. That reasoning does not transfer here:

- A slice reminder fires at a **time the device already knows** (the slice's start minus a lead), so
  nothing server-side is needed. An inactivity nudge depends on knowledge **only the backend holds** —
  whether *any* of the user's devices has recorded a focus session in the last 24h, and whether a
  session is currently active. A single device cannot compute "the user has been inactive
  everywhere"; the server can, from `focus_sessions` across the account.
- The nudge is not anchored to a user-authored time; it is the backend's decision to reach out. That
  is exactly what the FCM pipeline ([ADR-010](010-fcm-push-notifications.md)) already does for session
  events.

## Decision

Add a backend `@Scheduled` `EngagementReminderScheduler` (modeled on `SliceEndWarningScheduler`) that
runs hourly and sends inactivity nudges over the existing FCM pipeline (`PushNotificationService`,
`Device` tokens). Key choices:

- **Opt-out toggle, separate from `notificationEnabled`.** New `UserPreferences.engagementRemindersEnabled`
  (default `true`) so motivational nudges can be silenced **without** losing session/phase alerts. The
  two audiences are different and deserve independent control.
- **Eligibility resolved in one query.** A user is eligible when: reminders enabled, ≥1 registered
  device, **no** focus session started in the last 24h, **no** currently active (RUNNING/PAUSED)
  session, and not already nudged in the last 24h. The recent-activity and active-session checks are
  both kept — a session started >24h ago that is still running must still suppress the nudge.
- **Frequency cap via `lastEngagementNudgeAt`.** Stamped on each send; enforces max one nudge per user
  per day. Kept as internal bookkeeping — **not** exposed in the preferences API, since the client has
  no use for it.
- **Fixed UTC quiet-hours window (08:00–21:00).** No per-user timezone is stored yet, so v1 gates on
  UTC. This is a deliberate rough edge (a user well east/west of UTC may be nudged at an odd local
  hour); per-user quiet hours are out of scope until a timezone is available. Window and enable flag
  are configurable (`slicefocus.engagement-reminder.*`).
- **Rotating message pool, sincere/minimal voice.** A small pool of variants rotated deterministically
  per user per day (no `Math.random`) to avoid staleness, in the [ADR-042](042-minimal-sincere-marketing-design-language.md)
  voice — invitational, never nagging.

## Alternatives Considered

- **Device-local scheduled notifications (the ADR-050 approach).** Rejected: the device cannot know the
  user is inactive *across all their devices*, and there is no user-authored time to anchor to. This is
  inherently server-knowledge-dependent.
- **Reusing `notificationEnabled` as the gate.** Rejected: conflates two distinct consent decisions —
  a user may want session alerts but not motivational nudges (or vice versa).
- **Per-slice / streak-based triggers.** Out of scope for v1 (no streak tracking exists); inactivity is
  the simplest signal that already has the data behind it.
- **Per-user quiet hours.** Deferred until a user timezone is stored; fixed UTC window is the v1
  compromise.

## Consequences

- Motivational re-engagement now has a dedicated, independently-controllable channel over the existing
  FCM infrastructure — no new vendor or delivery mechanism.
- The FCM-vs-EventKit split is now use-case-driven and documented: **user-authored, time-anchored,
  single-device** nudges stay device-local (ADR-050); **server-knowledge-dependent** nudges go through
  FCM (this ADR).
- UTC quiet hours is a known limitation; revisiting it requires capturing a per-user timezone.
- macOS FCM delivery (the first release target) is a delivery risk flagged in the FE sub-issue, not the
  backend.

*Related: BE #346, epic #345. Builds on [ADR-010](010-fcm-push-notifications.md); contrasts with
[ADR-050](050-ekreminders-for-slices.md); voice per [ADR-042](042-minimal-sincere-marketing-design-language.md).*
