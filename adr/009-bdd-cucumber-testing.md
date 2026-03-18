# ADR-009: BDD/Cucumber for component testing with enforced coverage thresholds

**Status:** Accepted
**Date:** 2026-03-05
**Deciders:** Mert Ertugrul

## Context

The project needed a testing strategy that verified API behavior end-to-end through the Spring Boot stack while also maintaining fast unit tests for domain logic. Coverage needed to be enforced automatically to prevent regressions as the codebase grew rapidly.

## Decision

Adopt a dual-layer testing strategy: JUnit 5 + Mockito unit tests for domain/service logic (Surefire, 80% threshold), and Cucumber BDD component tests for end-to-end API behavior (Failsafe, 70% threshold). Coverage thresholds are enforced by JaCoCo during `mvnw verify`. Shared API steps in ApiStepDefinitions.java and scenario state via ApiScenarioContext reduce duplication. A component-test Spring profile uses H2 in-memory database.

## Alternatives Considered

- **Integration tests with TestRestTemplate alone**: Lacked the readable specification format of Gherkin.
- **Single test layer**: Unit tests alone cannot catch wiring/serialization issues, and component tests alone are too slow for rapid domain logic iteration.

## Consequences

- Test LOC (5,003) is nearly 2x source LOC (2,595), with 258 unit tests and 90 BDD scenarios
- Coverage thresholds catch regressions automatically
- Dual-layer approach requires maintaining both test suites but catches different categories of bugs
- Milestone quality gates verify thresholds before milestone completion

*Related: MER-64, MER-143, MER-155, MER-164, MER-176*
