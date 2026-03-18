# ADR-013: Explicit CORS configuration per environment profile

**Status:** Accepted
**Date:** 2026-03-16
**Deciders:** Mert Ertugrul

## Context

The application had no explicit CORS configuration, relying on Spring Security's restrictive defaults. As the frontend deployed to different environments, CORS needed to be configured explicitly to allow legitimate cross-origin requests while blocking unauthorized origins.

## Decision

Add a CorsConfigurationSource bean in SecurityConfig with allowed origins configured per Spring profile: dev allows localhost:*, test/prod allow specific frontend domains. Allow standard headers plus X-Request-Id for correlation ID tracing. Allow credentials for authenticated STOMP WebSocket connections.

## Alternatives Considered

- **Permissive allow-all CORS policy**: Rejected for security reasons.
- **Proxy-based CORS handling at load balancer level**: Adds infrastructure complexity and makes local development harder.

## Consequences

- Frontend can make cross-origin requests to the correct backend environment
- WebSocket STOMP connections work across origins
- Adding new frontend domains requires updating the profile-specific configuration
- Correlation ID header (X-Request-Id) is explicitly allowed

*Related: MER-255, MER-252, MER-271*
