# ADR-035: Versioning Strategy (FE, BE, API)

**Status:** Accepted
**Date:** 2026-03-26
**Deciders:** Mert Ertugrul

## Context

SliceFocus has three components that evolve independently:
- **Frontend** (Flutter) — deployed to App Store / Play Store
- **Backend** (Spring Boot) — deployed to Cloud Run
- **REST API** — the contract between them

Each has different release cadences, audiences, and constraints. We need a versioning strategy that:
- Communicates the nature of changes (breaking, feature, fix)
- Supports independent deployment
- Satisfies App Store requirements (monotonic build numbers)
- Enables safe API evolution without forcing synchronized releases

## Decision

### 1. Semantic Versioning (SemVer) for all components

All components follow `MAJOR.MINOR.PATCH`:
- **PATCH** — bug fixes, no new features, no API changes
- **MINOR** — new features, backwards compatible
- **MAJOR** — breaking changes

### 2. Independent version numbers

FE and BE are versioned independently. `FE 2.3.0` can work with `BE 1.8.0` as long as they share a compatible API version.

| Component | Version source | Current |
|-----------|---------------|---------|
| FE (Flutter) | `pubspec.yaml` → `version: X.Y.Z+build` | `1.0.0+1` |
| BE (Spring Boot) | `pom.xml` → `<version>X.Y.Z</version>` | `4.0.4` |
| API | URL prefix `/api/v1/` | v1 |

### 3. Git tags per component

Tags use a component prefix to distinguish releases in the shared issue tracker:
- `fe/v1.0.0` — FE release
- `be/v4.1.0` — BE release

Tags are created on the release commit (after PR merge to main).

### 4. Flutter build number

The `+N` suffix in `pubspec.yaml` is the build number required by App Store / Play Store:
- **Must be monotonically increasing** — each submission must have a higher build number
- Auto-incremented in CI or manually before release
- Independent of the version string (can reset on major version bumps if needed)

### 5. API versioning

The REST API uses URL-prefix versioning (`/api/v1/`, `/api/v2/`):
- Breaking API changes require a new version prefix
- Old versions are maintained until all clients upgrade
- Non-breaking additions (new optional fields, new endpoints) stay in the current version
- The FE's `AppConfig` maps environments to base URLs; API version is part of the path

### 6. When to bump

| Event | Version change | Who |
|-------|---------------|-----|
| Bug fix PR merged | Patch (`1.0.1`) | At release time, not per PR |
| New feature merged | Minor (`1.1.0`) | At release time |
| Breaking change | Major (`2.0.0`) | Deliberate decision |
| App Store submission | Bump build number (`+2`) | Before submission |
| Hotfix | Patch from release tag | Branch from tag, merge back |

Version bumps happen at **release time**, not per PR. Multiple PRs accumulate between releases.

### 7. Changelog

Each release tag includes a changelog in the tag annotation (or `CHANGELOG.md`):
```
## v1.1.0 (2026-03-26)
### Features
- Quick Focus: one-tap session start (#175)
- Edit time block from timeline (#187)
### Fixes
- Timer restart on desktop switch (#181)
- Soundscape selection persists correctly (#178)
```

## Alternatives Considered

- **Monorepo with shared version**: Forces synchronized releases. Rejected — FE and BE have different release cadences (App Store review vs Cloud Run deploy).
- **CalVer (calendar versioning)**: `2026.03.1`. Simple but doesn't communicate change impact. Rejected for SemVer's clearer signal.
- **No formal versioning**: Just use git hashes. Rejected — App Store requires version strings, and users/support need human-readable version identifiers.

## Consequences

- Clear communication of change impact to team and users
- Independent deployment without version coordination
- App Store compliance via build number management
- API evolution without breaking existing clients
- Slight overhead of remembering to tag releases (automated in CI later)

*Related: MER-160 (App Store metadata), MER-162 (Release signing)*
