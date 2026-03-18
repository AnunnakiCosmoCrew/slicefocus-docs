# ADR-018: Micrometer + Prometheus for application observability

**Status:** Accepted
**Date:** 2026-03-16
**Deciders:** Mert Ertugrul

## Context

The backend only exposed a basic /actuator/health endpoint. No application metrics existed. Monitoring performance or setting alerts for traffic spikes was impossible.

## Decision

Add micrometer-registry-prometheus and expose /actuator/prometheus endpoint (secured, not public). Add custom business metrics: focus_sessions_started_total, focus_sessions_completed_total counters and active session gauges. Standard Spring Boot metrics (HTTP durations, JVM stats, DB pool) included automatically.

## Alternatives Considered

- **Cloud Run built-in metrics**: Provide infrastructure-level data but no application-level business metrics.
- **Datadog/New Relic APM**: Vendor lock-in and cost.
- **Custom logging-based metrics**: Harder to query and alert on.

## Consequences

- Application performance is observable via standard Prometheus endpoints
- Business metrics enable alerting on focus session patterns
- Prometheus endpoint is secured to prevent data leakage
- Foundation for Grafana dashboards or Cloud Monitoring integration

*Related: MER-253*
