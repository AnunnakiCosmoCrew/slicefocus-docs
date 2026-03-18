# ADR-030: Flutter Environment Configuration via --dart-define

**Status:** Accepted
**Date:** 2026-03-18
**Deciders:** Mert Ertugrul

## Context

The app needs to target different backend environments (local, dev, prod) without rebuilding. Runtime config files (e.g., `.env` parsing) add startup latency and error handling complexity. Flavor-based builds multiply CI complexity by requiring separate build configurations per environment.

## Decision

Use `--dart-define` compile-time constants (`ENV`, `API_BASE_URL`, `SLICE_API_PATH_PREFIX`) for environment switching. `AppConfig` reads values from `String.fromEnvironment` at startup.

Resolution order: explicit `API_BASE_URL` > `ENV` preset > platform default (localhost). The Android emulator auto-detects `10.0.2.2` for localhost connectivity to the host machine.

Preset environments:

| ENV   | API Base URL                                                              |
|-------|---------------------------------------------------------------------------|
| local | `http://localhost:8080` (Android: `http://10.0.2.2:8080`)                 |
| dev   | `https://slicefocus-backend-dev-743920918625.europe-west3.run.app`        |
| prod  | `https://slicefocus-backend-743920918625.europe-west3.run.app`            |

## Alternatives Considered

- **`.env` file parsing at runtime**: Adds startup latency and requires error handling for missing/malformed files.
- **Flutter flavors**: Multiplies build variants and CI configurations (one per environment per platform).
- **Firebase Remote Config**: Adds a network dependency at startup; unsuitable for the API base URL the app needs before any network call.

## Consequences

- Zero runtime overhead — values are compile-time constants inlined by the Dart compiler
- CI pipelines pass `--dart-define` flags for each target environment
- Developers only need to know two flags (`ENV` or `API_BASE_URL`)
- Cannot change environment without a rebuild (acceptable trade-off for simplicity)

*Related: MER-206*
