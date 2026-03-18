# ADR-020: Graceful shutdown with active session awareness

**Status:** Accepted
**Date:** 2026-03-16
**Deciders:** Mert Ertugrul

## Context

Cloud Run instances can be terminated at any time (scale-down, deployment, maintenance). Without shutdown hooks, in-flight requests would be dropped, WebSocket connections severed, and active focus sessions left in an inconsistent state.

## Decision

Enable Spring Boot graceful shutdown (server.shutdown=graceful) with a 30-second timeout per shutdown phase. Add @PreDestroy hook in FocusSessionService to log active sessions on shutdown. Ensure WebSocket connections are cleanly closed during shutdown.

## Alternatives Considered

- **Immediate shutdown (Cloud Run default)**: Drops in-flight requests and breaks WebSocket connections.
- **External health check-based draining**: Spring Boot's built-in graceful shutdown covers the use case with less complexity.

## Consequences

- Clean request completion during deployments
- WebSocket clients receive proper disconnect frames
- Active sessions are logged for debugging but not forcefully completed (client handles reconnection)
- 30-second timeout balances between request completion and fast instance recycling

*Related: MER-260*
