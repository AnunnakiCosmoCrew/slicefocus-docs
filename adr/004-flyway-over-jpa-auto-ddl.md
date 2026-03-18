# ADR-004: Flyway migrations over JPA auto-DDL

**Status:** Accepted
**Date:** 2026-02-18
**Deciders:** Mert Ertugrul

## Context

Need a database schema management strategy for PostgreSQL (Neon) across dev, test, and prod environments.

## Decision

Use Flyway SQL migrations for all schema changes. JPA DDL auto is set to `validate` — it checks entity mappings match the schema but never modifies it.

## Alternatives Considered

- **JPA `hibernate.ddl-auto=update`**: Convenient for prototyping but dangerous in production. Can't rename columns, drops data, no rollback, no audit trail. Many production incidents in the industry caused by this.
- **Liquibase**: Similar to Flyway but XML-based format is more verbose. Flyway's plain SQL is simpler and more transparent.
- **Manual SQL scripts**: No version tracking, no ordering guarantees, no migration state management.

## Consequences

- Every schema change requires a new migration file (`V{N}__{description}.sql`)
- Full audit trail of every schema change in git
- Migrations run automatically on app startup
- `validate` mode catches entity/schema drift immediately
- Slightly more work than auto-DDL but prevents data loss accidents
- H2 in-memory for tests runs the same migrations ensuring test/prod parity
