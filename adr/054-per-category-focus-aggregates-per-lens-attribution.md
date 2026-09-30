# ADR-054: Per-category focus aggregates use per-lens (non-partition) attribution

**Status:** Accepted
**Date:** 2026-07-03
**Deciders:** Mert Ertugrul

## Context

Categories became first-class (M:N slice tagging, emojis, seeded defaults), but the analytics surfaces (`GET /api/v1/focus-sessions/stats`, `GET /api/v1/reports/trends`) exposed no category dimension. The FE needs to answer "where did my time go, by category" (epic: Analytics refresh).

The modelling question is how to attribute a focus session's minutes when its slice carries **multiple** categories. A session is linked to exactly one slice; a slice can be tagged with N categories. There is no per-category split of the minutes within a session — the tags are a labelling lens, not a time allocation.

## Decision

Attribute each session's **full** focus minutes (and a `+1` session count) to **every** category on its slice — a per-lens view, not a partition.

- New `CategoryBreakdownEntry` schema (`categoryId`, `name`, `emoji`, `focusMinutes`, `sessions`, `sharePercent`), added as `categoryBreakdown` to both `FocusSessionStatsResponse` and `ProductivityTrendsResponse` (period-level; the daily breakdown stays category-agnostic).
- `sharePercent` = category focus minutes ÷ **total** focus minutes across all completed sessions × 100. Because minutes are counted once per category, shares across entries **may sum to more than 100%**. This is documented in the OpenAPI schema description.
- Sessions whose slice has no categories roll into a single **Uncategorized** bucket (`categoryId = null`), always sorted last; category entries sort by focus minutes desc, then name asc.
- A shared `CategoryBreakdownCalculator` computes the breakdown for both endpoints. A new optional `categoryId` filter on `GET /api/v1/focus-sessions` matches sessions whose slice is tagged with that category, via a criteria subquery over `slice_categories`. All queries stay scoped by `userId`.

## Alternatives Considered

- **Partition minutes across a slice's categories (e.g. split evenly).** Rejected: the split would be arbitrary (no signal says a "Deep Work + Learning" session was 50/50), shares would falsely sum to 100%, and totals would no longer match `totalFocusMinutes`. It fabricates precision the data doesn't have.
- **Only aggregate single-category slices; ignore multi-tag slices.** Rejected: silently drops data and confuses users who deliberately multi-tag.
- **A denormalized `category_id` column on `focus_sessions`.** Rejected: sessions are M:N to categories through the slice; a single FK can't represent multi-tagging, and it would drift from the slice's tags.

## Consequences

- FE category charts must treat shares as **overlapping lenses**, not slices of a pie — a stacked/pie chart that assumes shares sum to 100% would misrepresent multi-tagged time. Bar-per-category or explicit "may overlap" framing is appropriate.
- `Σ categoryBreakdown.focusMinutes ≥ totalFocusMinutes`; only the Uncategorized bucket plus single-category slices sum exactly. Consumers must not reconcile category minutes against the grand total.
- The breakdown resolves each session's slice (with categories eagerly loaded) — an extra scoped `slice_categories` read per stats/trends call, bounded by the number of distinct slices in range.
- Adding a category to a slice retroactively changes historical breakdowns (they reflect current tags, not tags-at-session-time). Acceptable for a labelling lens; noted here so it isn't mistaken for a bug.
