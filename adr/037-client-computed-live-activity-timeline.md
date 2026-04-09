# ADR-037: Client-Computed Pomodoro Timeline for Live Activity

**Status:** Accepted — amended: phase transition re-rendering requires Cloud Tasks APNS push trigger (ADR-038) since iOS Live Activities cannot re-evaluate the view body without an external content-state update
**Date:** 2026-04-07
**Deciders:** Mert Ertugrul

## Context

ADR-036 established a direct APNs HTTP/2 client for updating the iOS Live Activity on the lock screen during focus sessions. The initial approach required the **server to push an APNS update on every phase transition** — every time a WORK phase ends and SHORT_BREAK begins, or vice versa. This led to two complex server-side scheduling implementations:

1. **Polling scheduler** (30s fixed-rate `@Scheduled`): Queried the DB every 30 seconds for sessions with expired phases. Wasteful, imprecise (0–30s latency), and redundant DB load.

2. **Event-driven per-session TaskScheduler**: Scheduled an in-memory timer for each session's exact phase expiration. More precise but added complexity — lifecycle hooks in every `FocusSessionService` method, startup recovery from DB, multi-instance deduplication via optimistic locking.

Both approaches treated the server as the authoritative source for phase transitions, contradicting ADR-011 which established that **the Pomodoro state machine runs client-side** and the server only handles bookkeeping.

The core insight: the Pomodoro formula is **deterministic**. Given `sessionStartTime`, `workDurationMinutes`, `breakDurationMinutes`, `longBreakDurationMinutes`, `cyclesBeforeLongBreak`, and `totalPausedSeconds`, any client can compute exactly which phase the user is in and how many seconds remain — at any point in time. There is no ambiguity.

## Decision

Instead of the server pushing every phase transition via APNS, **send the Pomodoro configuration to the Live Activity widget once** (on session start and on mutations). The widget computes phase transitions locally for up to 8 hours (the maximum session duration).

### What the widget receives

The APNS content-state contains the session **config**, not derived state:

```json
{
  "sessionStartTime": 1712480000,
  "workDurationSeconds": 1500,
  "breakDurationSeconds": 300,
  "longBreakDurationSeconds": 900,
  "cyclesBeforeLongBreak": 4,
  "completedCyclesAtStart": 0,
  "totalPausedSeconds": 0,
  "isPaused": false,
  "maxSessionSeconds": 28800
}
```

The widget uses this config to compute the full Pomodoro timeline autonomously:
- 0:00–25:00 → WORK
- 25:00–30:00 → SHORT BREAK
- 30:00–55:00 → WORK
- ... and so on for up to 8 hours

### When APNS pushes are sent

APNS is only needed for **mutations** — events that change the inputs to the formula:

| Event | APNS Push | Content |
|-------|-----------|---------|
| Session start | Yes (initial) | Full config |
| Pause | Yes | `isPaused: true`, `totalPausedSeconds` updated |
| Resume | Yes | `isPaused: false`, `totalPausedSeconds` updated |
| Cancel/Complete | Yes (end event) | Dismiss Live Activity |
| Phase transition | **No** | Widget computes locally |
| 8-hour limit | Yes (end event) | Dismiss Live Activity |

This reduces APNS pushes from ~4–7/hour (one per phase) to **0–2 per session** (only on pause/resume/cancel).

### Cross-platform consistency

All platforms compute the same timer from the same inputs:
- **iOS widget**: Uses the config from APNS content-state
- **iOS/Android app** (foreground): Uses the config from the session response
- **macOS/web** (future): Uses the config from WebSocket session events

When a user unlocks their phone and opens the app, `GET /active` returns the session with its current config. The app recomputes the timer — same formula, same answer as the widget was showing. The server's reactive auto-advance on fetch (ADR, #247) keeps the entity state consistent for history/stats.

### 8-hour session cap

The widget autonomously ends after `maxSessionSeconds`. The server can optionally schedule a single cleanup task at the 8-hour mark to send an end notification, or handle it reactively when the client next contacts the server.

## Alternatives Considered

1. **Server pushes every phase transition (ADR-036 original):** Requires server-side scheduling (polling or event-driven), adds infrastructure complexity, and contradicts ADR-011's client-side timer philosophy. Rejected — unnecessary server involvement for deterministic computation.

2. **Event-driven per-session TaskScheduler:** Precise timing but complex — lifecycle hooks in 7 methods, startup recovery, multi-instance deduplication. Rejected — the widget can do this locally without any server involvement.

3. **Polling scheduler (30s fixed-rate):** Simpler than event-driven but imprecise, wasteful, and scales poorly. Rejected — worst of both worlds.

4. **Full timeline array in content-state:** Send a pre-computed array of all phase boundaries. Rejected — the config is smaller and the widget can compute phases from the same deterministic formula.

## Consequences

### Easier
- **Zero server-side scheduling** for phase transitions — no polling, no timers, no DB queries
- **APNS budget reduced** from ~4–7 pushes/hour to 0–2 per session — well within any Apple limit
- **Offline resilient** — widget works even if the server is unreachable after session start
- **Scales infinitely** — no server-side work between user actions regardless of concurrent sessions
- **Consistent with ADR-011** — the Pomodoro state machine runs client-side; the server handles bookkeeping

### Harder
- **Widget complexity increases** — the iOS widget must implement the Pomodoro formula (phase computation, cycle tracking, long break detection)
- **All platforms must implement the same formula** — any platform showing the timer needs the deterministic computation logic, risk of drift if implementations diverge
- **Pause state requires APNS** — if a user pauses from another device, the widget needs an APNS push to know about it (same as before)

### Migration

- Revert `PhaseAutoAdvanceScheduler` (merged in #264) — no longer needed
- Revert debug APNS logging (#261) — temporary, no longer needed
- Close event-driven scheduler PR (#266) — superseded by this approach
- Keep APNS infrastructure (#258, #260) — still needed for mutation pushes
- FE: Update `ContentState` struct to accept config instead of derived phase state (#262)

*Related: ADR-011 (client-side timer), ADR-036 (APNS for Live Activity), #250 (parent), #263 (server-side scheduling — superseded)*
