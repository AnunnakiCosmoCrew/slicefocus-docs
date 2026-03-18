# ADR-012: Bucket4j rate limiting with per-user and per-IP tiers

**Status:** Accepted
**Date:** 2026-03-16
**Deciders:** Mert Ertugrul

## Context

The API had no rate limiting, making it vulnerable to brute force attacks, DDoS, and abuse in production. With Cloud Run auto-scaling, unconstrained request rates could also cause cost spikes.

## Decision

Implement rate limiting using Bucket4j with a Spring Boot OncePerRequestFilter. Configure per-user limits for authenticated endpoints (100 reads/min, 30 writes/min) and per-IP limits for public endpoints (20 req/min). Return 429 Too Many Requests with Retry-After header. Frontend handles 429 responses gracefully with user-friendly messaging and automatic retry.

## Alternatives Considered

- **Cloud Run-level rate limiting**: Too coarse-grained (no per-user differentiation).
- **API gateway rate limiting (Apigee)**: Over-engineered for a solo-developer project.
- **No rate limiting**: Rejected as a production readiness concern.

## Consequences

- Protection against abuse and cost spikes
- Requires frontend to handle 429 responses
- Simple in-memory token buckets work for single-instance but would need Redis for multi-instance
- Retry-After header enables intelligent client-side backoff

*Related: MER-251, MER-270*
