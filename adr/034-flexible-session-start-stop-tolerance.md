# ADR-034: Flexible session start/stop with ±5 minute tolerance

**Status:** Accepted
**Date:** 2026-03-25
**Deciders:** Mert Ertugrul

## Context

Users schedule time blocks (slices) at specific times but rarely start or stop exactly on the minute. A user who scheduled "Deep Work" at 18:00 might sit down at 17:58 or 18:03. Currently, the focus session starts but the slice boundaries remain unchanged, creating a mismatch between the schedule and actual focus time.

Worse, if the user is more than a few minutes early, the current time falls outside the slice's range, which can cause issues with the slice-end watcher and the "Start Focus" visibility logic.

Users also want to start sessions early (e.g., 17:50 for an 18:00 slice) or late (e.g., 18:10), and the slice should adjust to reflect when they actually started.

## Decision

Introduce a **±5 minute tolerance window** around slice boundaries when starting or stopping a focus session:

### On session start

| Scenario | Gap from slice start | Action |
|----------|---------------------|--------|
| Within tolerance | ≤ 5 min early or late | Start session as-is, no slice modification |
| Early start | > 5 min before slice start | Start session, update slice `startTime` to now |
| Late start | > 5 min after slice start | Start session, update slice `startTime` to now |

### On session stop

| Scenario | Gap from slice end | Action |
|----------|-------------------|--------|
| Within tolerance | ≤ 5 min before or after slice end | Stop session as-is, no slice modification |
| Early stop | > 5 min before slice end | Stop session, update slice `endTime` to now |
| Late stop | > 5 min after slice end | Stop session, update slice `endTime` to now |

### Key design choices

- **Tolerance is 5 minutes** — covers natural variance (bathroom break, getting coffee) without allowing large drift to go unrecorded.
- **Slice is modified, not the session** — the schedule adapts to reality. The session records actual start/stop times independently.
- **Slice update is best-effort** — if the PATCH fails (e.g., overlap conflict), the session still starts/stops normally. The slice mismatch is cosmetic, not blocking.
- **FE-only logic** — the tolerance check and slice update happen in the FE presentation layer. The backend slice PATCH endpoint already exists (`PUT /api/v1/slices/{id}`).
- **No new backend API needed** — reuses existing `updateSlice()`.

## Alternatives Considered

- **Strict boundaries**: Require exact timing. Rejected — terrible UX for a productivity app. Users shouldn't stress about starting at exactly 18:00:00.
- **Always adjust**: Update slice boundaries on every start/stop regardless of gap. Rejected — unnecessary API calls and visual churn for 1-2 minute differences.
- **Configurable tolerance**: Let users set their own tolerance in settings. Rejected — over-engineering for v1. 5 minutes is a sensible default. Can revisit if users request it.
- **Backend-side tolerance**: Move the logic to the backend's `startSession`/`completeSession` endpoints. Rejected — adds coupling between session and slice APIs. The FE already has the slice data and can make the PATCH independently.

## Consequences

- Users can start sessions early or late without friction — the schedule adapts
- Actual vs Planned analytics become more accurate (planned times reflect reality)
- Slice-end watcher works correctly when sessions start early (slice boundary moves with the user)
- Quick Focus sessions are unaffected (they create slices at the current time by design)
- The 5-minute threshold may need tuning based on user feedback
- Concurrent edits: if a user edits a slice on another device while starting a session, the PATCH could conflict — handled gracefully by ignoring the failure

*Related: MER-156, MER-158*
