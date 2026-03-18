# ADR-011: Client-side Pomodoro timer with server-side session bookkeeping

**Status:** Accepted
**Date:** 2026-03-05
**Deciders:** Mert Ertugrul

## Context

The initial architecture ran the Pomodoro state machine entirely server-side (pause/resume/advance endpoints), but this caused timer drift issues when the app was backgrounded (timer showed 0 instead of correct remaining time), required complex phaseStartedAt/remainingSeconds sync, and made the timer unusable offline.

## Decision

Move the Pomodoro state machine (WORK/SHORT_BREAK/LONG_BREAK transitions, cycle counting, pause/resume) to a client-side FocusTimerController in pure Dart. The server only handles session start (for conflict prevention via unique DB constraint) and session completion (POST /complete with summary data). Deprecated server-side real-time endpoints were removed.

## Alternatives Considered

- **Fully server-driven state machine**: Rejected due to timer drift, offline fragility, and polling overhead.
- **Hybrid approach with server-authoritative phase tracking**: Too complex for the observed benefits.

## Consequences

- Timer works perfectly offline and when app is backgrounded
- Eliminated 4 server-side endpoints and the 30s polling timer
- Session data is eventually consistent (summary posted on completion)
- Lost ability to enforce server-side phase durations, but gained instant UI responsiveness and offline resilience
- Required adding an action queue for offline session actions

*Related: MER-127, MER-128, MER-129, MER-130, MER-131, MER-234, MER-123*
