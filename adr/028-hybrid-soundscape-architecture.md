# ADR-028: Hybrid Soundscape Architecture (CDN File-Based + Programmatic Noise)

**Status:** Accepted
**Date:** 2026-03-18
**Deciders:** Mert Ertugrul

## Context

ADR-015 covers programmatic noise generation for white/brown/pink noise. However, the broader soundscape system serves two distinct categories of audio: nature/ambient sounds (rain, forest, cafe) that require pre-recorded audio files, and colored noise types that can be generated algorithmically in real-time. These two categories have fundamentally different delivery and playback requirements.

## Decision

Adopt a hybrid architecture with two audio delivery mechanisms, coordinated by a `SoundscapeOrchestrator`:

- **Nature/ambient sounds** (rain, forest, cafe, etc.) are OGG files hosted in a GCS bucket with CDN, streamed via `just_audio`. The backend soundscape catalog API returns an `audioUrl` pointing to the CDN.
- **Noise types** (white, brown, pink) are generated programmatically using `flutter_pcm_sound` with zero asset size. The catalog API returns `audioUrl = null` for these entries, signaling the client to use local generation.

The `SoundscapeOrchestrator` inspects the soundscape type and delegates to the appropriate playback service — `CdnSoundscapeService` for file-based audio or `NoiseGeneratorService` for programmatic noise. This selection is transparent to the UI layer.

## Alternatives Considered

- **Bundle all audio in the app binary:** Would add 600MB+ to the app size. Unacceptable for mobile distribution.
- **Stream everything from CDN (including noise):** Wasteful — colored noise is trivially generated from a few lines of DSP code. Streaming it burns bandwidth for no benefit.
- **Programmatic-only (synthesize all sounds):** Complex ambient soundscapes (cafe chatter, rainstorms) cannot be convincingly synthesized in real-time with reasonable CPU cost.

## Consequences

- Zero app size impact for noise types — they exist purely as code
- CDN hosting adds operational cost for ambient sound files, but enables adding new soundscapes without app updates
- Offline behavior differs by type: noise generation always works offline, while ambient sounds require prior streaming/caching
- The `audioUrl` null-check convention is a simple but effective contract between backend catalog and client orchestrator

*Related: MER-237, MER-238, MER-239, MER-250, MER-236*
