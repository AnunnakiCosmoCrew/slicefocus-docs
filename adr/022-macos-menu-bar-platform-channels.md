# ADR-022: Native platform channels for macOS menu bar timer

**Status:** Accepted
**Date:** 2026-03-13
**Deciders:** Mert Ertugrul

## Context

Users needed to track their focus session while working in other apps on macOS. A menu bar timer showing remaining time and cycle info is the standard macOS pattern. Flutter does not have a built-in API for macOS NSStatusItem.

## Decision

Use Flutter Platform Channels to bridge Dart and native Swift code. The Swift side creates an NSStatusItem with FlutterMethodChannel handlers (updateTimer, hideTimer). The Dart side wraps the channel in MenuBarTimerService. An orchestrator integrates it with the focus session lifecycle. For platforms without menu bar support, a NoopLockScreenTimerService is used.

## Alternatives Considered

- **Third-party Flutter packages**: None supported NSStatusItem text display (most focus on tray icons).
- **Separate native macOS companion app**: Over-engineered for displaying a text string.
- **Using only notifications**: Too intrusive for continuous status display.

## Consequences

- Native-feeling macOS integration with minimal Swift code
- Platform channel pattern is reusable for future native integrations
- Requires maintaining both Dart and Swift code
- Abstract LockScreenTimerService interface enables platform-specific implementations (iOS Live Activities, Android notifications)

*Related: MER-220, MER-221, MER-222, MER-205*
