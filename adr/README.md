# Architecture Decision Records

This directory captures significant technical decisions for SliceFocus.

## Index

### Foundation & Infrastructure
| ADR | Decision | Status | Date |
|-----|----------|--------|------|
| [005](005-flutter-cross-platform.md) | Flutter for cross-platform frontend | Accepted | 2026-02-15 |
| [006](006-cloud-run-neon-postgres.md) | Cloud Run + Neon PostgreSQL for deployment | Accepted | 2026-02-22 |
| [004](004-flyway-over-jpa-auto-ddl.md) | Flyway migrations over JPA auto-DDL | Accepted | 2026-02-18 |

### API & Backend Architecture
| ADR | Decision | Status | Date |
|-----|----------|--------|------|
| [002](002-contract-first-openapi.md) | Contract-first API development with OpenAPI | Accepted | 2026-02-16 |
| [001](001-url-path-api-versioning.md) | Use URL path versioning for API | Accepted | 2026-03-16 |
| [003](003-stateless-jwt-auth.md) | Stateless JWT authentication with Firebase | Accepted | 2026-02-20 |
| [026](026-multi-provider-oidc.md) | Multi-Provider OIDC (Google + Apple) with auto-provisioning | Accepted | 2026-03-18 |
| [024](024-db-unique-constraint-active-session.md) | DB-level unique constraint for single active session | Accepted | 2026-03-03 |
| [011](011-client-side-pomodoro-timer.md) | Client-side Pomodoro timer with server bookkeeping | Accepted | 2026-03-05 |
| [034](034-flexible-session-start-stop-tolerance.md) | Flexible session start/stop with ±5 minute tolerance | Accepted | 2026-03-25 |
| [017](017-spring-cache.md) | Spring Cache with ConcurrentMapCache for hot data | Accepted | 2026-03-16 |
| [021](021-audit-logging.md) | Audit logging for sensitive operations | Accepted | 2026-03-16 |
| [020](020-graceful-shutdown.md) | Graceful shutdown with active session awareness | Accepted | 2026-03-16 |
| [032](032-server-synced-preferences.md) | Server-synced user preferences with local fallback | Accepted | 2026-03-18 |
| [033](033-api-client-retry-rate-limit.md) | API client with retry logic and rate-limit awareness | Accepted | 2026-03-18 |

### Security & Observability
| ADR | Decision | Status | Date |
|-----|----------|--------|------|
| [012](012-rate-limiting.md) | Bucket4j rate limiting with per-user and per-IP tiers | Accepted | 2026-03-16 |
| [013](013-cors-per-environment.md) | Explicit CORS configuration per environment profile | Accepted | 2026-03-16 |
| [014](014-request-correlation-ids.md) | Request correlation IDs via X-Request-Id and MDC | Accepted | 2026-03-16 |
| [018](018-micrometer-prometheus.md) | Micrometer + Prometheus for application observability | Accepted | 2026-03-16 |
| [016](016-checkstyle-ci-quality-gates.md) | Checkstyle for static analysis with CI quality gates | Accepted | 2026-03-14 |

### Networking & Resilience
| ADR | Decision | Status | Date |
|-----|----------|--------|------|
| [007](007-websocket-stomp-realtime.md) | WebSocket with STOMP for real-time sync | Accepted | 2026-03-01 |
| [010](010-fcm-push-notifications.md) | FCM push notifications via Firebase Admin SDK | Accepted | 2026-03-08 |
| [036](036-apns-live-activity-push.md) | Direct APNs for Live Activity updates | Amended by ADR-037 | 2026-04-06 |
| [037](037-client-computed-live-activity-timeline.md) | Client-computed Pomodoro timeline for Live Activity | Amended by ADR-038 | 2026-04-07 |
| [038](038-cloud-tasks-phase-scheduling.md) | Cloud Tasks for Live Activity phase transition scheduling | Accepted | 2026-04-09 |
| [027](027-offline-action-queue.md) | Offline action queue with automatic retry | Accepted | 2026-03-18 |

### Frontend Architecture
| ADR | Decision | Status | Date |
|-----|----------|--------|------|
| [008](008-riverpod-state-management.md) | Riverpod for Flutter state management | Accepted | 2026-03-16 |
| [019](019-custom-painter-radial-chart.md) | CustomPainter for radial chart rendering | Accepted | 2026-02-24 |
| [022](022-macos-menu-bar-platform-channels.md) | Native platform channels for macOS menu bar timer | Accepted | 2026-03-13 |
| [015](015-programmatic-noise-generation.md) | Programmatic noise generation over bundled audio | Accepted | 2026-03-17 |
| [028](028-hybrid-soundscape-architecture.md) | Hybrid soundscape architecture (CDN + programmatic noise) | Accepted | 2026-03-18 |
| [029](029-shared-preferences-persistence.md) | SharedPreferences over SQLite for local persistence | Accepted | 2026-03-18 |
| [030](030-dart-define-env-configuration.md) | Flutter environment configuration via --dart-define | Accepted | 2026-03-18 |
| [031](031-rejecting-freezed-codegen.md) | Rejecting freezed/code-generation for domain models | Accepted | 2026-03-18 |

### Testing & CI/CD
| ADR | Decision | Status | Date |
|-----|----------|--------|------|
| [009](009-bdd-cucumber-testing.md) | BDD/Cucumber for component testing with coverage thresholds | Accepted | 2026-03-05 |
| [023](023-pr-validation-squash-merge.md) | PR validation workflow with squash-merge-only policy | Accepted | 2026-03-14 |
| [035](035-versioning-strategy.md) | Versioning strategy (FE, BE, API) | Accepted | 2026-03-26 |

### Process & Governance
| ADR | Decision | Status | Date |
|-----|----------|--------|------|
| [025](025-architecture-decision-records.md) | Use Architecture Decision Records for technical decisions | Accepted | 2026-03-18 |

## How to add a new ADR

1. Copy [template.md](template.md) to `NNN-short-title.md`
2. Fill in the fields
3. Open a PR
4. Update this index
