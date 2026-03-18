# ADR-015: Programmatic noise generation over bundled audio files

**Status:** Accepted
**Date:** 2026-03-17
**Deciders:** Mert Ertugrul

## Context

The soundscape feature includes 10 options: 7 nature/ambient sounds and 3 noise types (white, brown, pink). Bundling pre-recorded 60-minute WAV files for noise types would add ~600MB each to the app size, and loop boundaries create audible artifacts.

## Decision

Generate white, brown, and pink noise programmatically using flutter_pcm_sound (~100-200 lines of Dart per noise type) instead of bundling audio files. Nature/ambient soundscapes use pre-recorded OGG files from CDN, downloaded and cached on first use. A SoundscapeOrchestrator delegates to the appropriate service based on soundscape type.

## Alternatives Considered

- **Bundling all audio files**: Rejected due to ~600MB app size increase for noise types alone.
- **Streaming noise from a server**: Unnecessary complexity and bandwidth waste for algorithmically trivial signals.
- **Only programmatic generation**: Would limit the feature (no nature sounds).

## Consequences

- Zero asset size for noise types with infinite seamless playback and no loop artifacts
- Nature sounds require CDN hosting and first-use download (~4-19MB per file)
- Dual-service architecture adds some complexity but cleanly separates concerns

*Related: MER-237, MER-235, MER-236, MER-238, MER-239*
