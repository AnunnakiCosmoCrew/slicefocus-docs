# ADR-059: Focus Shield on macOS: Safari extension for websites, interruption for apps

**Status:** Proposed
**Date:** 2026-10-15
**Deciders:** Mert Ertugrul

## Context

[ADR-057](057-focus-shield-screen-time-app-and-website-blocking.md) shipped Focus Shield on iOS through the Screen Time API, and its 2026-10-15 addendum (spike FE #526) recommended keeping macOS out of scope, because no Apple API lets a third-party app block other apps on macOS and the Screen Time types are `@available(macOS, unavailable)`. The founder now wants Mac coverage (epic FE #536). This ADR **supersedes that recommendation** and settles how the Mac version works within what the platform allows. The addendum's SDK findings still stand; this ADR accepts their limits.

The Mac app is a sandboxed native Flutter build, so it can neither terminate other apps nor install a system-wide network filter without a restricted entitlement.

## Decision

### 1. Mechanisms

- **Websites: a Safari web extension** using `declarativeNetRequest` **dynamic rules**. Safari supports this from 15.4 (macOS 12.3), and the app's deployment target is 12.0. The Mac target is raised to 12.3 rather than letting the extension declare its own minimum, so the app never shows a Focus Shield switch on a macOS where it cannot work (the epic's extension issue does the bump).
- **Apps: interruption, not blocking.** The app observes `NSWorkspace.didActivateApplicationNotification`; when a selected app comes to the front during a work phase it covers it with a shield window. It does not terminate or hide the app (sandbox). The UI says "interrupted", never "blocked".
- **Accepted cost.** Forum reports put a dynamic-rule update at 6 to 8 seconds before Safari honours it. That is acceptable because rules change only at work-phase boundaries.
- **Alternative not taken: a content script** that hides or redirects pages. It reacts faster but needs "read and change all websites" on every page, and the user can trivially switch it off per site; `declarativeNetRequest` needs no page access. If the rule-update delay proves unacceptable in practice, the content script is the fallback.

### 2. Shared-state contract (Flutter app and extension)

Stored in the app group as one JSON document:

```json
{ "active": true, "untilEpochMs": 1760000000000, "domains": ["news.example"], "version": 3 }
```

- `active`: whether rules should be installed. `untilEpochMs`: planned end of the work phase, or null. `domains`: block list. `version`: monotonically increasing, written by the app on every change.
- The extension reads it through `browser.runtime.sendNativeMessage` on start-up and on a `browser.alarms` tick of at most 60 s, and re-applies rules only when `version` changed.
- **The extension lifts its rules on its own once `untilEpochMs` passes**, even if the app was killed. Same idea as the iOS DeviceActivity fallback (FE #525): enforcement must never outlive the session because the app died.
- **App-group id: to be confirmed in FE #539, not assumed.** The iOS id (`group.com.slicefocus.app`) is the iOS convention and must not be copied across. The working choice is the team-prefixed form `$(TeamIdentifierPrefix)com.slicefocus.app`, the macOS convention for sandboxed apps. Whether `group.`-prefixed ids are also accepted for our distribution channel depends on the macOS version, and must be checked against Apple's current App Groups documentation and a real signed build before the entitlement is committed. This ADR did not verify it.

### 3. Storage

The Mac block list and enabled flag are **device-local**: `SharedPreferences` on the Dart side, with the domain list mirrored to the app group for the extension. They are **not server-synced**, because iOS cannot consume an app or domain list written on a Mac (iOS selections are opaque, device-bound tokens, per ADR-057).

### 4. Capabilities

`FocusShieldService` stays one class over the `com.slicefocus.focusshield` channel and exposes `FocusShieldCapabilities` so the UI never branches on the platform itself:

| | iOS | macOS | other |
|---|---|---|---|
| `apps` | `enforced` | `interrupt` | `none` |
| `websites` | `system` | `safariExtension` | `none` |
| `selectionEditor` | `nativePicker` | `inApp` | n/a |
| `requiresAuthorization` | true | false | false |

`isSupported` is true on iOS and macOS. On macOS `requestAuthorization` and `authorizationStatus` report `approved` without touching the channel, and `presentPicker` is not a channel call (the editor is Flutter). The orchestrator is unchanged.

## Alternatives Considered

- **Keep macOS out of scope** (the ADR-057 addendum). Honest about the platform, but gives Mac users nothing; superseded by the founder's call.
- **Mac Catalyst or `NEFilterDataProvider`.** Rejected in the addendum for sandbox and entitlement reasons; nothing has changed.
- **Content-script blocking.** See above.
- **Sync the Mac list to the server.** Rejected: iOS cannot use it.

## Consequences

- Mac users get real website blocking in Safari only; other browsers are not covered, and the user can disable the extension in Safari settings. The copy must say so.
- Apps are interrupted, not blocked, which is weaker than iOS. This is a product call, flagged for founder review.
- A Safari extension target, an app group and a signing change enter the Mac build; each is its own issue in the epic.
- The Mac deployment target rises from 12.0 to 12.3.
- Re-check the addendum's SDK availability lines on each new Xcode major; if the Screen Time types gain macOS availability, revisit this ADR.
