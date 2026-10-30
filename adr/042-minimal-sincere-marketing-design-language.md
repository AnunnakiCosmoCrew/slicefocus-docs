# ADR-042: Minimal & sincere marketing design language

**Status:** Accepted (amended 2026-10-23, 2026-10-30)
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
marketing and reward sincerity. The target tone: lots of whitespace, the real
product shown doing one real thing in a short silent loop, honest and informative
supporting sections, no hype.

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

## Amendment 2026-10-23: time compression and framing moves for the hero

The first hero take under this ADR (a fixed 1440x900 capture of the Mac window in which only the timer panel changed) did no more than a still image would. For the hero loop on slicefocus.app, and only there, two things the decision above forbids are now allowed, by the founder's call ([plans/hero-video-brief.md](../plans/hero-video-brief.md), SliceFocusFE#622):

- **Time compression of real footage.** A real countdown may be sped up so a 25-minute phase passes in a few seconds, with the digits visibly racing and the clip settling back to real time before the handoff. The footage stays real: the phase change is the app's own, never staged or cut in.
- **Restrained, eased framing moves.** Gentle push-ins of at most 1.15x and pans between the canvas and the timer panel, with ease-in-out and no hard zooms, cuts to detail, or parallax.

Everything else holds: the real UI only, a silent file, no captions or burned-in text, no device bezels, no particles or motion graphics, no cursor theatre, no hype. The "no zoom/parallax/kinetic effects" line in the decision still governs the feature clips and the store assets.

## Amendment 2026-10-30: the site palette comes from the app icon

The first landing page borrowed another company's look: Anthropic's cream canvas (#f0eee6) and clay accent (#d97757), with code comments that said so. A product site should look like its own product, so the founder compared three palettes on the live page and chose the one taken from the SliceFocus icon (slicefocus-web#34):

- **Light:** sage-white canvas `#f3f5f0`, green-black text `#16211c`, forest green accent `#3c7a5e`, deep forest pill button `#2d4a3e`.
- **Dark:** `#111714` canvas, `#e5ece7` text, sage accent `#8dbf91`.
- **Every text colour is at least 4.5:1** on its background (the old tertiary grey was 3.0:1).
- The Open Graph card uses the same palette. Fraunces and Inter stay.

The alternative closest to the app screens (its own `#F4F7FB` background and `#005A9C` blue) was also considered. The known trade-off of the forest palette is that green is also the colour of Forest, a competitor the site has a comparison page against.

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
