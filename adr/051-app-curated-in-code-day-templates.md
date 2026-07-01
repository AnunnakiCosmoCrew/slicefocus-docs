# ADR-051: App-curated, in-code day templates

**Status:** Accepted
**Date:** 2026-07-01
**Deciders:** Mert Ertugrul

## Context

A blank 24-hour canvas is a cold start for a new user (FE #323). Starter "templates" — named routines
like *Maker Schedule* or *9-to-5 + Gym* — let someone drop a full day of slices in one tap and then
edit. The question is where those templates come from and how applying one interacts with the existing
slice write path.

The templates are also the raw material for gap-fill suggestions
([ADR-045](045-gap-fill-activity-suggestions.md)): the suggestion catalog is derived from the same
labels, so a second, independent source of activity definitions would immediately drift.

## Decision

Define templates as a small, hardcoded, app-curated library in Dart and apply them through the ordinary
slice repository. Key design choices:

- **In-code definitions, not a backend catalog.** `TemplateLibrary.all` is a `const` list of five
  `DayTemplate`s (`templates/domain/`), each a name, description, emoji, and a `const` list of
  `DaySlice`s with hardcoded palette colours chosen for a cohesive per-template look. No API, no fetch,
  no cache — the library ships in the binary.
- **Apply = loop `saveSlice`, with rollback.** Using a template persists each of its slices through the
  existing `SliceRepository.saveSlice` (the repository pattern, not a new bulk endpoint). If a save
  fails partway, the flow attempts to restore the day's original slices before surfacing the error, so a
  half-applied template doesn't leave the day in a mangled state.
- **Single source of activity labels.** The suggestion catalog (ADR-045) is derived from
  `TemplateLibrary` rather than maintaining its own list, so template labels and suggested activities
  stay in lockstep by construction.
- **Confirm-before-replace UX.** The picker sheet browses templates, expands to preview each one's slices
  with times, and warns before overwriting a day that already has slices.

## Alternatives Considered

- **Backend-served template catalog.** Would let us add or A/B templates without shipping an app update,
  and eventually offer user-created/shared templates. Rejected for launch: it needs an endpoint, a
  schema, caching, and versioning to serve five static routines. The in-code path is a clean seam to
  swap in a backend source later (`TemplateLibrary.all` becomes a fetch) without changing the apply flow.
- **A dedicated bulk "apply template" API.** One request instead of N `saveSlice` calls, atomic on the
  server. Deferred: it duplicates slice-write logic on the backend and bypasses the client's existing,
  well-tested save path; the per-slice loop with client-side rollback is good enough for a five-to-ten
  slice template.
- **A separate hardcoded activity list for suggestions.** Simpler wiring for ADR-045, but two lists of
  "things you might do in a day" would inevitably diverge. Deriving suggestions from the templates keeps
  one source of truth.
- **Ship templates as seed data written once on first launch.** Pollutes the user's real slices with
  starter content they didn't choose and is awkward to update; templates should be an explicit, repeatable
  action, not a one-time seed.

## Consequences

- New users get a usable day in one tap; the five routines cover common shapes (maker, 9-to-5, student,
  sprint, wind-down).
- Templates and gap-fill suggestions cannot drift because they share `TemplateLibrary` as their source.
- Curating or fixing a template requires an app release — acceptable at this stage, and the apply flow is
  already isolated from the source so a backend catalog can replace the `const` list later with no churn
  downstream.
- Applying a template is not server-atomic; a mid-apply failure is handled by a best-effort client
  rollback rather than a transaction, so a pathological failure could still leave the day partially
  changed.

*Related: FE #323, [ADR-045](045-gap-fill-activity-suggestions.md), [ADR-008](008-riverpod-state-management.md)*
