# ADR-032: Server-Synced User Preferences with Local Fallback

**Status:** Accepted
**Date:** 2026-03-18
**Deciders:** Mert Ertugrul

## Context

User preferences (theme, soundscape, timer durations, notification settings) were initially local-only via `SharedPreferences`. Users expected settings to follow them across devices (phone, tablet, desktop), requiring a synchronization mechanism.

## Decision

Sync preferences to the backend via `GET`/`PUT /api/v1/users/preferences`. A `PreferencesSyncService` orchestrates the flow:

1. **On initialize**: Fetch remote preferences and reconcile with local state.
2. **On local change**: Push to backend (fire-and-forget). Mark dirty if offline; flush on reconnect.
3. **Storage format**: A single row per user stores all preferences as flat JSON fields (not a key-value store).

Conflict resolution strategy: remote wins, except when local state has user-customized values and remote returns defaults (indicating the user has not yet synced from another device).

## Alternatives Considered

- **Key-value store per setting**: More granular sync but introduces complex merge logic and higher request volume.
- **Real-time sync via WebSocket**: Overkill for settings that change infrequently (minutes/hours apart).
- **Local-only (SharedPreferences)**: Simplest approach but fails the cross-device requirement.

## Consequences

- Cross-device preference consistency without requiring manual export/import
- Requires conflict resolution logic (remote-wins with default detection)
- Adds network dependency for preference changes, mitigated by dirty-flag retry on reconnect
- Backend must validate all preference fields and return 400 for unknown keys
- Offline-first: the app always works with local state; sync is best-effort

*Related: MER-156, MER-157, MER-227, MER-230, MER-231*
