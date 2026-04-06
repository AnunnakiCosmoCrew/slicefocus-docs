# ADR-036: Direct APNs for Live Activity Updates

**Status:** Accepted
**Date:** 2026-04-06
**Deciders:** Mert Ertugrul

## Context

SliceFocus displays an iOS Live Activity on the lock screen during focus sessions, showing the current Pomodoro phase and a countdown timer. When the app is suspended (screen locked, backgrounded), the Dart runtime is paused — no timers fire, no state transitions occur. The Live Activity's native `Text(timerInterval:countsDown:)` continues counting down visually, but when the countdown reaches 00:00, nothing updates the display to show the next phase.

We investigated three approaches to update the Live Activity while the app is suspended:
1. **TimelineView-based conditional rendering** — Write next-phase data to UserDefaults and check `now >= phaseEndTime` every second via `TimelineView`. This doesn't work because Live Activities don't re-evaluate their view body on timeline schedules — they use ActivityKit content state updates.
2. **Local notification as a trigger** — Schedule a `zonedSchedule` notification at phase end time. The notification fires but can't update the Live Activity (only the main app or APNs can modify it).
3. **APNs push notification** — The server sends an APNs push with updated content state when a phase advances. iOS processes it at the OS level and updates the Live Activity immediately.

The existing push notification system uses Firebase Cloud Messaging (FCM) for regular notifications (phase completion alerts, session end). FCM delivers to iOS via APNs passthrough, but **FCM does not support the Live Activity content-state payload format** (`apns-push-type: liveactivity` + `content-state` body). A direct APNs HTTP/2 client is required.

## Decision

Add a direct APNs HTTP/2 client alongside the existing FCM integration. The two push channels serve different purposes:

| Channel | Purpose | Token Lifetime | Token Storage | Payload |
|---------|---------|----------------|---------------|---------|
| **FCM** | Regular push notifications (phase alerts, session end, slice warnings) | Per-device, long-lived | `devices` table | Standard notification + data |
| **APNs (direct)** | Live Activity content-state updates | Per-activity, ephemeral | `focus_sessions.live_activity_push_token` | ActivityKit content-state |

### Architecture

1. **FE creates Live Activity** → receives an ActivityKit push token (per-activity, not per-device)
2. **FE sends token to BE** → `POST /api/v1/focus-sessions/active/live-activity-token`
3. **BE stores token** on the `FocusSession` entity (ephemeral — cleared on session complete/cancel)
4. **BE auto-advances phase** (ADR pending, #247) → sends APNs push with updated content state
5. **iOS receives push** → updates Live Activity on lock screen instantly

### Implementation Pattern

The APNs client follows the same patterns established by Firebase:

- **Feature toggle:** `slicefocus.apns.enabled` (default `false`), mirroring `slicefocus.firebase.enabled`
- **Credentials:** Apple Developer key stored in Secret Manager (per environment), mirroring Firebase credentials
- **Conditional bean:** `ApnsConfig` conditionally creates the APNs client bean, mirroring `FirebaseConfig`
- **Async dispatch:** Push sent via `@Async` annotation in `SessionNotificationService`, same as FCM
- **Error handling:** Token-level errors (invalid/expired) clear the stored token; delivery failures are logged but don't block the advance

### Rate Limiting

Apple allows approximately 15–20 Live Activity push updates per hour. Pomodoro intervals produce:
- Default settings (25 min work + 5 min break): ~4 pushes/hour
- Minimum settings (15 min work + 3 min break): ~7 pushes/hour

Both are well within Apple's budget.

## Alternatives Considered

1. **FCM passthrough for Live Activities:** FCM can send APNs payloads via its `apns` field, but it does not support the `liveactivity` push type or the `content-state` payload structure required by ActivityKit. Rejected — technically impossible.

2. **Background App Refresh (`BGAppRefreshTask`):** Request a brief wake-up near phase end time. The system determines when to actually execute it — not guaranteed to fire at the exact time. Rejected — unreliable timing.

3. **Silent audio background mode:** Play inaudible audio to keep the app alive. Rejected — violates App Store guidelines (Section 2.5.4) and would cause rejection.

4. **Accept the limitation:** Show "00:00" on the lock screen and auto-advance on unlock. Rejected — poor user experience for a timer app; users expect the lock screen to reflect real-time state.

## Consequences

### Easier
- Live Activity stays accurate while the phone is locked — the primary use case for a focus timer
- Multi-device sessions stay in sync (all devices with Live Activities receive the push)
- Server becomes the authoritative source for phase state (reduces client-side drift issues)

### Harder
- New infrastructure dependency (APNs key management per environment)
- Two push systems to maintain (FCM + APNs)
- APNs key rotation requires Secret Manager updates across environments
- Token lifecycle is more complex (per-activity vs per-device)
- Direct APNs HTTP/2 client adds a dependency (vs FCM which is part of Firebase Admin SDK)

*Related: #250 (parent), #251 (BE implementation), #252 (FE implementation)*
