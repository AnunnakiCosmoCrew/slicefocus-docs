# ADR-062: The server owns the focus timeline; every client renders it

**Status:** Accepted (founder, 2026-10-16); amended 2026-10-20 (auto-advance on by default, SliceFocus#450). Implementation is tracked in the issues under *Rollout*.
**Date:** 2026-10-16
**Deciders:** Mert Ertugrul
**Supersedes:** [ADR-011](011-client-side-pomodoro-timer.md) (client-side Pomodoro timer with server-side session bookkeeping)
**Amends:** [ADR-037](037-client-computed-live-activity-timeline.md) (the client-computed timeline stays, but it renders the server's anchor and the Live Activity has one writer), [ADR-038](038-cloud-tasks-phase-scheduling.md) (the boundary webhook also alerts and broadcasts), [ADR-061](061-pull-plus-push-sync-replaces-persistent-websocket.md) (its premise that the timer stays local; server-originated sync pushes; the hidden resident Mac)
**Relates to:** [ADR-010](010-fcm-push-notifications.md), [ADR-032](032-server-synced-preferences.md), [ADR-036](036-apns-live-activity-push.md), [ADR-060](060-websocket-lifecycle-policy.md)

## Context

### The promise

A user has the Mac app and the iPhone app open and starts a session on the Mac. The iPhone lock screen, the iPhone app, the Mac menu bar and the Mac window show the same countdown. When the work phase ends, every surface moves to the break. The user walks away with the phone, follows the timer there, gets a notification when the break ends, comes back and finds the Mac in the next work phase. The cloud is the source of truth; the devices show it. On TestFlight 1.6.1+35 (2026-10-16) the founder saw the iPhone app and the lock screen minutes apart after a lock, and the review that followed found the promise is not kept at most of these steps.

### What runs today

As of 2026-10-16 (SliceFocusFE `main` at 3e6f7b4, SliceFocus `origin/main` at 0403f37, SliceFocusFE#565 in draft), four components each compute "which phase, how much is left" on their own:

| Component | Model | Clock | Corrected when | Runs the state machine? |
|---|---|---|---|---|
| Flutter app, `FocusTimerController` (iPhone and Mac alike) | an integer `remainingSeconds` decremented by a self-rescheduling 1 s `Timer` | none; it counts ticks | `syncFromServer` on `resumed` only, plus socket events while connected | yes, its own copy (`_tick`, `advance`, `_transitionToNextPhase`) |
| Backend `FocusSession` | anchor: `phaseStartedAt`, `lifecycleState`, `completedCycles`, per-phase `totalPausedSeconds`, `pausedAt` | server `Clock` | catches up on `GET /active`, in the Cloud Tasks webhook and on the auto-advance toggle, when the setting is on (SliceFocus#434) | yes, `advance()` and `autoAdvance()` |
| iOS Live Activity widget | the same anchor; `Text(timerInterval:)` counts natively | device clock | on every content-state write: the server's APNs push or the app's local update | no; renders one phase and clamps at 00:00 (ADR-037, MER-358) |
| macOS menu bar | a string such as `C1 23:45` | none | on every Dart tick | no |

What each one does wrong:

- **App.** Suspension stops the integer; it continues from where it stopped. At a boundary it dwells, then a 2 s timer in `HomePage` calls `advance()` with a fresh full phase, so each boundary puts the app a couple of seconds behind the server and nothing corrects a Mac that stays focused. The "is local ahead of the server" heuristics in `syncFromServer` (deleted in #565) and the `_pendingServerAdvances` echo counter exist only to referee this model against the server's.
- **Backend.** The catch-up on `GET /active` broadcasts nothing, so other devices never learn of it. `POST /active/advance` is one `advance(now)` with no idempotency, so a client advance that races the webhook skips a phase. Pause, resume and token registration act on the stored phase without catching up. No response carries the server's time. The only FCM phase alert is the reply to a manual advance; `handlePhaseTransitionWebhook` sends none, by design. Boundaries are computed in whole seconds from a sub-second `phaseStartedAt`. Nothing ends a session at the 8-hour cap.
- **Live Activity.** Two writers, two storages. The server's APNs push writes the activity's content state; the app writes UserDefaults through the `live_activities` 2.6.0 plugin, whose `update` also replaces the content state with `{appGroupId, updateId}`, so the widget (`getConfig`, keyed on `sessionStartTime > 0`) falls back to the app's UserDefaults. Whichever side wrote last wins. `LockScreenTimerOrchestrator` re-anchors `_phaseStartedAt = _clock()` on every phase change, including one that `syncFromServer` applied, so after a resume across a boundary the lock screen gets a full phase while the server's phase started minutes earlier. The push token is forwarded once per activity; a rotated token is lost.
- **macOS.** Swift sets `statusItem.button.title` and has no clock. A hidden, menu-bar-resident app (MER-542) gets no `resumed`, holds no socket (ADR-060) and never resyncs; a stop from the phone is invisible until the window opens.
- **Alerts.** `zonedSchedule` for the current phase only, per device. A locked iPhone gets "work ended", scheduled while it was awake, and never "break ended", because the app was suspended when the break began. The Mac alerts from its own timer. A stop elsewhere does not cancel the other device's pending alert.

### How the promise fails, step by step

| Step | Today | Cause |
|---|---|---|
| Start on the Mac, open on the phone | The phone hydrates from the server anchor; the Mac keeps counting its own integer | two timelines from the first second |
| Phone locked through a boundary | Prod backend (53c5b95): the lock screen freezes at 00:00. Dev backend plus #565: it moves on via the Cloud Tasks push | SliceFocus#434 and SliceFocusFE#559 cover this step |
| Mac at the boundary | Dwells, then a fresh full phase a couple of seconds late, while the server advanced at the real boundary | countdown model |
| Unlock the phone, open the app | Today the resync is skipped (SliceFocusFE#564). With #565 the app resyncs, then the orchestrator re-anchors the lock screen to "now" with a full phase, so lock screen and app disagree again | orchestrator re-anchor, dual storage |
| "Break ended, go back" on the phone | Never arrives while locked | no server boundary alert; the local alert covers the current phase only |
| Back at the Mac | Running, a few seconds off; with the window closed it is never corrected, and a stop from the phone is not seen until the window opens | socket off while hidden, no resume, no poll |

### How we got here

ADR-011 moved the state machine to the client to stop server-side drift and polling. ADR-036 added direct APNs so the lock screen could follow; ADR-037 made the widget compute from a config; ADR-038 added Cloud Tasks because the widget cannot re-render without a push; MER-358 re-anchored the content state at the phase after the three implementations disagreed; SliceFocus#434 gave the server the auto-advance setting and the boundary advance; SliceFocusFE#565 removes the client heuristics. Each step fixed the visible symptom by adding an owner or a referee. The remaining defects (#564, the lock-screen gap on 1.6.1+35, the silent phone) are not new bugs; they are the model.

## Decision

One rule: **the server owns the timeline, and every client is a renderer of it.** No client keeps a countdown or decides a phase on its own; it holds the server's anchor and computes "phase and remaining as of now" from it, with one shared formula.

### The anchor and the render function

```
anchor = { phaseStartedAt, lifecycleState, completedCycles, phasePausedSeconds, pausedAt, isPaused,
           workSeconds, shortBreakSeconds, longBreakSeconds, cyclesBeforeLongBreak,
           sessionStartTime, maxSessionSeconds, autoAdvance, completedWorkSeconds }

render(anchor, now):
  ref = isPaused ? pausedAt : now
  if autoAdvance and not isPaused:
    while ref - phaseStartedAt - phasePausedSeconds >= duration(lifecycleState)
          and phaseStartedAt + duration(lifecycleState) < sessionStartTime + maxSessionSeconds:
      phaseStartedAt += duration(lifecycleState) + phasePausedSeconds   # the boundary, not now
      phasePausedSeconds = 0
      lifecycleState, completedCycles = next(lifecycleState, completedCycles)
  remaining = max(0, duration(lifecycleState) - (ref - phaseStartedAt - phasePausedSeconds))
  if ref - sessionStartTime >= maxSessionSeconds: remaining = 0
```

`next` is the backend's `FocusSession.advance()`: WORK increments `completedCycles` and goes to LONG_BREAK when the count reaches `cyclesBeforeLongBreak`, else SHORT_BREAK; SHORT_BREAK goes to WORK; LONG_BREAK goes to WORK and resets the count to 0. With auto-advance off the render dwells at 0 and the user advances by hand. **Amendment (founder, 2026-10-20, [SliceFocus#450](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/450)):** auto-advance is **on by default** for both boundaries, on the server (`user_preferences.auto_advance` default true, existing rows flipped by V48) and in the app before its first preferences sync ([SliceFocusFE#600](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/600)); the dwell is the opt-in behaviour behind the Settings toggle. A parked boundary counted the wait nowhere and let two devices drift around the manual tap ([SliceFocusFE#598](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/598)). This is one function in four places: `FocusSession.autoAdvance()` on the backend, the renderer controller in Dart, `computePomodoroState` in the widget (which keeps its clamp and relies on the push for the walk, ADR-038), and the Swift menu bar.

### The six points

1. **The client renders; it does not count.** `FocusTimerController` holds the anchor and a render clock: a 1 s timer that recomputes `render(anchor, now)` and re-renders also on `resumed`, on window focus and after any anchor change. Nothing decrements, so a wake after suspension is right on the first frame without a sync. Local actions update the anchor optimistically with the same rules (pause: `pausedAt = now`; resume: `phasePausedSeconds += now − pausedAt`; manual advance with auto-advance off: next phase anchored at `now`; stop) and post to the server as today; the server's response or event then replaces the anchor. `hydrate` and `syncFromServer` collapse into "replace the anchor". Focus time is `completedWorkSeconds` plus the active elapsed of the current WORK phase, as `focusSeconds` computes it on the backend. The 2 s advance timer, the echo counter, the `hydrated*` getters and the per-device scheduled phase alert go away.
2. **Server time on the wire.** Every session response and every session event carries `serverTime`. The client keeps `offset = serverTime − localReceiveTime` and renders with `now = DateTime.now() + offset`. Clock skew between a device and the server stops reaching the display; the Live Activity keeps the device clock, since its anchors arrive from the server and the residual skew of an NTP-synced phone is under a second.
3. **Boundary alerts come from the server.** The Cloud Tasks webhook already fires at every boundary (ADR-038). After the advance (auto-advance on) or at the dwell boundary (off) it sends the FCM phase notification to all of the user's devices, honours `notificationEnabled`, sends nothing for a paused, completed or cancelled session, and never alerts the same `(sessionId, phaseStartedAt)` twice across the webhook and a `GET /active` catch-up. The app's local `zonedSchedule` alert is removed, except for a session whose start the server has not confirmed (the offline case), where the device keeps one local alert for the current phase. This is what delivers "break ended, go back to your computer" on a locked phone, and what stops stale alerts after a stop elsewhere.
4. **The Live Activity has one writer.** The app creates the activity once per session with the server's anchor (that is how it obtains the push token) and never updates it locally. Every change (pause, resume, advance, boundary, auto-advance toggle, end) reaches the widget as the server's APNs push. Lock-screen buttons keep their deep link into the app, which posts to the server; the server's push updates the widget. Every push token event is forwarded, deduplicated by value, so a token iOS rotates mid-activity reaches the backend.
5. **Every state change reaches every device, including server-originated ones.** The `GET /active` catch-up broadcasts `SESSION_ADVANCED` like the webhook does; `POST /active/advance` carries the boundary the client expects (`expectedLifecycleState`, `expectedPhaseStartedAt`), runs the catch-up first and is a no-op when the server has already moved; pause, resume and token registration catch up first; the session event carries the full anchor and is sent after commit. Under ADR-061 the data-only sync push for the `session` collection also fires for these server-originated changes, with no origin device to exclude. A hidden, menu-bar-resident Mac with a RUNNING or PAUSED session keeps a sync path while hidden: a 60 s `GET /active` poll under ADR-060, the sync push plus the same poll as fallback under ADR-061. A session lasts at most 8 h, so the poll is bounded.
6. **The macOS menu bar renders from the anchor in Swift.** Dart sends the anchor once per state change over the existing channel; Swift renders the string with a 1 s main-run-loop timer, walks boundaries when auto-advance is on, dwells when off, and re-renders on wake. One channel call per change instead of one per second, immune to a throttled isolate, and the same formula as everything else.

What stays: the Pomodoro formula is still evaluated on the client (ADR-037's insight), Cloud Tasks still triggers the widget's re-render at each boundary (ADR-038), the auto-advance setting is still the server's (SliceFocus#434), the offline action queue still replays completions (ADR-027/049), and an offline-started session still renders locally from its own anchor until the start reaches the server.

## Alternatives Considered

1. **Keep the countdown and add heuristics.** The history above is this option applied five times. Every referee between the local counter and the server's anchor (the "local ahead" block, the echo counter, the 2 s timer, the orchestrator's re-anchor) was correct for the case it was written for and wrong for the next one. Rejected: the number of owners is the defect.
2. **Fix the Live Activity only (one writer), keep the rest.** Removes the gap the founder saw on 1.6.1+35, but the app and the Mac still drift against the server and each other, the phone still gets no break-ended alert, and the hidden Mac still freezes. Rejected: it fixes one of the four owners.
3. **Server tick over the socket.** The server pushes `remainingSeconds` every second and clients only display it. Sub-second agreement while connected, nothing while suspended or hidden, and a per-client cost that ADR-060 and ADR-061 exist to remove. Rejected.
4. **Let the widget walk phases itself** (the pre-MER-358 session walk). The widget cannot re-render without a push (ADR-038), so it would compute the right phase and show the old one; and it would be a fifth implementation of the state machine. Rejected.
5. **Client-authoritative with the server as a log** (ADR-011 strengthened: the device that started the session owns it, the server records). The founder's requirement is the opposite, and the owning device is the one iOS suspends. Rejected.

## Consequences

### Easier

- One formula, one anchor, one owner. The app, the lock screen, the Mac menu bar and the server agree to within the push latency, and a wake after a lock or a sleep is correct on the first frame.
- The phone gets every boundary alert while locked, and alerts stop when the session is stopped anywhere.
- The code that refereed two models goes: the countdown and its decrement, the "local ahead" heuristics, the echo counter, the 2 s advance timer, the orchestrator's anchor bookkeeping and re-anchor, the per-tick menu-bar channel call, the per-device scheduled alert.
- ADR-061 gets simpler: with phase changes computed from the anchor, a sync push is only ever "something changed, refetch the session", including for server-originated changes.

### Harder

- Lock-screen pause and resume wait for a server round-trip and an APNs push instead of a local update; sub-second on Wi-Fi, visible on a poor connection.
- Offline sessions lose boundary alerts beyond the current phase, and an offline-started session cannot be followed on another device until the start reaches the server; both were true before, less visibly.
- The widget and the Swift menu bar carry the shared formula; a change to the Pomodoro rules now touches Java, Dart and Swift (it already touched Java, Dart and the widget).
- A hidden Mac with an active session polls once a minute; bounded by the 8 h cap, and replaced by the sync push under ADR-061.

### Rollout

In this order, each step shippable on its own:

1. Merge SliceFocusFE#565 (MER-559, the FE half of SliceFocus#434) and close SliceFocusFE#564 with it.
2. Backend, independent of each other: [SliceFocus#442](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/442) (`serverTime`, full anchor on the event, broadcast after commit), [SliceFocus#443](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/443) (FCM phase alert from the webhook), [SliceFocus#444](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/444) (catch-up broadcast, idempotent advance, catch-up before pause, resume and token registration, whole-second boundaries), [SliceFocus#445](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/445) (end the session at the 8 h cap). Then `Promote Image to Production`, which also carries the #434 half.
3. Front end: [SliceFocusFE#567](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/567) (renderer controller and server-time offset; blocked by #559, SliceFocus#442 and #444), then [SliceFocusFE#568](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/568) (one-writer Live Activity, token re-registration) and [SliceFocusFE#569](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/569) (menu bar from the anchor in Swift, poll while hidden with an active session), both blocked by #567.
4. Then the ADR-061 epic (SliceFocus#438), amended as in point 5.

### Verification

The founder's scenario is the acceptance test, on dev with the build from step 3:

| Step | Expected |
|---|---|
| Start on the Mac, open the phone, lock it | lock screen, phone app, Mac window and menu bar within a second of each other |
| Lock through two boundaries | the lock screen moves to break and back to work by itself; the phone gets a notification at each boundary |
| Unlock and open the app | the app shows the lock screen's phase and time; locking again changes nothing |
| Stop or pause from the phone | the Mac menu bar reflects it within the push latency, or within a minute with the window closed |
| Close the Mac window, sleep the Mac, wake it | the menu bar reads the right time at once |

ADR-011 is marked Superseded; ADR-037, ADR-038 and ADR-061 carry an amendment line pointing here.
