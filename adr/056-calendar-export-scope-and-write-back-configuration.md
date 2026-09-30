# ADR-056: Calendar-export scope filter, write-back toggle, and event options

**Status:** Accepted
**Date:** 2026-07-06
**Deciders:** Mert Ertugrul

## Context

One-way slice → calendar export ([ADR-046](046-calendar-export-device-local-id-map.md)) and two-way reconciliation ([ADR-047](047-two-way-calendar-reconciliation-snapshot-echo-guard.md)) shipped as a single opt-in toggle: turning export on mirrored **every** planned slice to the calendar and simultaneously let Calendar-app edits/deletions write back onto slices. Two gaps surfaced in use (FE #445, #446):

- **No scope control.** Rest blocks a user would never want on their work calendar — "Wind-down", "Lunch", "Unwind" — were exported alongside focused work, and there was no way to keep a whole category (e.g. personal) off the calendar.
- **Write-back was not separable.** A user who wanted their plan mirrored out had no way to prevent the reverse: an edit to the mirrored event silently mutating (or deleting) the underlying slice.
- **No event options.** Exported events had no alert and used the calendar's default availability.

The guiding principle was **progressive disclosure with useful defaults** — add configurability without overwhelming the user, so the common case needs no attention.

## Decision

Extend the device-local `CalendarExportPreferences` (still not server-synced, per ADR-046) with four fields and enforce them at the existing single choke point, the `CalendarExportSliceRepository` decorator.

- **Scope filter — skip rest by default.** `includeBreaks` (default **false**) drops slices whose label is classified rest by a new shared `RestActivityLabels` classifier (`lib/src/shared/`), extracted from the suggestions feature's former private `_breakLabels` so both features share one vocabulary. `excludedCategoryIds` stores **exclusions** (not inclusions) so a newly created category exports by default; a slice tagged with any excluded category is skipped; uncategorized slices always export. The single predicate `CalendarExportPreferences.allowsExport(label, categoryIds)` is consulted by the decorator's upsert path, the enable-time/day resync, and — via the stored snapshot — the sweep.
- **Grandfathering.** `fromJson` defaults `includeBreaks` to the stored `enabled` value when the key is absent: a user who already had export on before this change keeps their breaks exported, so upgrading never silently deletes already-mirrored lunch/wind-down events. Fresh installs (no stored JSON) get the const default `false`.
- **Transition = retraction, mapping-first.** When a slice that *was* exported becomes out of scope (category excluded, label edited to a rest label, or scope settings changed), its event is retracted. `CalendarExportReflector.retract` drops the slice→event id-map entry **before** deleting the event — load-bearing because the slice still exists, so removing the mapping first makes the event invisible to reconciliation and prevents our own retraction from being misread as a user deletion (which would delete the slice). A transient failure enqueues a durable delete carrying the known event id.
- **Day resync.** `exportDay` (fired on enable and on scope-settings change) now also **sweeps** mappings whose slice is not in the current day's list, retracting any the filter now excludes; the per-slice snapshot gained `categoryIds` so the sweep can evaluate the category filter. Snapshot-less legacy mappings are left untouched. Retroactivity is bounded: the current day is resynced, other days re-export on their next slice edit — no historical sweep.
- **Write-back as a separate sub-toggle.** `writeBackEnabled` (default **true**, preserving ADR-047 behavior) gates reconciliation independently: the service now runs only while `enabled && writeBackEnabled`.
- **Event options (FE #446).** `alertMinutes` (null = none, default) and `availability` (busy/free, default busy) extend the EventKit bridge; availability is applied only when the target calendar's `supportedEventAvailabilities` allows it, else left at the calendar default. Options apply to new/updated events plus the current-day resync — no historical sweep. Alarms on app-managed events are replaced wholesale on each export (a manually added alarm is overwritten on the next slice edit).

## Alternatives Considered

- **Store inclusions instead of exclusions.** Rejected: a new category would silently *not* export until the user found and enabled it — surprising, and the opposite of the "sensible default" goal. Exclusions make "export everything except X" the natural default.
- **Make the suggestions catalog's break-label set public and import it from the calendar feature.** Rejected as the wrong dependency direction (suggestions is a consumer feature, not shared vocabulary); the classifier belongs in `lib/src/shared/`.
- **Keep write-back coupled to export.** Rejected: the two are genuinely different consents — mirroring out vs. letting the calendar mutate the plan. A separate default-on toggle preserves behavior while giving an escape hatch.
- **Retroactively re-apply scope/options to all historical events.** Rejected as disproportionate; current-day resync plus lazy per-edit convergence matches the existing enable-time backfill scope.

## Consequences

- The default export is quieter and safer: focused work mirrors out, rest stays private, and the calendar cannot silently rewrite the plan unless the user leaves write-back on (which, by default, it is — no behavior change for existing users).
- One enforcement predicate (`allowsExport`) keeps the write path, resync, and sweep consistent; adding a future scope dimension is a change in one place.
- The id-map snapshot now carries `categoryIds`; this is backward-compatible (absent on legacy entries, which the sweep skips) but means the map schema grew — noted here so future readers don't treat the field as accidental.
- Re-including a previously excluded category/breaks restores the current day on resync but relies on later edits to restore other days — an accepted, documented limitation shared with the original enable-time backfill.
- A calendar-app rename of an exported event *to* a rest label does not itself retract the event; only the next in-app edit re-evaluates scope. Accepted asymmetry.
