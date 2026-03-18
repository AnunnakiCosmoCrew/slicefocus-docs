# ADR-006: Cloud Run + Neon PostgreSQL for deployment

**Status:** Accepted
**Date:** 2026-02-22
**Deciders:** Mert Ertugrul

## Context

Need a deployment target for the Spring Boot backend that is cost-effective for a solo developer, supports autoscaling, and requires minimal infrastructure management.

## Decision

Deploy to Google Cloud Run with Neon serverless PostgreSQL.

## Alternatives Considered

- **Cloud Run + Cloud SQL**: Cloud SQL has a minimum monthly cost (~$7/month for the smallest instance) even when idle. Neon's free tier is generous and scales to zero.
- **AWS ECS/Fargate**: More complex networking (VPC, subnets, security groups). Higher learning curve. No equivalent to Cloud Run's simplicity.
- **Kubernetes (GKE/EKS)**: Massive overkill for a single-service app. Operational overhead is not justified.
- **Heroku**: Simpler but more expensive at scale, less control over infrastructure, and vendor lock-in concerns.
- **Railway/Render**: Good developer experience but less mature for production workloads. Limited IAM/security controls.

## Consequences

- Scale-to-zero when no traffic — cost-effective for early stage
- OIDC + Workload Identity Federation — no static API keys
- Container-based deployment with multi-stage Docker builds
- Neon's serverless Postgres has cold-start latency on first query after idle
- Artifact promotion model: same image flows dev → test → prod
- Cloud Run min-instances=1 needed to eliminate cold starts for production
- Flyway migrations run on startup, which can slow cold starts
