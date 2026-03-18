# ADR-010: FCM push notifications via Firebase Admin SDK

**Status:** Accepted
**Date:** 2026-03-08
**Deciders:** Mert Ertugrul

## Context

Users needed to be notified of focus session phase transitions (work to break, break to work, session completed) even when the app was in the background or the screen was locked. The app already used Firebase for authentication, making FCM a natural fit.

## Decision

Use Firebase Cloud Messaging (FCM) for push notifications. Backend integrates Firebase Admin SDK to send notifications, with device tokens registered via POST /api/devices endpoint. Notifications trigger on phase transitions, session completion, and slice-end warnings. Frontend uses firebase_messaging for both foreground and background notification handling.

## Alternatives Considered

- **Local-only notifications**: Insufficient for cross-device sync scenarios.
- **Third-party push services (OneSignal, Pusher)**: Would add another vendor dependency when Firebase was already in the stack.
- **APNs directly**: Rejected to avoid platform-specific server code.

## Consequences

- Single Firebase project handles auth, push notifications, and crash reporting
- Device token lifecycle management (registration, stale token cleanup) adds complexity
- Push notification rate-limiting prevents spamming users
- Requires Firebase project setup and service account credential management via Secret Manager

*Related: MER-148, MER-149, MER-150, MER-179, MER-180, MER-184*
