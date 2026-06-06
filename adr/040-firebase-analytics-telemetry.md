# ADR-040: Firebase Analytics for product telemetry

**Status:** Accepted
**Date:** 2026-06-05
**Deciders:** Mert Ertugrul

## Context

Through v1.1, SliceFocus had two observability signals and a blind spot:

- **Crash/error data** — Firebase Crashlytics on the client (production builds only).
- **Service health** — Micrometer + Prometheus on the backend (ADR-018), exposing aggregate counters (sessions started/completed/cancelled) and request performance.

What was missing was **product telemetry**: how the app is actually used. The backend counters answer "is the service healthy" but not "do users complete focus sessions, how many cycles per session, how long do they focus." Crashlytics only fires on failures. There was no funnel for the core action the whole app exists to support — completing a Pomodoro session.

The privacy policy already disclosed "anonymous, aggregated metrics for service health monitoring," but no client-side product-analytics path existed to populate them.

## Decision

Adopt **Firebase Analytics** (`firebase_analytics ^12.4.2`) on the Flutter client for lightweight product telemetry. The app already initializes Firebase (`firebase_core`, `firebase_messaging`, `firebase_crashlytics`, `firebase_auth`), so this adds an SDK to an existing integration rather than a new vendor.

Two events are tracked:

- **`app_open`** — Firebase's built-in automatic event (no custom code).
- **`timer_session_completed`** — custom event logged when a Pomodoro session completes, with non-PII parameters `completed_cycles` and `total_focus_minutes`.

The custom event lives in [`lib/src/core/telemetry.dart`](https://github.com/AnunnakiCosmoCrew/SliceFocusFE) and is deliberately constrained:

- **Silenced in debug builds** (`kDebugMode` guard) so development traffic does not pollute the dashboard.
- **No-ops safely when Firebase is not initialized** (e.g. widget tests) — wrapped in a `try/catch` after a `Firebase.app()` liveness check.
- **No personally identifiable information** in event parameters.

Scope is the **client only** — the backend gains no analytics SDK (it keeps Micrometer/Prometheus per ADR-018). This was shipped under MER-264 in the v1.2 cycle.

## Alternatives Considered

- **Amplitude / Mixpanel** — Purpose-built product analytics with richer funnels and retention cohorts. Rejected: another vendor + SDK + account to manage, and a paid tier looms as volume grows. Overkill for a solo side project that needs two events, not a behavioral-analytics suite.
- **Self-hosted (PostHog)** — Full ownership of data. Rejected: operational burden (another service to run and secure) that defeats the "lightweight" goal.
- **Backend-only Micrometer counters (status quo)** — Already present via ADR-018 and good for service health. Rejected as *sufficient*: server-side aggregates lack client context (which platform, session shape) and can't capture client-only events like `app_open`. Kept as complementary, not a replacement.
- **No product analytics** — Rejected. Shipping a productivity app with zero visibility into whether its core loop (complete a session) is used is flying blind.

## Consequences

- **Cost:** $0 — Firebase Analytics is free and reuses the existing Firebase project.
- **Minimal surface:** One small file (`telemetry.dart`) and one custom event; the rest is Firebase's automatic collection.
- **Privacy disclosure required:** The privacy policy must list Firebase Analytics as a data processor and describe what it collects (usage events, and Firebase's app-instance identifier). Updated in v1.2 — see [`legal/privacy-policy.md`](../legal/privacy-policy.md).
- **App-instance identifier:** Even with no PII in event params, Firebase Analytics assigns a pseudonymous app-instance ID. This is standard for Firebase Analytics and is covered by Google's data processing terms; relevant for GDPR transparency.
- **Clean dashboard:** Debug-mode silencing means the dashboard reflects real users, not development runs.
- **Reversibility:** Removing the SDK and `telemetry.dart` is straightforward; no other code depends on it.

*Related: ADR-018 (Micrometer + Prometheus observability), ADR-010 (FCM / Firebase Admin SDK), MER-264. Client-side crash reporting via Crashlytics predates the ADR process and is documented in [`legal/privacy-policy.md`](../legal/privacy-policy.md).*
