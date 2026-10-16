# ADR-060: Hold the realtime WebSocket only while the app is in use

**Status:** Accepted
**Date:** 2026-10-16
**Deciders:** Mert Ertugrul
**Refines:** [ADR-007](007-websocket-stomp-realtime.md) (WebSocket with STOMP for real-time sync)

## Context

ADR-007 gave every client a STOMP-over-SockJS socket to `slicefocus-backend` for cross-device sync. The Flutter app opened it at start-up and kept it open for as long as the process lived, whatever the window or lifecycle state, and reconnected on every drop.

`slicefocus-backend` runs on Cloud Run with request-based billing and one minimum instance. Cloud Run treats an open WebSocket as an in-flight request, so while any client is connected the instance is billed at the active rate (about four times the idle min-instance rate) rather than the idle rate. Each connection lasts exactly the 300 s request timeout, then the client reconnects (~290 upgrades a day). Between 1 and 16 October 2026 the service was active 77% of wall time (1,068,857 CPU-seconds on "Services CPU Tier 2 (Request-based billing)", +688% against the previous period, about ₺112/day), nearly all of it from one macOS build left running around the clock, and the account hit its monthly budget on 16 October ([SliceFocus#436](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/436)).

What the socket carries is thin: user-scoped broadcasts on `/topic/user/{id}/session|slices|categories|day-templates`, all of which the app already reconciles over REST when it returns to the foreground (`HomePage._onAppResumed` fetches the active session and invalidates the day's providers). The timer runs locally; phase changes and server-side stops also arrive as push notifications. Nothing on the socket is needed while the app is hidden, backgrounded, or long unfocused.

## Decision

1. **The client holds the socket only while the app is in use.** A pure-Dart `SocketLifecyclePolicy` (SliceFocusFE `lib/src/core/network/socket_lifecycle_policy.dart`), fed by an `AppLifecycleListener`, drives `SessionEventClient`:

   | Lifecycle state | Decision |
   |---|---|
   | `resumed` | connect; cancel any grace timer |
   | `inactive` (visible, not focused) | connect if not connected; disconnect after a **5 min** grace |
   | `hidden` (window hidden or about to be suspended) | disconnect after **10 s**; never connects |
   | `paused`, `detached` | disconnect at once (iOS suspends the process before a timer would fire) |

   `SessionEventClient.disconnect()` now *suspends* the client: nothing inside it reconnects until the policy calls `connect()` again, and callbacks from a replaced socket are ignored. The app no longer connects at start-up when it launches hidden (menu-bar resident, MER-542).

2. **The server sends and expects STOMP heartbeats every 10 s** (`slicefocus.websocket.heartbeat-millis`, `WebSocketConfig`). Spring's simple broker closes a session that has sent nothing for three intervals, so a connection that died without a close frame (suspended iOS app, closed lid, dropped Wi-Fi, any pre-policy build that vanishes) stops counting as an in-flight request after about 30 s instead of the 300 s request timeout. `0` disables heartbeats and restores the previous behaviour without a code change.

3. **Explicit, cheap scaling limits in `deploy.sh`:** `--max-instances 3` (a hard cost ceiling: an active instance is about ₺3,400/month on request-based billing, so about ₺10,000/month at worst) and `--concurrency 500` (idle WebSockets hold no Tomcat thread, so one instance keeps many of them instead of scaling out; Tomcat's 200 worker threads remain the REST ceiling). Before this a deploy kept whatever the console last had (maxScale 20, concurrency 80).

4. **Not changed:** request-based billing, minScale 1, the 300 s request timeout, SockJS, the topics and payloads, the REST reconcile on resume.

This is the short-term layer. A separate ADR (in preparation) looks at replacing the always-open socket with foreground refresh plus silent pushes (APNs is already wired into the backend); the policy here is kept simple and configurable (two grace constants, one heartbeat property) so it can be narrowed or removed when that lands.

## Alternatives Considered

- **Server-side idle close after N minutes.** Saves nothing on its own: the shipped client reconnects within 1 s (its own backoff) or 5 s (the STOMP library), and the server cannot tell an idle foreground client from a hidden one. Kept only in the narrow heartbeat form above.
- **Instance-based billing for the service.** Billed 100% of the time at roughly 75% of the active request rate: about ₺100/day, barely below the measured ₺112 and two to three times what the client policy reaches. It only pays off when the instance is active more than roughly 65% of the time, which it will not be once idle clients disconnect. Kept as a later option, not part of this change.
- **Disconnect on `inactive` immediately.** Would cut most of the remaining cost on desktop but makes a side-by-side window go dark the moment focus leaves it. The 5 min grace is the product trade-off; it is one constant.
- **Replace the socket with polling or push only.** Loses sub-second cross-device updates while two devices are in active use, which is the one case the socket is for.

## Consequences

- Expected active time falls from 77% to roughly the share of the day the app is in front of the user; at about 10% that is ≈ ₺35–45/day instead of ₺112, a saving of ≈ ₺1,500–2,000 a month. It arrives once the backend is deployed to prod and every long-running client is rebuilt from the policy branch (or quit).
- A visible-but-unfocused desktop window stops receiving live cross-device updates five minutes after the last focus and catches up the moment it is focused again. The local timer is unaffected.
- Reconnects become routine (each focus after a long absence costs one SockJS handshake and four subscriptions), so the client's stale-callback guard matters more than before.
- Verification on prod: the SKU "Services CPU Tier 2 (Request-based billing)" and the daily count of `httpRequest.status=101` log entries for `slicefocus-backend` should both drop (the 101 count from ~290/day to a handful per foreground session).
- Heartbeats add one newline frame each way every 10 s per connected client; negligible, and only while connected.
