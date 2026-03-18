# ADR-024: DB-level unique constraint for single active session per user

**Status:** Accepted
**Date:** 2026-03-03
**Deciders:** Mert Ertugrul

## Context

The Pomodoro domain invariant requires at most one active (RUNNING) focus session per user. Initially enforced only at the service layer, but under concurrency (rapid double-tap, race between devices), two sessions could be created before the service-layer check completed.

## Decision

Add a partial unique index on the focus_sessions table via Flyway: a unique constraint on user_id where status = 'RUNNING' (using a computed column active_session_user_id that is non-null only for running sessions). DataIntegrityViolationException is caught and mapped to 409 Conflict. Frontend handles 409 by cancelling the stale session and retrying.

## Alternatives Considered

- **Service-layer-only enforcement**: Insufficient under concurrency.
- **Pessimistic locking (SELECT FOR UPDATE)**: Adds contention.
- **Distributed locks (Redis)**: Overkill for the single-database architecture.

## Consequences

- Strongest possible enforcement of the single-active-session invariant
- Race conditions produce a clean 409 error instead of data corruption
- Frontend conflict recovery flow provides seamless UX
- Computed column pattern is slightly unusual but well-supported by PostgreSQL and H2

*Related: MER-87, MER-98, MER-94, MER-139*
