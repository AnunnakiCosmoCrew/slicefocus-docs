# ADR-038: Google Cloud Tasks for Live Activity Phase Transition Scheduling

**Status:** Accepted
**Date:** 2026-04-09
**Deciders:** Mert Ertugrul

## Context

ADR-037 established that the iOS Live Activity widget computes Pomodoro phase transitions locally from a deterministic formula. The server sends session config via APNS, and the widget derives the current phase.

However, iOS Live Activities have a hard platform limitation: **the SwiftUI view body only re-evaluates when an APNS content-state push arrives**. `TimelineView` does not trigger re-renders in Live Activities (confirmed through extensive testing). This means the widget cannot autonomously transition between phases while the phone is locked — it computes the right answer, but the view never re-renders to show it.

The server must send an APNS push at each phase boundary to trigger the widget to re-render. The formula then computes the correct current phase.

### Previous approaches (rejected)

| Approach | Status | Problem |
|----------|--------|---------|
| `@Scheduled` 30s polling (#264) | Reverted | Wasteful DB queries, imprecise timing, doesn't survive Cloud Run scale-to-zero |
| Per-session `TaskScheduler` (#266) | Closed | In-memory timers lost on Cloud Run restarts, complex lifecycle management |
| Client-only formula (ADR-037 original) | Amended | Widget can't re-render without external trigger |
| Client-side Dart scheduling | Not viable | Dart runtime is suspended when phone is locked |

## Decision

Use **Google Cloud Tasks** to schedule one HTTP task per Pomodoro phase boundary. When a session starts, the server computes all future phase boundary timestamps and creates a Cloud Task for each. At the scheduled time, Cloud Tasks calls an internal webhook on the Cloud Run service, which sends an APNS push to trigger the widget re-render.

### Architecture

```
Session starts
  → compute all phase boundaries (Work end, Break end, Work end, ...)
  → create one Cloud Task per boundary via REST API
  → each task scheduled at the exact transition time

At phase boundary:
  Cloud Tasks → POST /internal/cloud-tasks/phase-transition
    → load session from DB
    → if RUNNING and has push token → send APNS push
    → widget receives push → formula computes correct phase → re-renders
```

### Key design choices

**REST API over gRPC client library:** The initial implementation used `google-cloud-tasks` (gRPC + Netty), which consumed ~200-300MB of memory — unacceptable for a 512MB Cloud Run container. Replaced with direct REST API calls using `java.net.http.HttpClient` + `GoogleCredentials` (from Firebase Admin SDK transitives). Memory overhead: ~0MB.

**Config-based APNS payload (ADR-037 preserved):** The APNS push carries the session config, not derived phase state. The widget's deterministic formula computes the current phase from `elapsed = now - sessionStartTime - totalPausedSeconds`. Cloud Tasks just triggers the re-render — it doesn't carry phase labels or end times.

**Async task creation:** Creating ~30 tasks (for default 25/5/15/4 config over 8 hours) takes 3-7 seconds synchronously. Task creation runs asynchronously via `@Async` after the transaction commits (`TransactionSynchronization.afterCommit`), so the API response is not blocked.

**Separate security filter chain:** Cloud Tasks sends OIDC tokens (issuer: `accounts.google.com`), which conflicts with the Firebase JWT decoder (issuer: `securetoken.google.com`). A separate `SecurityFilterChain` for `/internal/**` bypasses OAuth2 resource server. `CloudTasksAuthFilter` verifies the OIDC token independently.

### Session lifecycle integration

| Event | Cloud Tasks Action | APNS Action |
|-------|--------------------|-------------|
| Start session | Schedule all phase transitions | — |
| Token registration | Schedule if not already scheduled | Send initial config push |
| Pause | Cancel all pending tasks | Send pause push |
| Resume | Reschedule from current state | Send resume push |
| Advance (manual/auto) | Reschedule | — |
| Complete | Cancel all pending tasks | Send end event |
| Cancel | Cancel all pending tasks | Send end event |

### Infrastructure

- **Queue:** `phase-transitions` in `europe-west3`
- **IAM:** Cloud Run SA has `roles/cloudtasks.enqueuer`; Cloud Tasks SA has `roles/run.invoker`
- **Webhook:** `POST /internal/cloud-tasks/phase-transition` — not in OpenAPI, secured via OIDC token verification
- **Feature toggle:** `slicefocus.cloud-tasks.enabled` (same pattern as Firebase/APNS)
- **Keep-alive:** Cloud Scheduler pings `/actuator/health` every 4 minutes on dev/test to prevent cold starts

## Alternatives Considered

1. **Accept the 0:00 freeze:** Let the widget show stale data until the user unlocks. Rejected — poor UX for a timer app, the core use case.

2. **Apple Push Notification Service only (no scheduling):** Send APNS push from a server-side scheduler. But Cloud Run is stateless — in-memory schedulers don't survive restarts or scale-to-zero. Cloud Tasks externalizes the scheduling.

3. **Firebase Cloud Messaging for Live Activities:** FCM does not support the `apns-push-type: liveactivity` header or `content-state` payload format required by ActivityKit (established in ADR-036).

## Consequences

### Easier
- Live Activity transitions at exact phase boundaries while phone is locked
- Survives Cloud Run restarts, scale-to-zero, and multi-instance deployments
- Negligible cost: 1M tasks/month free, ~$0.40/million after
- Webhook processing is ~50ms per task — minimal Cloud Run billing
- Feature-toggled — can be disabled without code changes

### Harder
- Additional GCP dependency (Cloud Tasks API + queue management)
- Task lifecycle must be managed on pause/resume/complete/cancel
- Duplicate tasks from concurrent scheduling paths (startSession + registerLiveActivityToken)
- ~3-5 second inherent latency (Cloud Tasks dispatch + APNS delivery) at phase boundaries

*Related: ADR-037 (client-computed formula — amended), ADR-036 (APNS infrastructure), #272 (parent), #273 (BE implementation), #280 (REST migration)*
