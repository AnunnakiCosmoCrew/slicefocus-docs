# ADR-042: Minimal & sincere marketing design language

**Status:** Accepted
**Date:** 2026-06-06
**Deciders:** Mert Ertugrul

## Context

The Launch push epic (#309) produces several outward-facing assets that must feel
like one product: the auto-advance hero video (#311), the one-page landing site
(#312), the ASO screenshots/copy, and the Reddit (#313) and Show HN (#314) posts.
Without a shared design language these drift into different tones — a glossy promo
video next to a plain landing page next to hype-y store copy — which reads as
incoherent and, worse, contradicts the product itself.

SliceFocus's USP is **calm, zero-touch auto-advance** on a 24-hour radial canvas.
Loud, motion-graphics-heavy, feature-bombarding marketing actively undercuts that
pitch. Separately, the two highest-leverage launch moments — Reddit's productivity
subs and especially Show HN — are audiences that *distrust* over-produced
marketing and reward sincerity. The claude.ai homepage is a strong reference for
the target tone: lots of whitespace, the real product shown doing one real thing
in a short silent loop, honest and informative supporting sections, no hype.

## Decision

Adopt a **minimal & sincere** design language across all launch-facing marketing
surfaces. Concretely:

- **Show the real product doing the real thing.** No abstracted illustrations,
  no glamour device-bezel hero shots, no zoom/parallax/kinetic effects. The app
  UI on a calm neutral background, with generous whitespace.
- **Let the product be the only saturation.** Muted, restrained palette; the
  radial block colors carry the visual interest.
- **Motion over captions.** Hero/loop media is silent and caption-free where the
  motion is self-explanatory; the auto-advance handoff speaks for itself.
- **Inform, don't bombard.** Supporting sections are genuinely informative (how it
  works, the science) rather than a wall of feature bullets.
- **Sincere copy.** Plain, honest, no superlatives or growth-hack tone.

This governs #311 (video), #312 (landing page), the ASO assets, and the #313/#314
post copy and media.

## Alternatives Considered

- **Polished, high-production promo (motion graphics, music, kinetic text).**
  Rejected: expensive for a solo dev to do well, contradicts the calm product
  pitch, and is exactly the register HN punishes.
- **No shared language — decide tone per asset.** Rejected: produces incoherent
  surfaces and re-litigates the same overlay/effects/copy debates on every ticket.
- **Feature-forward landing page (comprehensive feature grid).** Rejected:
  dilutes the single USP (auto-advance) the whole launch is built to showcase.

## Consequences

- **Easier:** asset decisions collapse to a single question — "is this minimal and
  sincere?" The video's overlay/effects debate is resolved (no captions); the
  landing page scope shrinks (whitespace + loop + a few honest sections).
- **Easier:** consistent tone across video, site, store, and posts reinforces one
  brand impression and de-risks the HN/Reddit moments.
- **Harder / requires discipline:** restraint is a constraint — the temptation to
  add "just one more feature callout" or a flashy transition must be resisted.
- **Constraint:** sincerity means the assets must show genuine product behavior;
  staging may compress the clock (short blocks) but must never fake the feature.
