# ADR-017: Spring Cache with ConcurrentMapCache for hot data

**Status:** Accepted
**Date:** 2026-03-16
**Deciders:** Mert Ertugrul

## Context

User preferences were loaded from the database on every focus session start, and the soundscape catalog (static data) was re-queried on every request. These repetitive queries added unnecessary database load and latency.

## Decision

Add spring-boot-starter-cache with @EnableCaching. Apply @Cacheable to user preferences (cached per user, evicted on update) and soundscape catalog (static data, long TTL). Use simple in-memory ConcurrentMapCache, appropriate for the single-instance Cloud Run deployment.

## Alternatives Considered

- **Redis/Memcached**: Adds infrastructure complexity for a single-instance deployment.
- **No caching**: Rejected due to unnecessary DB round-trips for frequently accessed data.
- **HTTP-level caching (ETags)**: Insufficient for server-side service calls.

## Consequences

- Reduced database load for hot paths
- Cache invalidation on preference updates prevents stale data
- Would need migration to Redis for multi-instance scaling
- Minimal configuration overhead with Spring Boot's cache abstraction

*Related: MER-258*
