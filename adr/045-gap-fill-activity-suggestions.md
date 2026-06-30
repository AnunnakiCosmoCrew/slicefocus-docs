# ADR-045: Gap-fill activity suggestions between events

**Status:** Accepted
**Date:** 2026-06-30
**Deciders:** Mert Ertugrul

## Context

r/ProductivityApps launch feedback (Jun 2026) asked the app to help users fill
the empty space in a day, not just stare at a blank canvas. SliceFocusFE issue
[#324](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/324) (Smart-planning
epic [#314](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/314), milestone
v1.4.0) calls for detecting gaps between scheduled activities and suggesting a
curated activity — a productivity block or a break/relaxation — addable in one tap.

The app already lays slices on a 24-hour radial canvas and computes the day's
empty gaps (`DaySlice.withUnplannedGaps`), and ships an app-curated template set
(`TemplateLibrary`) plus user categories (epics #322/#323, both closed). What was
missing was a way to turn a gap into a concrete, sized, one-tap suggestion.

## Decision

Add a new pure-Dart feature module `lib/src/features/suggestions/` and surface it
on the home screen.

- **Gap source = `DaySlice.withUnplannedGaps`.** Suggestions are computed from the
  app's own scheduled slices. The engine takes only the interleaved slice list, so
  it extends naturally to imported system-calendar events once those render in the
  day view (#357) — but #324 takes **no** dependency on the calendar work.
- **Curated catalog derived from `TemplateLibrary`** (`SuggestionCatalog`), not a
  second hardcoded list and not a backend. Each distinct template-slice label
  becomes one suggestion, classified productivity vs break/relaxation by a small
  label set, with two generic fallbacks ("Focus" / "Recharge") so every gap can
  offer one of each kind.
- **Deterministic engine** (`SuggestionEngine`): sizes each suggestion to the gap
  (clamped, min-fit 15 min), ranks by the gap's time-of-day (morning → focus,
  evening → relaxation), always mixes both kinds when there is room, and **excludes
  activities already planned that day** so it never suggests a duplicate.
- **Two surfaces:** (a) a proactive home card highlighting the day's largest
  fillable gap (between canvas and timeline), and (b) tapping any empty gap (the
  timeline separator, plus a "Suggest activity" canvas context-menu item) opens a
  suggestion sheet.
- **Add immediately:** choosing a suggestion writes a real slice into the gap via
  the existing `saveSlice` path (auto-sized, auto-coloured); it is then editable
  like any other slice. No mandatory editor step, no new persistence.

## Alternatives Considered

- **Pull-only (tap a gap) or push-only (card).** Rejected in favour of both:
  the card is discoverable, the gap-tap is precise. Confirmed with the product owner.
- **Open a pre-filled editor instead of adding immediately.** Rejected — the issue
  asks for one-tap add; the slice stays editable afterward, so the editor step is
  redundant friction.
- **A separate hardcoded suggestion catalog or a backend-driven one.** Rejected —
  deriving from `TemplateLibrary` keeps suggestions in sync with the curated set
  for free, and the EventKit epic (#315) already established that this client-side,
  no-backend approach is preferred where possible.
- **Empty-day gap is fillable too.** Rejected — an empty day has no events to gap
  *between*; the existing "Start with a template" call-to-action owns that case, so
  the proactive card and the tappable separator are suppressed on an empty day.

## Consequences

- New always-on home UI: the proactive card renders whenever a planned day has a
  ≥30-min gap. Existing home widget tests that asserted exact icon/label counts
  needed scoping (the separator no longer adds an `Icons.add`; suggestions dedupe
  against planned labels), and the `mer215` golden was regenerated.
- The suggestion engine is pure-Dart and fully unit-tested (100% module coverage);
  time-of-day and classification heuristics are intentionally simple and live in
  one place to evolve later (e.g. weighting by user categories or history).
- When calendar import (#357) lands, feeding imported events into the same slice
  timeline will make gap detection calendar-aware with no engine change.
