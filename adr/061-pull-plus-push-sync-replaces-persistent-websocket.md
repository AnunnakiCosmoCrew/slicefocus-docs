# ADR-061: Pull-plus-push sync replaces the persistent WebSocket

**Status:** Proposed (product decision; accepted when the founder moves the epic to Todo)
**Date:** 2026-10-16
**Deciders:** Mert Ertugrul
**Supersedes:** [ADR-007](007-websocket-stomp-realtime.md) (WebSocket with STOMP for real-time sync), once the last sub-issue of the epic lands
**Relates to:** [ADR-060](060-websocket-lifecycle-policy.md) (the stopgap this replaces), [ADR-010](010-fcm-push-notifications.md), [ADR-036](036-apns-live-activity-push.md), [ADR-038](038-cloud-tasks-phase-scheduling.md), [ADR-027](027-offline-action-queue.md)

## Context

### What the socket costs

`slicefocus-backend` runs on Cloud Run (request-based billing, one minimum instance, 80 requests per instance, up to 20 instances before 2026-10-16, capped at 3 since). Cloud Run bills an open WebSocket as an in-flight request, so one connected client keeps an instance at the active rate. Measured in October 2026: one instance held active around the clock costs about ₺112/day, ≈ ₺3,400/month ([SliceFocus#436](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/436)). Each instance holds 80 sockets, so the shipped design costs ≈ ₺3,400/month per 80 concurrently open apps: 1,600 users with the app open would need 20 instances, ≈ ₺68,000/month, for a free app. The cost is a function of how many apps are open, not of how much anyone uses them.

ADR-060 (the #436 stopgap) cuts the exposure by holding the socket only while the app is in front of the user, and by closing dead connections after 30 s of missed heartbeats. It does not change the shape of the cost: an instance is active whenever any user anywhere has the app in the foreground, and users still arrive 80 to an instance.

### What the socket actually carries

One STOMP-over-SockJS connection per app (`SessionEventClient`, `/ws`), subscribed to four topics scoped to the signed-in user (`WebSocketTopics`): `session`, `slices`, `categories`, `day-templates`. All traffic is server to client; the app never sends a frame. The payloads are thin:

| Topic | Publisher (backend) | Payload | What the app does with it (`HomePage`) |
|---|---|---|---|
| `session` | `FocusSessionService.broadcastSessionEvent` | started / advanced / paused / resumed / completed / cancelled, with `phaseStartedAt`, `remainingSeconds`, cycle counts | `FocusTimerController.applyServerEvent`: re-aligns the local timer; shows the "stopped on another device" banner |
| `slices` | `SliceService` | id, type, date | `ref.invalidate(slicesProvider)` if the event's date is the selected day |
| `categories` | `CategoryService`, `DefaultCategorySeedingService` | id, type | `ref.invalidate(categoriesProvider)` |
| `day-templates` | `DayTemplateService` | id, type | `ref.invalidate(userTemplatesProvider)` |

Three of the four handlers are "something changed, refetch the list". Only the session topic carries state, and the app already treats that state as advisory: `_onAppResumed` re-fetches the active session over REST on every return to the foreground (`fetchActiveSession` → `syncFromServer` or `_hydrateActiveSession`), and the `_syncTimerFromServer` comment records that "the dead socket never delivered SESSION_COMPLETED" was a real case the REST path had to cover. The socket syncs one user's own data between that user's devices; there is no shared or collaborative data.

The Pomodoro timer itself does not depend on the socket. It runs locally from a deterministic formula (ADR-037), the iOS Live Activity is driven by direct APNs pushes scheduled through Cloud Tasks (ADR-036, ADR-038), and phase alerts arrive as FCM notifications (ADR-010).

### What already exists

- **FCM, both ends.** The backend sends FCM through `PushNotificationService.sendToUser` (Firebase Admin SDK, `devices` table keyed by `fcmToken` + `platform`, stale-token cleanup). The app registers a token on every platform (`FcmTokenService`; macOS registers as `ios`), listens in the foreground (`FirebaseMessaging.onMessage`) and has a top-level background handler that today only initialises Firebase. `macos/Runner/*.entitlements` carry `aps-environment`. `ios/Runner/Info.plist` does **not** declare `UIBackgroundModes: remote-notification`, so iOS silent pushes are not delivered to a backgrounded app yet.
- **Direct APNs** (`ApnsService`) for Live Activity content-state pushes, and **Cloud Tasks** for scheduled phase boundaries. Neither is needed for data sync; FCM is the right channel because it already reaches every platform and token lifecycle is solved.
- **REST reconcile on resume** (`_onAppResumed`), the offline action queue (ADR-027/049) and per-collection Riverpod providers that refetch on invalidation.
- **No "changes since" endpoint.** `Category` and `DayTemplate` have `updated_at`; `Slice` has only `created_at` and `version`. Resume today refetches four small collections unconditionally.

## Decision

Replace the persistent socket with **pull plus push**: the app pulls over REST at the moments it needs fresh data, and the server sends a **silent, data-only FCM push** when another device of the same user changes data. No connection stays open between requests, so Cloud Run is active only while a request is being served, and cost scales with real usage.

### Pull

1. **On resume** (every transition to `resumed`, as today) and **on sign-in**, the app calls `GET /api/v1/sync/stamps?dates=<selected day>` and receives one change stamp per collection: `activeSession`, `slices[date]`, `categories`, `dayTemplates`. A stamp is `max(updated_at)` plus the row count of the collection (count catches deletions), which needs an `updated_at` column on `slices`. The app refetches only the collections whose stamp changed, falling back to a full refetch if the call fails. This is the "changes since" endpoint; it is a stamps endpoint rather than a delta feed because the collections are small (one day of slices, a few dozen categories and templates) and a delta feed would need tombstones and cursors for no measurable gain.
2. **Foreground polling fallback.** While the app is `resumed` and silent push is not known to work on this device, the app polls the stamps endpoint every 60 s. "Not known to work" means: no FCM token registered, notification permission denied, or the platform's background delivery unverified (macOS until the epic verifies it). A stamps call is one indexed query and a few hundred bytes. Devices with working push do not poll.

### Push

3. **On every mutation the five services broadcast today**, the backend sends a data-only FCM message (`content-available: 1`, `apns-priority: 5`, no `notification` block) to the user's other devices: `{ "type": "sync", "collection": "slices", "date": "2026-10-16", "origin": "<deviceId>" }`. The originating device is excluded (the app sends an `X-Device-Id` header on writes; the backend already knows the device from the `devices` table) so a device never refreshes because of its own write. A per-user, per-collection coalescing window of about 2 s absorbs bursts such as a drag edit that issues several PUTs. Coalescing is in-memory per instance and best-effort: a missed push is covered by the next resume or poll.
4. **The app treats a sync push as a stamp-check trigger**, not as data: foreground (`onMessage`) and background handler both call the stamps endpoint and invalidate the changed providers; the session collection goes through the existing `fetchActiveSession` → `syncFromServer` path. iOS gets `UIBackgroundModes: remote-notification`; the macOS app receives pushes while running (macOS does not suspend it); Android receives data messages subject to Doze.

### Timer live state

The `session` topic is the only one where the socket carried state rather than a hint. After this change: a start, pause, resume, stop or cancel on device A reaches device B as a sync push within a few seconds (one push, one GET of the active session, `syncFromServer`), or within 60 s by polling, or on the next resume. Phase advances need no sync at all: both devices compute the same timeline from `phaseStartedAt` and the session configuration (ADR-037); the Live Activity keeps its Cloud Tasks + APNs path (ADR-038); phase alerts keep their FCM notifications (ADR-010). The "stopped on another device" banner is raised from the REST result instead of a socket frame.

### Rollout

The change ships in four steps, each independently deployable and reversible: backend push (behind `slicefocus.sync.push-enabled`, the WebSocket broadcast continues in parallel); backend stamps endpoint; app resume-refresh, push handling and polling fallback, with the socket switched off behind a build flag (`SLICEFOCUS_REALTIME=push|socket`, default `push`); then removal of `/ws`, the STOMP broker and `stomp_dart_client` once the push build is the minimum supported version. ADR-007 is marked superseded at that last step; ADR-060 stays in force for every socket build in the wild until then.

## Alternatives Considered

Per-user figures are rough and assume 1,600 users who each have the app in the foreground about 10% of the day (≈ 160 apps open at any moment), the measured ₺3,400/month for an active instance, and a quarter of that (≈ ₺850/month) for the idle minimum instance, which every option pays and which is excluded from the per-user lines.

| | Option | Per-user cost, monthly | Latency of a cross-device update | Verdict |
|---|---|---|---|---|
| (a) | **Keep the socket, foreground-only with idle cut-off** (ADR-060, #436) | ≈ ₺4 (160 concurrent sockets = 2 active instances ≈ ₺6,800). Floor ₺3,400/month as soon as a few dozen users span the day. With the 3-instance cap the 241st concurrent app is refused | Sub-second while both devices are in the foreground | **Stopgap only.** Cost still scales with open apps, not usage, and the cap turns growth into connection failures |
| (b) | **Pull on resume + silent push, no persistent socket** | ≈ ₺0.1–0.4 (≈ 20 resumes/day × a few 100 ms requests; FCM and APNs are free). Instance active only while serving requests | A few seconds via push; ≤ 60 s where polling fallback applies; otherwise next resume | **Chosen** |
| (c) | **Short polling while in the foreground** (stamps every 30 s, no push) | ≈ ₺1 (≈ 5 req/s across 1,600 users keeps one instance roughly half active ≈ ₺1,700/month); linear in the poll rate, ≈ ₺3 at 10 s | ≤ poll interval | **Kept as the fallback in (b)**, at 60 s, only on devices without working push. Alone, it reintroduces a usage-independent cost and Neon load |
| (d) | **Managed realtime service** (Ably, Pusher Channels, Firebase Realtime Database listeners) | Priced on concurrent connections and messages: on the order of $30–50/month at this size, ≈ $0.03 per user, plus the backend still publishes every change. Not free at 10× the users | Sub-second | **Rejected.** A second system to authenticate against and operate, a vendor dependency for a feature whose value is "refresh the list", and the client keeps an always-connected pattern that costs battery. Firebase RTDB also recreates the two-sources-of-truth problem ADR-007 rejected |

Also considered and rejected: **Server-Sent Events** (same in-flight-request billing as the socket; nothing gained); **instance-based billing** (bills 100% of the time at ≈ 75% of the active rate, ≈ ₺100/day per instance, and keeps the 80-per-instance scaling); **dropping the minimum instance to zero** (saves the idle ₺850/month but adds a Spring Boot cold start to every resume and to the Cloud Tasks phase webhooks; worth a separate decision once the socket is gone, not before).

## Consequences

- **Cost scales with usage.** Backend cost per user falls from ≈ ₺4/month (and a hard cap on concurrent users) to well under ₺1/month, with the only fixed cost the idle minimum instance already paid today. 1,600 users cost roughly what 16 cost. The 3-instance cap can stay as a safety limit instead of a product limit.
- **Cross-device updates lose sub-second latency.** A change on another device appears after a few seconds (push), up to 60 s (polling fallback), or on the next resume. For a single user's own planner this is the product trade-off; it is the one thing the founder must accept, and it is a visible change for anyone who runs the Mac and the phone side by side.
- **Dependency on FCM delivery.** Silent pushes are best-effort: iOS throttles background pushes (a few per hour) and sends none to a force-quit app; Android Doze delays them; macOS delivers only to a running app. Every path therefore ends in a REST pull, and the polling fallback covers devices where push cannot be confirmed. The epic must verify delivery on macOS and iOS before the flag defaults to `push` in a release.
- **A new header and a new column.** Writes carry `X-Device-Id`; `slices` gains `updated_at` (a migration with a backfill from `created_at`). The stamps endpoint is additive.
- **Simpler backend at the end.** `WebSocketConfig`, `WebSocketAuthInterceptor`, the simple broker, `WebSocketTopics` and the SockJS allowed-origin configuration go away, and with them the "will need an external broker before horizontal scaling" consequence of ADR-007: FCM fan-out works across any number of instances. Cloud Run's 300 s request timeout stops mattering.
- **Simpler client, more lifecycle code.** `SessionEventClient`, `SocketLifecyclePolicy` (ADR-060) and `stomp_dart_client` go away; sync-push handling, device-id plumbing, the stamps client and the polling fallback come in. The event classes (`SessionEvent`, `SliceEvent`, `CategoryEvent`, `TemplateEvent`) survive only where `FocusTimerController.applyServerEvent` still consumes a `SessionEvent` built from the REST result.
- **Two mechanisms in the wild for one release cycle.** Until the push build is the floor, the backend both broadcasts to the socket and sends sync pushes; the app flag decides which it uses. ADR-060 governs the socket builds in the meantime.
- **Verification on prod:** after the flag flips, the SKU "Services CPU Tier 2 (Request-based billing)" should track request count rather than wall time, and `httpRequest.status=101` upgrades should fall to zero for push builds; FCM delivery can be checked from the Firebase console's message delivery data.
