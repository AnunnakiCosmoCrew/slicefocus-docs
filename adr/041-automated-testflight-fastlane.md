# ADR-041: Automated TestFlight delivery via Fastlane

**Status:** Accepted
**Date:** 2026-06-05
**Deciders:** Mert Ertugrul

## Context

Through v1.1, iOS releases were **manual**: archive in Xcode, sign, and upload to App Store Connect by hand. Issue #165 ("TestFlight setup") only covered the *first* manual submission — it did not establish a repeatable pipeline. The v1.1 release audit flagged this directly as a v1.2 action item: *"Add FE deployment pipeline — FE has CI but no automated deploy workflow."*

Manual iOS delivery is slow, error-prone (signing state, build numbers), and not reproducible. The frontend already had CI for quality (`ci.yml`: format + analyze + test) but nothing that produced and shipped a build.

## Decision

Add a GitHub Actions workflow, [`.github/workflows/testflight.yml`](https://github.com/AnunnakiCosmoCrew/SliceFocusFE) (introduced in PR #267 / MER-266), that builds and uploads to TestFlight via **Fastlane**:

- **Code signing:** Fastlane `match`, with certificates and provisioning profiles stored in a private `slicefocus-certs` git repo. `match` provisions **two** bundle IDs — the app (`com.anunnakicosmo.sliceFocusFe`) and the Live Activity widget extension (`...SliceFocusTimerWidgetExtension`).
- **Upload:** `upload_to_testflight` (pilot), authenticated with an App Store Connect API key.
- **Runner:** `macos-15`, latest Xcode selected, Ruby + Bundler in `ios/`.
- **Two-job gate:** a `decide` job determines whether to ship; a `deploy` job runs only when it should.

**Triggers:**
- Push a `fe/v*` tag → always ships (tag glob corrected to `fe/v*` in #271, aligning with the ADR-035 versioning convention).
- Version bump merged to `main` → ships only if `pubspec.yaml`'s `version:` changed (#274). Dependency/doc commits run `decide` and skip `deploy`.
- `workflow_dispatch` → always ships.

Build-number bumping stays **manual** (bump `+N` in `pubspec.yaml` via PR). CI-time code signing is pinned to manual via `update_code_signing_settings` **guarded by `if ENV["CI"]`**, because the Xcode project uses `CODE_SIGN_STYLE = Automatic` for local development (which CI cannot satisfy at archive time). The CI guard keeps a local `fastlane beta` from rewriting the project.

The first proven end-to-end run was **2026-06-05** (build `1.2.0 (21)`).

## Alternatives Considered

- **Xcode Cloud** — Apple-native, tight App Store Connect integration. Rejected: paid compute beyond a small free allotment, and less flexible than a scriptable Fastfile for the dual-bundle-ID + Live Activity widget signing this app needs.
- **Codemagic / Bitrise** — Flutter-friendly mobile CI SaaS. Rejected: another paid platform and account; the project already lives on GitHub Actions, and Fastlane runs there fine on `macos-15` runners.
- **Status quo (manual archive + upload)** — Rejected: the exact problem this ADR solves; not reproducible, blocks fast iteration.
- **GitHub Actions without Fastlane** — Possible, but would mean re-implementing signing, profile management, and TestFlight upload by hand. Fastlane's `match` + `pilot` are the de-facto standard for precisely this.

## Consequences

- **Repeatable releases:** A version bump (or tag) ships a TestFlight build with no manual Xcode steps.
- **Intentional gating:** A non-version change merged to `main` does **not** ship — the bump trigger sees no version change and skips `deploy`. To deploy current `main` without bumping (e.g. after a CI/Fastfile fix), run `gh workflow run testflight.yml --ref main`.
- **Secrets to manage:** `MATCH_PASSWORD`, `MATCH_GIT_URL`, `MATCH_GIT_BASIC_AUTHORIZATION`, `ASC_KEY_ID`, `ASC_ISSUER_ID`, `ASC_KEY_CONTENT`.
- **Signing complexity:** The `Automatic` (local) vs pinned-manual (CI) split is subtle but necessary; documented in the Fastfile and team memory.
- **Cost:** macOS runner minutes per shipping run (and the `swift-tests.yml` runs). Acceptable for the release cadence.
- **Android gap:** This pipeline is iOS-only. Android/Play delivery remains manual — a candidate for a future ADR.
- **Closes v1.1 audit action #9** ("Add FE deployment pipeline").

*Related: ADR-035 (versioning strategy — `fe/v*` tags, `pubspec.yaml X.Y.Z+build`), ADR-023 (PR validation / CI workflow precedent), MER-266 / #267, MER-274, MER-276/#277, MER-271.*
