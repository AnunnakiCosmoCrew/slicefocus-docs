# ADR-016: Checkstyle for static code analysis with CI quality gates

**Status:** Accepted
**Date:** 2026-03-14
**Deciders:** Mert Ertugrul

## Context

The project had no static analysis or code quality enforcement in CI. As the codebase grew to 54 source files and 2,595 LOC, maintaining consistent code style relied entirely on developer discipline.

## Decision

Add maven-checkstyle-plugin with a custom checkstyle.xml configuration. Rules enforce: unused/redundant imports, equals/hashCode consistency, boolean simplification, string literal equality, need braces, modifier order, and 150-char max line length. Generated code is excluded. OWASP Dependency-Check was later added for security scanning.

## Alternatives Considered

- **SpotBugs alone**: Focuses on bug patterns rather than style consistency.
- **SonarQube**: Comprehensive but requires server infrastructure.
- **ESLint-style per-file configuration**: Rejected in favor of Maven plugin integration.

## Consequences

- Code style enforced automatically on every build
- New contributors get immediate feedback on style violations
- CI pipeline catches issues before merge
- OWASP scanning fails builds on HIGH/CRITICAL CVEs

*Related: MER-163, MER-164, MER-257*
