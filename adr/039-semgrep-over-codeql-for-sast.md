# ADR-039: Semgrep over CodeQL for SAST

**Status:** Accepted
**Date:** 2026-04-16
**Deciders:** Mert Ertugrul

## Context

CodeQL was added as the SAST tool during v1.0 (see ADR-016 era CI hardening) via GitHub Advanced Security (GHAS) on the private repository. It ran on every PR and push to `main` and was a required status check.

GHAS Code Security is billed at ~$30 per active committer per month. Across one month of v1.1 development:

- CodeQL surfaced findings on only 1 of 48 merged BE PRs (PR #277).
- The codebase relies on patterns that pre-empt the vulnerability classes CodeQL excels at finding: JPA repositories (no SQL string concatenation), Spring Security defaults (no homegrown crypto), scoped queries (`findByIdAndUserId` — IDOR pre-empted at the framework level), and a small attack surface (no file uploads, no admin panel, stateless API).
- CodeQL builds took 3-5 minutes per PR, slowing down merge cadence.

The cost of $360/year for one committer is meaningful for a solo-developer side project, especially when the underlying value (zero high-severity findings in a month of active development) is low.

## Decision

Disable GHAS Code Security and replace CodeQL with Semgrep's free tier as the SAST tool.

The Semgrep workflow runs `semgrep ci` in the official `semgrep/semgrep` Docker image with the rule packs `p/security-audit`, `p/secrets`, `p/java`, and `p/owasp-top-ten`. Initial rollout uses `--severity=ERROR` so only high-severity findings block PRs; lower-severity findings still appear in the run log for review and can be promoted to blocking later.

The Semgrep workflow becomes the new required status check (`scan`) on the `main` branch protection rules, replacing the previous `analyze` (CodeQL) check.

Snyk is planned separately for dependency scanning (free tier, GitHub App), and may eventually supersede the monthly OWASP dependency-check workflow.

## Alternatives Considered

- **Keep CodeQL with GHAS** — Rejected on cost vs measured value. The pattern depth CodeQL offers (multi-step taint tracking, complex injection chains) is not exercised by this codebase's architecture.
- **Snyk Code (free tier) as primary SAST** — Considered. Strong product, easier setup via GitHub App, but Semgrep's rule packs are more transparent and locally inspectable, and its CLI fits better with the existing workflow-file-driven CI.
- **SonarCloud** — Free for public repos only; private requires paid. Strong on code quality but secondary on security-specific findings.
- **Make the repo public to get free CodeQL** — Rejected for now; the repo contains internal architecture choices and an active product. Could be revisited.
- **No SAST at all** — Rejected. Even if findings are rare, having a continuous low-friction safety net is worth ~$0/month and ~30 seconds of CI time.

## Consequences

- **Cost:** ~$30/month → $0/month (~$360/year saved).
- **Speed:** CI scan time drops from 3-5 min (CodeQL) to ~30 seconds (Semgrep).
- **Coverage trade-off:** Semgrep is pattern-based; CodeQL's taint-tracking depth is genuinely better at chained vulnerability discovery. For this codebase's architecture, the gap is estimated at 10-15% of theoretical findings — and most of that gap is in vulnerability classes that don't apply here.
- **Security tab integration lost:** Without GHAS, Semgrep findings cannot be uploaded to the GitHub Security dashboard via SARIF. Findings appear only in the workflow run logs. If centralized triage becomes important, revisit by either re-enabling GHAS or using Semgrep AppSec Platform (also paid).
- **Branch protection updated:** Required check renamed from `analyze` to `scan`. Documented in `CLAUDE.md`.
- **Reversibility:** Re-enabling GHAS and restoring `codeql.yml` is straightforward (~1 hour) if the trade-off proves wrong.

*Related: [SliceFocus#298](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/298), [PR #299](https://github.com/AnunnakiCosmoCrew/SliceFocus/pull/299), ADR-016 (Checkstyle), ADR-023 (PR validation)*
