# ADR-058: Apple builds on Swift Package Manager only, with `flutter_pcm_sound` vendored

**Status:** Accepted
**Date:** 2026-10-13
**Deciders:** Mert Ertugrul

## Context

The CocoaPods trunk registry becomes **permanently read-only on 2 December 2026**, Firebase stopped
publishing new versions to CocoaPods in October 2026, and CocoaPods itself is in maintenance mode and
will eventually be removed from Flutter. Swift Package Manager (SPM) has been Flutter's default since
3.44; the project is on 3.47.

Despite that default, SliceFocusFE was still built **entirely** through CocoaPods (FE #432). Nothing had
actually been migrated: neither Xcode project referenced `FlutterGeneratedPluginSwiftPackage`, both
`Podfile.lock`s listed every plugin, and the local Flutter config carried an explicit
`enable-swift-package-manager: false`. CI, running a fresh stable-channel Flutter where the flag
defaults to on, built differently from every developer machine — the most likely explanation for the
`Swift Tests` workflow having been red since 2026-06-03 with `Module 'audio_session' not found`.

By October 2026, 17 of the 18 Apple plugins shipped a `Package.swift`: `permission_handler_apple` and
`patrol`, both named as blockers when #432 was written, had resolved themselves through ordinary
dependency bumps. One plugin was left, and it was the only thing keeping CocoaPods in the build:

**`flutter_pcm_sound`**, which generates the soundscape white/brown/pink noise. It never gained SPM
support, and on **2026-10-02 its repository was archived** with a "NO LONGER MAINTAINED" README and its
issue tracker disabled. The migration could therefore not be contributed upstream, which had been the
preferred plan. Its last pub.dev release is `3.3.3` (October 2025).

The dependency was already costing us: with CocoaPods broken on the founder's machine after the arm64
migration, *no* iOS or macOS build could run locally at all.

## Decision

Drop CocoaPods from both Apple targets entirely, and **vendor `flutter_pcm_sound` in-tree** to get there.

- **No CocoaPods in the build.** `ios/Podfile`, `macos/Podfile`, both `Podfile.lock`s and the `Pods`
  project references are gone (`pod deintegrate` on both projects, plus removal of the `Pods-Runner`
  xcconfig includes). `flutter build <platform> --config-only` is now what resolves native dependencies
  and wires them into the Xcode project, in CI as well as locally.
- **`Package.resolved` is the new lockfile.** The workspace copies for both platforms are committed,
  and pin `firebase-ios-sdk` and the rest of the native graph exactly as `Podfile.lock` used to. The
  copies Xcode writes under `Runner.xcodeproj/project.xcworkspace/` are local state and are gitignored.
- **`flutter_pcm_sound` lives in `packages/flutter_pcm_sound/`** as a path dependency, copied from
  upstream `master` at `985fa9a` — the final state of the archived project. Only the *build layout*
  changed: the byte-identical `ios/Classes` and `macos/Classes` trees became one shared
  `darwin/.../Sources` tree (`sharedDarwinSource: true`) with a `Package.swift`, and the two podspecs
  became one, kept only as a fallback. The native and Dart code are upstream's. The licence is the
  Unlicense, so vendoring is unencumbered.
- **A single podspec is retained** in the vendored package so the plugin still resolves if CocoaPods is
  ever deliberately re-enabled; nothing in the build path reads it.

Vendoring from `master` rather than from `3.3.3` also picks up three fixes upstream never released:
Java 17 compatibility, `compileSdkVersion flutter.compileSdkVersion` instead of a pinned 33, and the
`androidAudioUsage` family of `setup()` arguments. Those arguments are optional and defaulted, so the
Dart API stays source-compatible.

## Alternatives Considered

- **Contribute the SPM migration upstream.** The original plan in #432, and the right answer until
  2026-10-02. The repository is now archived with issues disabled, so there is no upstream to send it to.
- **Fork to an org repository and consume it via a git ref.** Identical maintenance burden to vendoring,
  with an extra moving part: CI would fetch a second repository at build time, and the pinned ref becomes
  another thing to keep honest. An in-tree path dependency is hermetic and reviewable in the same PR.
- **Replace the plugin with `flutter_soloud`** (5.2.0, SPM-ready, actively maintained). Keeps true
  real-time PCM feeding and removes code we own. Rejected *for now* on cost: an 18 MB C++ audio engine
  as a new dependency, a measurable binary-size increase, a rewrite of `NoiseGeneratorService` against a
  different API, and full soundscape audio-regression testing — none of which fits in front of a hard
  December deadline. It remains the natural successor if the vendored plugin ever becomes a burden.
- **Re-implement noise on `just_audio`** (already a dependency, already SPM-ready): pre-render N seconds
  of noise with the existing pure-Dart `NoiseAlgorithms`, wrap it in a WAV header and loop it. Removes a
  native plugin outright. Rejected on audio quality: it gives up real-time generation, and a looped
  finite buffer risks audible periodicity and a seam at the loop point, worst on brown noise.
- **Keep CocoaPods and reinstall it on the affected machine.** Buys time, buys nothing else, and the
  2 December freeze still arrives.

## Consequences

**Easier**

- Clean builds have no `pod install` step. Local Apple builds work again on a machine with no working
  CocoaPods — which, before this change, meant no local iOS or macOS build at all.
- Local and CI builds finally agree, since neither has a CocoaPods path to diverge down.
- `flutter_pcm_sound` stops being a dead-upstream risk: the code is ours, with three unreleased upstream
  fixes already folded in.

**Harder**

- We own ~380 lines of Objective-C per Apple platform. The surface is small — zero dependencies, one
  class — but nobody upstream will fix it.
- A new plugin without a `Package.swift` can no longer simply be added; it has to be vendored the same
  way or rejected.
- `flutter pub get` now checks third-party Swift packages out under `build/{ios,macos}/SourcePackages`,
  Dart sources included. `flutter analyze` excludes `build/**`, but a bare `dart format .` will reformat
  them. CI is unaffected (`analyze-and-test` runs on Linux, which resolves no Apple packages).
- `Package.resolved` has to be committed when native dependency versions move, exactly as `Podfile.lock`
  did. An uncommitted change there silently unpins the Firebase SDK.

**Carried along, and worth knowing**

- Flutter 3.47 raises the macOS deployment target from **10.15 to 12.0** on any build with this SDK. It
  is not a consequence of the SPM move, but it lands in the same change: the next macOS release drops
  support for Catalina and Big Sur.
