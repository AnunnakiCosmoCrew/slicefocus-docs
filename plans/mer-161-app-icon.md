# MER-161 Work Log — Design and Set Production App Icon

*Created: 2026-03-24*

## Problem

The app currently uses the default Flutter app icon. A branded icon is needed for App Store and Play Store submission.

---

## Platform Requirements

| Platform | Size | Format | Notes |
|----------|------|--------|-------|
| iOS | 1024x1024 | PNG (no alpha) | App Store requirement; auto-scaled for devices |
| Android | 512x512 | PNG | Play Store; adaptive icon (foreground + background layers) |
| macOS | 1024x1024 | .icns | Desktop distribution |

---

## Design Direction

The icon should represent the **radial time chart** concept — the core visual identity of SliceFocus.

### Final Design: Clock Forest

Segmented donut chart with 5 asymmetric slices, clock tick markers around the outer edge, a play triangle in the center, on a white background with rounded-square mask.

**Color palette (forest/nature):**
- Deep green `#2D4A3E` (primary slice + play button)
- Sage `#B8C9A3`
- Moss `#7BAE7F`
- Teal-green `#4A8C6F`
- Earth brown `#A67C52`

Source file: [`plans/icon-final-clock-forest.svg`](icon-final-clock-forest.svg)

### Design Decisions

| # | Decision | Status | Notes |
|---|----------|--------|-------|
| 1 | Visual concept / style | **Done** | Clock Forest — segmented donut with play button |
| 2 | Color palette | **Done** | Forest/nature palette (5 colors above) |
| 3 | Foreground/background split (Android adaptive) | **Done** | Donut+play as foreground, white background |
| 4 | Tool for generation | **Done** | `flutter_launcher_icons` package |

---

## Deliverables Checklist

- [x] Source design file (SVG) — `plans/icon-final-clock-forest.svg`
- [x] PNG preview — `plans/icon-final-clock-forest.png`
- [ ] iOS icon set (`ios/Runner/Assets.xcassets/AppIcon.appiconset/`)
- [ ] Android adaptive icon (`android/app/src/main/res/`)
- [ ] macOS icon (`.icns`)
- [ ] `flutter_launcher_icons` config in FE repo
- [ ] Verify no alpha channel on iOS icon

---

## Work Log

| Date | Activity | Outcome |
|------|----------|---------|
| 2026-03-24 | Created tracking document | — |
| 2026-03-24 | Explored 6 design directions (concept A/B/C, lettermark, notebook, clock) | Clock concept chosen |
| 2026-03-24 | Iterated on clock concept — refined to Clock Forest with nature palette | Final SVG + PNG ready |
| 2026-03-24 | Created MER-160 store metadata document | Full store copy drafted |
| 2026-03-24 | Cleaned up iteration files, kept only final deliverables | Ready for platform export |

---

## References

- Issue: [#161](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/161)
- Related: [#160 — App Store / Play Store metadata](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/160)
- Related: [#162 — Release signing for iOS and Android](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/162)
