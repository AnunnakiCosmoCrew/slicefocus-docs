# Architecture Decision Records

This directory captures significant technical decisions for SliceFocus.

| ADR | Decision | Status | Date |
|-----|----------|--------|------|
| [001](001-url-path-api-versioning.md) | Use URL path versioning for API | Accepted | 2026-03-16 |
| [002](002-contract-first-openapi.md) | Contract-first API development with OpenAPI | Accepted | 2026-02-16 |
| [003](003-stateless-jwt-auth.md) | Stateless JWT authentication with Firebase | Accepted | 2026-02-20 |
| [004](004-flyway-over-jpa-auto-ddl.md) | Flyway migrations over JPA auto-DDL | Accepted | 2026-02-18 |
| [005](005-flutter-cross-platform.md) | Flutter for cross-platform frontend | Accepted | 2026-02-15 |
| [006](006-cloud-run-neon-postgres.md) | Cloud Run + Neon PostgreSQL for deployment | Accepted | 2026-02-22 |
| [007](007-websocket-stomp-realtime.md) | WebSocket with STOMP for real-time sync | Accepted | 2026-03-01 |

## How to add a new ADR

1. Copy [template.md](template.md) to `NNN-short-title.md`
2. Fill in the fields
3. Open a PR
4. Update this index
