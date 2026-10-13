# ADR-057: Focus Shield — block selected apps and websites during a focus session (Screen Time API, iOS)

**Status:** Proposed
**Date:** 2026-10-13
**Deciders:** Mert Ertugrul

## Context

A focus session today changes nothing outside SliceFocus: the user can open Instagram or a news site mid-cycle and the timer keeps counting. The founder asked for a feature that blocks social media, or a user-chosen set of apps and websites, while a focus session runs (FE epic #522).

On Apple platforms the only sanctioned way for a third-party app to block other apps and websites is the **Screen Time API**: `FamilyControls` (authorization and the `FamilyActivityPicker`), `ManagedSettings` (the `ManagedSettingsStore` whose `shield` settings block apps and web domains system-wide), and `DeviceActivity` (schedules and a monitor extension). Constraints that shape the design:

- **Platform reach.** Apple lists `FamilyControls` for iOS 15+, iPadOS 15+ and Mac Catalyst 15+. There is no native macOS API, and the SliceFocus Mac app is a native Flutter macOS build, not Catalyst. Developer-forum reports also say Catalyst authorization fails under the sandbox.
- **Opaque selection.** The picker returns `ApplicationToken` / `WebDomainToken` / `ActivityCategoryToken` values. The app never learns which apps were chosen; tokens are only meaningful on the device that produced them. They cannot be synced through the backend.
- **Entitlement.** `com.apple.developer.family-controls` is free for development builds, but App Store distribution requires Apple's approval of a separate "Family Controls (Distribution)" request, with a described use case. Approval times vary from days to months.
- **Shields outlive the process.** A shield written to the store stays in force after the app is killed, until something clears it.
- **Existing shape.** The app already bridges native features with hand-rolled `MethodChannel`s (`com.slicefocus.calendar`, `com.slicefocus.reminders`, `com.slicefocus.menubar`) and mirrors timer state to native surfaces through small orchestrators that subscribe to `FocusTimerController.stateStream` (menu bar, lock-screen Live Activity, soundscape).

## Decision

Ship **Focus Shield** as an iOS-only, device-local feature built on the Screen Time API, wired like the existing native mirrors.

1. **Native bridge.** A `FocusShieldBridge` in the iOS Runner serves the `com.slicefocus.focusshield` `MethodChannel`: `authorizationStatus`, `requestAuthorization` (individual, not child), `presentPicker` (a `FamilyActivityPicker` in a `UIHostingController` sheet), `selectionSummary`, `clearSelection`, `applyShield`, `clearShield`. The `FamilyActivitySelection` is persisted as JSON in `UserDefaults` on the device; Dart only ever sees counts (apps, categories, websites), never the tokens.
2. **Shield lifecycle follows the timer.** A `FocusShieldOrchestrator` subscribes to `FocusTimerController.stateStream`. The shield is applied when the feature is enabled, a selection exists, and the timer is **running in a work phase**. It is cleared on short and long breaks, on pause, on completion, on idle, and when the user disables the feature. The orchestrator clears any stale shield at start-up when no session is running, so a crash or force-quit mid-session does not leave the device blocked after relaunch.
3. **Named store.** Shields are written to `ManagedSettingsStore(named: "focus")` and cleared with `clearAllSettings()` on that store, so SliceFocus never touches settings another app or Screen Time itself wrote.
4. **Settings UI.** A "Focus Shield" section in Settings, shown on iOS only: an on/off switch (turning it on requests Screen Time authorization; a denial reverts the switch) and a "Choose apps & websites" row that opens the picker and shows the selection summary. The enabled flag lives in `SharedPreferences` on the device; it is not synced to the backend because the selection it governs cannot be.
5. **Scope of the first release.** Work-phase-only blocking, no custom shield screen (the system "restricted" shield is used), no DeviceActivity schedule. Lifting the shield when the app is killed mid-session is a follow-up that adds a `DeviceActivityMonitor` extension to clear the store at the session's planned end (FE #525).
6. **Release gate.** The feature cannot ship to the App Store until Apple grants the distribution entitlement (FE #524, founder). Until then it works in development and TestFlight-internal builds only; the Settings section stays visible so the flow can be tested.

## Alternatives Considered

- **Network-level blocking (NEFilterDataProvider / VPN).** Rejected: content filters require supervised devices or the same Family Controls entitlement on iOS, and a local VPN is heavy, battery-hungry and would not block apps, only hosts.
- **Mac Catalyst build of the Mac app to get FamilyControls there.** Rejected for now: it would replace the native macOS Flutter target, and forum reports show Catalyst authorization failing in the sandbox. macOS parity is a research spike (FE #526), not a commitment.
- **Block during breaks too.** Rejected as the default: breaks are the moment a user is allowed to check their phone. A per-user option can be added later without changing the architecture.
- **Sync the selection to the backend.** Impossible: the tokens are device-bound and opaque, by Apple's design.
- **DeviceActivity schedule from day one.** Deferred: it needs a second app extension and app-group plumbing; the start-up stale-shield sweep covers the common crash case for a first release.
- **Custom shield UI (ShieldConfiguration extension).** Deferred: cosmetic, needs another extension target.

## Consequences

- Blocking is real and system-enforced on iOS: shielded apps show the system shield, shielded domains are blocked in Safari and WebKit views.
- The feature is iOS-only and device-local. The macOS app and the web build show nothing. Users must pick apps on each device.
- Shipping depends on an Apple approval that only the founder can request. The PR body for the feature marks it `Founder check: public`.
- SliceFocus takes on a Screen Time authorization prompt and a Family Controls entitlement, which App Review scrutinises; the use case (self-imposed focus blocking by the device owner) is the individual-authorization case, not parental control.
- A force-quit mid-session leaves the shield up until the next launch (or until the user disables it in iOS Screen Time settings). Accepted for the first release, tracked in FE #525.
- A new Swift source file and an entitlement key land in the Runner target; the Xcode project file gains the file reference, and the App ID gains the Family Controls capability on the next automatic-signing build.
