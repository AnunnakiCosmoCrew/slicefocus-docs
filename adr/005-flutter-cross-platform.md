# ADR-005: Flutter for cross-platform frontend

**Status:** Accepted
**Date:** 2026-02-15
**Deciders:** Mert Ertugrul

## Context

The app needs to run on macOS, iOS, and Android from a single codebase. It includes a custom radial pie chart (Canvas-based), real-time WebSocket updates, and platform-specific features (macOS menu bar timer, iOS lock screen timer).

## Decision

Use Flutter with Dart for the frontend across all platforms.

## Alternatives Considered

- **React Native**: Large ecosystem but custom Canvas rendering (pie chart) is harder. Bridge overhead for native features. JavaScript tooling complexity.
- **Native per platform (Swift + Kotlin)**: Best performance and native feel, but 3x the development effort for a solo developer. Not viable.
- **Kotlin Multiplatform**: Promising but immature for UI (Compose Multiplatform). macOS support still experimental.
- **Web app (PWA)**: No native menu bar, limited background timer support, no lock screen integration.

## Consequences

- Single codebase for macOS, iOS, Android
- Custom `CustomPainter` for pie chart works identically across platforms
- Platform channels needed for native features (macOS NSStatusItem, iOS lock screen)
- Dart ecosystem is smaller than JS/Swift but sufficient for this use case
- Hot reload significantly speeds development
- App size is larger than native (~30-50MB) but acceptable for a productivity app
