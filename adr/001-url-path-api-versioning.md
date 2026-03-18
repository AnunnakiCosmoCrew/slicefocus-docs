# ADR-001: Use URL path versioning for API

**Status:** Accepted
**Date:** 2026-03-16
**Deciders:** Mert Ertugrul

## Context

The API needed a versioning strategy to manage breaking changes as the API evolves and clients depend on specific response shapes. The frontend was already being developed against specific endpoints.

## Decision

Use URL path versioning (`/api/v1/`) for all API endpoints.

## Alternatives Considered

- **Accept header versioning** (`application/vnd.slicefocus.v1+json`): More RESTful, but harder to test (can't paste URLs in browser), harder to route at load balancer level, and less visible in logs.
- **Query parameter versioning** (`?version=1`): Easy to miss, pollutes query params, caching complications.
- **No versioning**: Not viable for a client-facing API with mobile apps that can't be force-updated.

## Consequences

- All endpoints prefixed with `/api/v1/`
- Breaking changes require a new version prefix (`/api/v2/`) and a deprecation timeline
- Easy to test, document, and debug — version is visible in every URL and log line
- OpenAPI spec, SecurityConfig, and all tests must include the version prefix
- Slightly less "pure REST" than Accept header, but far more practical for a small team
