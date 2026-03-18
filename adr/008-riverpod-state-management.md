# ADR-008: Riverpod for Flutter state management

**Status:** Accepted
**Date:** 2026-03-16
**Deciders:** Mert Ertugrul

## Context

The Flutter frontend used StatefulWidget + FutureBuilder + ValueNotifier for state management, leading to a 967-line home_page.dart with interleaved UI/state/business logic, manual setState + mounted checks, and 19 constructor params on SliceFocusApp from deep prop drilling. This made isolated testing of state transitions difficult and the codebase brittle.

## Decision

Migrate incrementally to Riverpod over 5 phases: (1) add Riverpod + migrate TimerPreferences, (2) migrate theme + soundscape, (3) migrate slice fetching with auth-gated providers, (4) migrate focus session state, (5) migrate connectivity + sync services. The freezed + json_serializable approach (MER-246) was cancelled in favor of keeping manual fromJson/toJson with Riverpod handling state management separately.

## Alternatives Considered

- **Keep StatefulWidget + FutureBuilder**: Rejected due to prop drilling and testability issues.
- **Bloc/Cubit**: Not considered given Riverpod's lighter boilerplate and better provider composition.
- **freezed code generation (MER-246)**: Cancelled to avoid build_runner complexity.

## Consequences

- Eliminated 19+ constructor params from SliceFocusApp, broke up the monolithic home_page.dart
- Made state transitions testable in isolation
- Required touching 100+ test dependencies during migration of slice fetching (MER-263 had to be split into 3 sub-issues)

*Related: MER-245, MER-261, MER-262, MER-263, MER-264, MER-265, MER-266, MER-267, MER-268*
