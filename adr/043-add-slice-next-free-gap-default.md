# ADR-043: Add-slice defaults into the next free gap

**Status:** Accepted
**Date:** 2026-06-09
**Deciders:** Mertugrul

## Context

The 24-hour radial canvas is primarily a day-planning tool: users build their day by
adding one time block after another. The "+" button in the AppBar and the "Add time
block" row in the timeline open the create-slice modal with a default start/end time.

The original default (ADR-less, introduced in MER-284 as `DaySlice.nextStartMinute`)
returned the **end time of the planned slice nearest to "now"** by circular distance,
then set a fixed 60-minute block (`start + 60`). Two problems surfaced in real
day-planning use:

1. **The default does not advance.** When a user plans their morning at, say, 10:00
   and keeps adding blocks, the planned-slice edge nearest to 10:00 does not move, so
   every new add keeps defaulting to the same time (e.g. 12:30). The user has to drag
   the start forward on every single add — tedious and error-prone.
2. **The default overlaps existing slices.** The nearest slice's end is usually the
   *start of the next slice*, so the default 60-minute block lands on top of an
   existing block. Example: with Focus Work 08:00–17:00 and Gym 17:00–18:30, at 16:00
   the default became 17:00–18:00, sitting entirely on top of Gym. The backend then
   rejects the overlap, surfacing an error for what should be a one-tap action.

## Decision

Replace `nextStartMinute` with `DaySlice.nextFreeSlot`, which returns both a start
**and** a clamped end for a new slice:

- The candidate free regions are exactly the gaps already computed by
  `DaySlice.withUnplannedGaps` (its "Unplanned" entries), so gap-finding reuses one
  source of truth and inherits its sorting and midnight-wrap handling.
- **Reference point is the viewed day, not today's clock.** The day-planning canvas
  can show any date, so the default is anchored to the pie currently on screen:
  - **Viewing today** — anchor to the live clock minute: the next free gap moving
    *forward* from now (a gap containing now starts there; otherwise the gap whose
    start is the smallest clockwise distance ahead). As gaps fill, the default
    advances automatically, so consecutive adds chain onto the day's progress.
  - **Viewing another day** (past/future) — there is no "now", so the default
    **appends after the last planned block** of that day (the latest-starting gap).
    Consecutive adds chain forward. An **empty** non-today day falls back to a
    **09:00** morning block.
- **End** = `start + 60`, **clamped to the gap's end** so the default block never
  overlaps the following slice. A narrow gap yields a shorter block that fills the gap
  exactly; a wide gap keeps the full hour.

When there are no planned slices, or the day is fully planned (no gap),
`nextFreeSlot` returns null and the caller falls back to a 1-hour block (the live
minute when viewing today, otherwise 09:00). This applies to the "+" / "Add time
block" entry point; the context-menu "add here" path (tapping an explicit minute on
the canvas) keeps its fixed 1-hour default since the user has chosen the exact spot.

This supersedes the MER-284 nearest-slice-end behaviour for the add-slice entry point.
Implemented in SliceFocusFE under MER-290.

## Alternatives Considered

- **Keep nearest-slice-end (MER-284).** Rejected: it is the source of both the
  non-advancing default and the overlap.
- **Closest gap in either direction (nearest by clock distance, forward or backward).**
  Rejected for day-planning: a gap just behind "now" is rarely what the user wants
  when scheduling the rest of the day; forward-only matches the mental model.
- **Keep a fixed 60-minute end (no clamping).** Rejected: a block can still overflow
  into the next slice and be rejected by the backend.
- **Skip to the next gap wide enough for a full 60 minutes.** Rejected: it silently
  jumps past usable shorter gaps; clamping to fill the nearest gap is more predictable
  and keeps the start where the user expects.
- **For non-today days: use today's clock / first gap from 00:00 / a fixed morning
  hour.** Rejected. Today's clock is the original bug (state from the wrong day's
  pie). "First gap from 00:00" fills pre-dawn gaps before the planned content. A fixed
  hour ignores what's already planned. Appending after the plan matches the sequential
  day-planning model, with 09:00 only as the empty-day seed.

## Consequences

- Sequential day-planning works with one tap per block — no repeated start edits,
  whether planning today or a future day.
- Defaults are computed against the **pie on screen**, so planning a future day no
  longer leaks the current day's clock state.
- The default never overlaps an existing slice, eliminating a class of backend
  overlap rejections for the add-slice flow.
- Gap logic stays centralized in `withUnplannedGaps`; `nextFreeSlot` is a thin,
  well-unit-tested consumer of it (chain-forward, append-after-plan, gap clamp,
  midnight wrap, empty and fully-planned days).
- The context-menu "add at tapped minute" path is intentionally unchanged, so behaviour
  diverges slightly between the two add entry points by design.
