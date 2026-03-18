# ADR-014: Request correlation IDs via X-Request-Id header and MDC

**Status:** Accepted
**Date:** 2026-03-16
**Deciders:** Mert Ertugrul

## Context

Debugging production issues was difficult without a way to trace a request across log lines. With async operations (WebSocket, push notifications), correlating logs for a single user action was impossible.

## Decision

Add a OncePerRequestFilter that generates a UUID correlation ID per request, sets it in SLF4J MDC so all log lines include it, and returns it in the X-Request-Id response header. If the client sends an X-Request-Id, the backend preserves it for end-to-end tracing. The frontend generates and sends X-Request-Id on every outgoing request and stores it for bug reports.

## Alternatives Considered

- **Distributed tracing with OpenTelemetry/Jaeger**: Overkill for a single-service architecture.
- **Cloud Run request IDs**: Not propagated into application logs or back to the client.

## Consequences

- Every log line can be correlated to a specific request
- Frontend can include correlation IDs in bug reports
- Adds minimal overhead per request (UUID generation + MDC set/clear)
- Provides foundation for future distributed tracing

*Related: MER-252, MER-271*
