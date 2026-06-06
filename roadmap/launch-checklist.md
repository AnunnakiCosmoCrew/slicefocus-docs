# SliceFocus App Store Launch Checklist

*Created: 2026-03-23 | Target: App Store + Play Store submission*

> **⚠️ Superseded (historical).** This pre-launch checklist is kept for historical reference. The macOS and iOS App Store launches are complete (v1.0/v1.1) and the app has shipped through **v1.2.0** (June 2026), including an automated TestFlight pipeline ([ADR-041](../adr/041-automated-testflight-fastlane.md)). The unchecked boxes below reflect the **March 2026 pre-launch state** and are no longer maintained. See the [v1.2 release audit](../audits/v1.2-release-audit.md) for current status.

---

## Phase 1: Critical Bug Fix
- [ ] [#153](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/153) — Slice deleted on session completion (data loss) `Urgent` `5pts`

## Phase 2: Quick Wins + Core UX Fixes
- [ ] [#155](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/155) — Default soundscape = None `Medium` `1pt`
- [ ] [#163](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/163) — Fix display name → "SliceFocus" `Low` `1pt`
- [ ] [#164](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/164) — Fix test env URL `Low` `1pt`
- [ ] [#154](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/154) — Can't start session before slice time `High` `3pts`

## Phase 3: Apple Sign-In + Backend Promotion
- [ ] [#123](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/123) — Configure Apple Sign-In *(In Progress)* `Medium`
- [ ] [#166](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/166) — Promote backend to test `Medium` `2pts`
- [ ] [#167](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/167) — Promote backend to prod `Medium` `2pts`

## Phase 4: Store Submission Blockers
- [ ] [#159](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/159) — Privacy Policy `High` `2pts`
- [ ] [#161](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/161) — App icon `High` `2pts`
- [ ] [#162](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/162) — Release signing (iOS + Android) `High` `3pts`
- [ ] [#160](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/160) — Store metadata (screenshots, description) `High` `3pts`
- [ ] [#148](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/148) — Accessibility Statement for App Store *(In Progress)* `1pt`
- [ ] [#165](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/165) — TestFlight beta `Medium` `3pts`

## Phase 5: Post-Launch Features
- [ ] [#157](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/157) — Edit time block name `Medium` `2pts`
- [ ] [#156](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/156) — Quick-start focus session with auto-extend `High` `8pts`
- [ ] [#158](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/158) — Notify/auto-start when time block begins `Medium` `3pts`

---

## Totals

| Phase | Points | Description |
|---|---|---|
| Phase 1 | 5 | Critical bug fix |
| Phase 2 | 6 | Quick wins + UX fix |
| Phase 3 | 4+ | Auth + backend environments |
| Phase 4 | 14+ | Store submission prep |
| Phase 5 | 13 | Post-launch features |
| **Total** | **42+** | |

**Launch-critical path (Phases 1–4): ~31 points to first TestFlight submission.**
