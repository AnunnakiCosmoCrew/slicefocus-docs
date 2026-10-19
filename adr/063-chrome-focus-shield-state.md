# ADR-063: How the Chrome Focus Shield extension learns the shield state

**Status:** Accepted (founder, 2026-10-19: option A on [SliceFocusFE#577](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/577)). The push-wake question is open until follow-up 3 amends this ADR.
**Date:** 2026-10-19
**Deciders:** Mert Ertugrul
**Relates to:** [ADR-059](059-focus-shield-on-macos-safari-extension-and-app-interruption.md) (the Safari extension whose rules it reuses), [ADR-060](060-websocket-lifecycle-policy.md) (no always-on poll), [ADR-061](061-pull-plus-push-sync-replaces-persistent-websocket.md) (the sync push it wants to wake on), [ADR-062](062-server-owned-focus-timeline-clients-render.md) (the server-owned phase it renders)

## Context

The Safari extension reads app-group state through `sendNativeMessage` (ADR-059). Chrome native messaging needs a host manifest under `~/Library/Application Support/Google/Chrome/NativeMessagingHosts/`, which a sandboxed App Store app cannot write. The backend already exposes `GET /api/v1/focus-sessions/active`, session STOMP events and FCM data-only sync pushes (dev only until the next prod promotion).

## Decision

**Option A, hybrid: backend-driven state, extension signs in.**

- **State:** the extension reads `GET /api/v1/focus-sessions/active` and maps it to the Safari `getState` shape (`active`, `untilEpochMs`, `domains`, `version`). `shieldRules.js` (`decide`/`buildRules`) is reused unchanged. Shield is up only for `status=RUNNING` and `lifecycleState=WORK`.
  - **Phase end:** `remainingSeconds` is authoritative when present; otherwise `phaseStartedAt + workDurationMinutes + totalPausedSeconds`. The bare `phaseStartedAt + workDurationMinutes` is wrong after a pause. Whether the backend re-stamps `phaseStartedAt` on resume is **not verified here**; follow-up 4 must check it against the backend before the fallback is trusted.
  - **Clock skew:** `phaseStartedAt` is server time, so the offset is computed against the server clock (the response `Date` header), not the browser's `Date.now()`: `untilEpochMs = serverNow + remainingSeconds*1000 - (serverNow - browserNow)`, i.e. the deadline is expressed in browser time using the measured offset. The spike uses the local clock and does not do this yet.
  - **Read results:** `404`/`204`/no session -> shield off, lift the rules. `401` -> keep the rules until the stored phase end (the alarm still lifts them), mark the extension signed out and prompt for sign-in; a signed-out extension never extends a shield past the end it already knew. Network error or `5xx` -> same as offline.
- **Auth:** Firebase ID token, same as the apps. Google sign-in through `chrome.identity.launchWebAuthFlow` + `signInWithCredential`; email sign-in also works. Popup/redirect flows do not work in an MV3 service worker. Sign in with Apple has no clean extension flow, so Apple-only users link a Google or email login in Settings first. Token kept in `chrome.storage.local`.
- **Wake / cost (ADR-060):** no always-on poll. A `chrome.alarms` alarm lifts the rules at the phase end the extension already knows. Starting a phase should wake it from the FCM data-only sync push (SliceFocus#439) via device registration (`/api/v1/devices`), costing one REST read per session change. Whether an MV3 extension can receive that push (`chrome.gcm` or Web Push) is **not yet verified**; the first follow-up must prove it. Fallback if it cannot: a slow `chrome.alarms` tick (the spike uses 1 minute), which costs one small read per tick only while the browser runs.
- **Domain list:** `chrome.storage.sync` with an options page and presets. No backend change now. A backend-owned list shared with Safari is a later, separate issue.
- **Repo layout:** separate `slicefocus-chrome` repo (JS/MV3 build and Web Store CI share nothing with the Flutter CI; keeps `analyze-and-test` minutes untouched).
- **Needs:** Chrome Web Store developer account (US$5 one-time); the extension origin/OAuth client added to the Firebase project.

## Alternatives Considered

- **B. App-issued pairing token:** works for every sign-in method including Apple, but needs a new backend endpoint and token type (auth-model change, security tier). Revisit if Apple-only users turn out to be common.
- **C. Mac-app native messaging host:** Mac-only, manual manifest copy with an absolute path into the app bundle that breaks when the app moves, a helper executable inside an App Store app (review risk), no Windows/Linux/ChromeOS.

## Consequences

- Chrome works on any OS, independent of the Mac app, but only for signed-in users and only when online (offline keeps the applied rules and the phase-end alarm lifts them).
- Apple-only accounts need a linked login first.
- **Latency:** without a working push wake, a session started elsewhere is shielded up to one tick (about 1 minute) late, and a cancelled or early-stopped session leaves the rules applied until the next successful read (about 1 minute online). The Safari path does not have this gap. This is accepted for a first version and is the reason follow-up 3 exists.
- **Shield outliving its session:** if the extension is offline when the session is cancelled, the alarm lifts the rules only at the phase end it already knew, so the shield can last at most the remaining work phase. That bound is accepted (fail-closed). To avoid a trap, the extension gets a manual "lift shield now" control in its popup (scope added to follow-up 4); the shield is a self-chosen focus aid, not a security boundary.
- The block list is per browser until the backend list exists; it does not sync with Safari.
- Prototype: branch `spike/mer-577-chrome-prototype` (`prototypes/chrome-focus-shield/`). It proves the state mapping and rule reuse with a node test; it was **not** run in a live Chrome with a real account, and covers neither sign-in nor the push wake.

## Follow-up issues (epic [SliceFocusFE#576](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/576))

| # | Issue | Scope | Est. |
|---|---|-------|------|
| 1 | [SliceFocusFE#588](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/588) | `slicefocus-chrome` repo scaffold: MV3, build, lint, node tests, port `shieldRules.js` | 3 |
| 2 | [SliceFocusFE#589](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/589) | Extension auth: Google (`launchWebAuthFlow` + Firebase) and email sign-in, token refresh | 5 |
| 3 | [SliceFocusFE#590](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/590) | Spike: verify push wake (`chrome.gcm`/Web Push) with FCM data-only sync push and `/api/v1/devices` | 3 |
| 4 | [SliceFocusFE#591](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/591) | State sync: `/active` read, phase-end alarm, offline/401/404 behaviour, verify resume re-stamping and clock offset, manual "lift now" control, blocked page | 3 |
| 5 | [SliceFocusFE#592](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/592) | Options page: domain list in `chrome.storage.sync`, presets | 3 |
| 6 | [SliceFocusFE#593](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/593) | Chrome Web Store listing, privacy disclosure, release CI | 5 |
| 7 | [SliceFocusFE#594](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/594) | Settings: link Google/email login for Apple-only accounts (FE) | 3 |
| 8 | [SliceFocus#448](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/448) | Optional, backend: user-owned block list endpoint shared with Safari (SliceFocus) | 5 |
