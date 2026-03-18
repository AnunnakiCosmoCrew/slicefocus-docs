# ADR-027: Offline Action Queue with Automatic Retry

**Status:** Accepted
**Date:** 2026-03-18
**Deciders:** Mert Ertugrul

## Context

Focus session completions can fail if the user goes offline mid-session. A Pomodoro cycle takes 25+ minutes of dedicated work — losing that completion data because of a transient network issue is unacceptable. The app needs a mechanism to persist failed mutations and replay them when connectivity is restored.

## Decision

Implement an `ActionQueue` that captures failed completions and persists them to SharedPreferences, keyed per user to prevent cross-account data leakage. The queue listens to `ConnectivityService.onlineStream` and auto-flushes pending actions when the device comes back online.

Key design choices:
- **Serialized flushing:** Actions are replayed one at a time in FIFO order to prevent race conditions and duplicate submissions
- **Per-user isolation:** Queue entries are namespaced by user ID so switching accounts never replays another user's actions
- **SharedPreferences storage:** Consistent with ADR-029; no additional persistence dependency needed
- **Automatic retry on reconnect:** The queue subscribes to connectivity changes and flushes without user intervention

## Alternatives Considered

- **Block actions when offline:** Simple to implement but terrible UX — the user completes a 25-minute focus session only to be told it won't be saved.
- **Full offline-first architecture with CRDTs:** Guarantees eventual consistency for all data, but massively over-engineered for a queue of failed completions. The app is online-first by design.
- **Fire-and-forget:** Silently dropping failed requests loses user data. Unacceptable for earned pomodoro cycles.

## Consequences

- Guarantees that focus session completions are eventually synced to the backend, even across app restarts
- Adds complexity to testing — connectivity must be mocked, and queue drain behavior needs verification
- The JSON serialization format of queued actions becomes a contract; changes require migration logic
- Queue size is bounded in practice (a user can only complete so many sessions offline) so SharedPreferences capacity is not a concern

*Related: MER-123, MER-120*
