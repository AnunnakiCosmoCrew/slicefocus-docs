# ADR-021: Audit logging for sensitive operations

**Status:** Accepted
**Date:** 2026-03-16
**Deciders:** Mert Ertugrul

## Context

There was no tracking of sensitive operations like OAuth account linking, user preference changes, or session lifecycle events. For security compliance and debugging, an audit trail was needed.

## Decision

Create an audit_log table via Flyway migration with columns for id, user_id, action (enum), details (JSONB), ip_address, and created_at. Create an AuditService that logs events asynchronously (@Async) to avoid impacting request latency. Covers OAuth account linking, preference changes, and session lifecycle events.

## Alternatives Considered

- **Application-level logging only**: Makes querying historical audit data difficult.
- **External audit services**: Add vendor dependency.
- **Synchronous audit logging**: Rejected to avoid adding latency to user-facing operations.

## Consequences

- Full audit trail for sensitive operations queryable via SQL
- Asynchronous logging prevents performance impact
- JSONB details column allows flexible schema per action type
- Adds a database write per audited operation but @Async minimizes user-perceived latency

*Related: MER-254*
