# ADR-031: Rejecting freezed/code-generation for Domain Models

**Status:** Accepted
**Date:** 2026-03-18
**Deciders:** Mert Ertugrul

## Context

As the number of domain models grew (DaySlice, FocusSession, TimerPreferences, UserPreferences, Soundscape), the team considered adopting `freezed` + `json_serializable` for automatic `copyWith`, `==`, `hashCode`, `fromJson`/`toJson` generation. A spike branch was created to evaluate the migration.

## Decision

Cancel the freezed migration. Maintain manual implementations of `fromJson`/`toJson`/`==`/`hashCode`/`copyWith` in domain models. The domain layer stays free of code-generation dependencies and `build_runner`.

## Alternatives Considered

- **freezed + json_serializable**: Auto-generates boilerplate but adds `build_runner` dependency, `.g.dart`/`.freezed.dart` files, and slower builds. Tightly couples domain models to code-gen annotations.
- **built_value**: More verbose than freezed, worse IDE support, same build_runner dependency.
- **equatable package**: Partial solution covering only `==`/`hashCode`; still requires manual serialization and `copyWith`.

## Consequences

- No `build_runner` in the build pipeline — faster builds and simpler CI
- Domain layer maintains zero external dependencies, consistent with Clean Architecture goals
- Developers must manually maintain equality and serialization methods (risk of drift on model changes)
- Current models are simple enough (5-10 fields each) that manual maintenance is manageable
- If model count or complexity grows significantly, this decision should be revisited

*Related: MER-246*
