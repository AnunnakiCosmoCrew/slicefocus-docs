# ADR-002: Contract-first API development with OpenAPI

**Status:** Accepted
**Date:** 2026-02-16
**Deciders:** Mert Ertugrul

## Context

Need a consistent approach to API development that keeps backend and frontend in sync and reduces manual boilerplate.

## Decision

Adopt contract-first development: define the API in `slicefocus-api.yaml` (OpenAPI 3.0), then generate Java interfaces and model classes using `openapi-generator-maven-plugin` with custom Mustache templates.

## Alternatives Considered

- **Code-first with SpringDoc**: Generate spec from annotations. Faster to start but spec drifts from reality, harder to enforce consistency, and frontend can't work ahead of backend.
- **Manual DTOs**: Write request/response classes by hand. Duplication, no single source of truth, easy to diverge from docs.
- **GraphQL**: Flexible querying but overkill for this domain (simple CRUD + Pomodoro state machine). Adds complexity without clear benefit.

## Consequences

- Single source of truth for API contract (`slicefocus-api.yaml`)
- Frontend can develop against the spec before backend is ready
- Custom Mustache templates enable features like `x-java-record` and `x-java-instant`
- Generated code must never be edited directly
- Any API change requires: edit spec → regenerate → implement
- Slight learning curve for custom OpenAPI extensions
