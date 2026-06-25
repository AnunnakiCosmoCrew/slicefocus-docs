# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SliceFocus is a cross-platform time management app featuring a 24-hour radial schedule canvas and Pomodoro timer. It targets macOS, iOS, and Android from a single Flutter codebase, backed by a Java 21 / Spring Boot API on Google Cloud Run with Neon PostgreSQL.

**This repository (`slicefocus-docs`) contains project documentation only.** The app code lives in two separate repos.

## Repository Map

| Repo | Purpose | Push to main? |
|------|---------|---------------|
| `AnunnakiCosmoCrew/slicefocus-docs` (this repo) | ADRs, specs, roadmaps, audits, screenshots | Yes — direct push |
| `AnunnakiCosmoCrew/SliceFocus` | Spring Boot backend (Java 21, Cloud Run, Neon PostgreSQL) | No — branch + PR only |
| `AnunnakiCosmoCrew/SliceFocusFE` | Flutter frontend (macOS, iOS, Android) | No — branch + PR only |

## Git Workflow (Trunk-Based Development)

`main` is the single integration branch. All code changes land via short-lived feature branches and squash-merge PRs.

### Docs repo (this repo)

Documentation changes are committed and pushed directly to `main`. No branch or PR needed.

**Commit convention for docs changes:**

- Pure documentation (no linked issue): `docs: <description>` — e.g., `docs: ADR-040 adopt Drift for local persistence`
- Tied to a tracked GitHub Issue: `MER-N docs: <description>` — e.g., `MER-245 docs: add architecture diagram for push notifications`

### App repos (`SliceFocus` and `SliceFocusFE`)

**Never push directly to main** — always create a branch and PR, even for one-line changes.

#### Branch naming

| Repo | Pattern | Example |
|------|---------|---------|
| Backend | `mertyertugrul/mer-{N}-{slug}` | `mertyertugrul/mer-123-add-rate-limiting` |
| Frontend | `feature/mer-{N}-{slug}` | `feature/mer-290-live-activity-phase-advance` |

`{N}` is the GitHub Issue number (lowercase kebab slug).

#### Commit message format

`MER-{N} <type>[(<scope>)]: description`

Types: `feat`, `fix`, `chore`, `test`, `docs`, `refactor`

Examples:
- `MER-123 feat(domain): add user preferences schema and entity`
- `MER-298 fix(ci): pass NVD API key via env block`
- `MER-219 test: reproduce lock-screen timer regression`

#### Issue Workflow — Follow for Every GitHub Issue

1. **Add issue to the project board** if created via `gh issue create` (it does NOT auto-add): `gh project item-add 8 --owner AnunnakiCosmoCrew --url <issue-url>`.
2. **Set an estimate** (Fibonacci: 0, 1, 2, 3, 5, 8, 13) on the board's "Estimate" field. Bugs are `0`. Note: `gh` CLI refuses `--number 0` — use the raw `updateProjectV2ItemFieldValue` GraphQL mutation for bug estimates.
3. **Move ticket to "In Progress"** on the project board before writing any code.
4. **Create a feature branch** from latest `main` using the naming convention above.
5. **Implement and verify**:
   - Backend: `./mvnw verify` (all tests pass, coverage ≥ 80% unit / 70% component, Checkstyle clean)
   - Frontend: `flutter analyze --no-fatal-infos`, `dart format --set-exit-if-changed`, `flutter test`
6. **Commit** to the feature branch with a descriptive message referencing the issue number.
7. **Push and open a PR** with `Closes AnunnakiCosmoCrew/<repo>#NNN` in the body so the issue auto-closes on merge. Push and open the PR immediately after completing the task — do not wait for the user to ask.

> **Do NOT skip steps 1–3.** The project board must reflect the current state of work at all times.

#### Bug Fixes — Test-Driven Bug Fixing (TDBF)

When fixing a bug, the failing test that reproduces it must be written and committed *before* the fix. Two commits on the same branch:

1. **Red commit** — test that reproduces the bug. `flutter test` / `./mvnw test` will fail on this test by design; analyze/format/Checkstyle must still pass. Commit: `MER-{N} test: reproduce <bug description>`. Push before writing the fix so CI confirms the red path.
2. **Green commit** — the fix. All tests pass. Commit: `MER-{N} fix[(<scope>)]: <what the fix does>`.

#### Branch Protection

- Squash merge only (merge commits and rebase merges disabled)
- Required CI checks must pass before merge:
  - Backend: `verify` (PR Quality & Security) + `scan` (Semgrep SAST)
  - Frontend: `analyze-and-test`
- All review conversations must be resolved before merge — reply **and** explicitly resolve each thread
- No approving review required (solo developer — Copilot provides automated review)
- Force push and branch deletion blocked on `main`; head branches auto-deleted after merge

## ADR Conventions

Architecture Decision Records live in [`adr/`](adr/). The next ADR number is **045**.

- **Filename**: `NNN-kebab-slug.md` (zero-padded to 3 digits, e.g., `042-new-decision.md`)
- **Template**: [`adr/template.md`](adr/template.md)
- **Index**: update [`adr/README.md`](adr/README.md) with a new row for every ADR
- **Linking**: reference ADRs from app repo comments/CLAUDE.md as `[ADR-NNN](https://github.com/AnunnakiCosmoCrew/slicefocus-docs/blob/main/adr/NNN-slug.md)`
- **Immutability**: once an ADR is accepted, mark superseded ones as `Superseded by ADR-NNN` rather than editing them

## Project Management

- **Issue tracking**: GitHub Issues on the app repos + [GitHub Projects board #8](https://github.com/orgs/AnunnakiCosmoCrew/projects/8) (migrated from Linear, March 2026)
- **Issue IDs**: `MER-N` (GitHub Issue numbers, not Linear IDs)
- **Issue location**: Backend and cross-stack issues live in `SliceFocus`; pure-FE issues live in `SliceFocusFE`; the project board aggregates both
- **Estimation**: Fibonacci story points (0, 1, 2, 3, 5, 8, 13) on the board's "Estimate" field; bugs are always `0`
- **Historical data**: Full Linear export (300 issues) available in [`data/`](data/) for velocity and analysis
- **Architecture decisions**: Documented as ADRs in [`adr/`](adr/) — consult ADRs before proposing changes to established patterns
