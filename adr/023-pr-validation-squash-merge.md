# ADR-023: PR validation workflow with squash-merge-only policy

**Status:** Accepted
**Date:** 2026-03-14
**Deciders:** Mert Ertugrul

## Context

The project audit revealed that CI had no PR validation workflow — all 50 analyzed PRs had 0% human review and 5-20 minute merge times. Broken code could reach main undetected. The project also had 60+ stale branches.

## Decision

Add branch protection rules on main: squash merge only, PR required for all changes (enforced for admins), all review conversations must be resolved before merge, force push and branch deletion blocked, head branches auto-deleted after merge. CI quality gates run on PR branches before merge is allowed. Trunk-based development with short-lived feature branches.

## Alternatives Considered

- **Merge commits**: Rejected to keep a clean linear history.
- **Rebase merge**: Rejected because squash merge better groups related commits.
- **No branch protection**: Rejected after audit flagged it as a key gap.

## Consequences

- Cleaner git history with one commit per feature/fix
- Forces all code through validation before reaching main
- Adds friction to merge process (must wait for CI) but prevents regressions
- Auto-deletion of head branches prevents stale branch accumulation

*Related: MER-163, MER-164*
