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

### Design Decisions

| # | Decision | Status | Notes |
|---|----------|--------|-------|
| 1 | Visual concept / style | Pending | |
| 2 | Color palette | Pending | Should align with app theme |
| 3 | Foreground/background split (Android adaptive) | Pending | |
| 4 | Tool for generation | Pending | `flutter_launcher_icons` or similar |

---

## Deliverables Checklist

- [ ] Source design file (Figma / SVG)
- [ ] iOS icon set (`ios/Runner/Assets.xcassets/AppIcon.appiconset/`)
- [ ] Android adaptive icon (`android/app/src/main/res/`)
- [ ] macOS icon (`.icns`)
- [ ] `flutter_launcher_icons` config (if used)
- [ ] Verify no alpha channel on iOS icon
- [ ] Screenshot / visual preview added to this document

---

## Work Log

| Date | Activity | Outcome |
|------|----------|---------|
| 2026-03-24 | Created tracking document | — |

---

## References

- Issue: [#161](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/161)
- Related: [#160 — App Store / Play Store metadata](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/160)
- Related: [#162 — Release signing for iOS and Android](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/162)
