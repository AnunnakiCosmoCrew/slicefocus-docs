# MER-155 Execution Plan — M3 Quality Gate & FE Sub-issues

*Migrated from Linear: 2026-03-18*

Execution order for MER-155 sub-issues. Phases are ordered by priority, dependencies, and complexity.

---

## Phase 1 — Quick wins & foundational FE work (no backend blockers)

- [X] MER-202 — Reduce main pie canvas size so timer panel is visible (1pt)
- [X] MER-200 — Cap max size of analytics & session history widgets (1pt)
- [X] MER-197 — Improve create-slice modal layout & duration styling (1pt)
- [X] MER-190 — Add user info display in AppBar (avatar + name) (1pt)
- [X] MER-206 — Support dev/test/prod environment configurations for FE (2pt, High)

## Phase 2 — Medium FE features

- [X] MER-193 — Distinct default colors per slice + user color customization (2pt)
- [X] MER-192 — Show slice title on hover/tap when too narrow (2pt)
- [X] MER-210 — Redesign slice list as Cal Newport-style time blocking timeline (3pt, High)
- [X] MER-213 — Show current time indicator on pie canvas and time block timeline (2pt, High)
- [X] MER-212 — Start focus session by tapping time block entry in timeline (1pt, Medium)
- [X] MER-191 — Add account settings page (3pt)
- [X] MER-194 — Theme selection (light/dark + accent variants) (2pt)

## Phase 3 — Larger FE features

- [X] MER-211 — Add pinch-to-zoom and scroll-wheel zoom on pie canvas (5pt, Medium)
- [X] MER-198 — Play sound/notification when focus session phase ends (2pt)
- [X] MER-199 — Auto-advance option for pomodoro phases (1pt, depends on MER-198)
- [X] MER-214 — Use light theme as default (1pt)
- [X] MER-215 — Style unplanned slice as empty grid notebook page (2pt)
- [X] MER-201 — Restructure session history with grouped table view (3pt)
- [X] MER-196 — Slice detail screen with pomodoro division animation (5pt)
- [X] MER-204 — Notify user when focus session is stopped by the app (2pt)
- [X] MER-216 — Auto-scroll to timer panel when focus session starts (1pt)
- [X] MER-205 — Show focus session timer on lock screen (mobile) (5pt, High)
- ~~MER-195 — Custom theme builder (user-defined colors) (5pt, Low) — **Cancelled**~~

## Phase 3b — macOS menu bar timer (desktop)

Show remaining time and cycle info in the macOS menu bar so users can track their focus session while working in other apps.

- [X] MER-220 — Set up macOS NSStatusItem with Platform Channel — Swift side (2pt)
- [X] MER-221 — Create MenuBarTimerService — Dart side (2pt, depends on MER-220)
- [X] MER-222 — Integrate menu bar timer with focus session lifecycle (2pt, depends on MER-221)

## Bugs

- [X] MER-209 — Slice overlap 409 error closes create dialog instead of showing inline feedback (High)

## Phase 4 — Backend docs & QA (after M3 backend issues complete)

- [X] MER-172 — Update system-design.md — WebSocket & push notification sections (2pt)
- [X] MER-173 — Update system-design.md — session history/analytics & data design (1pt)
- [X] MER-174 — Update architecture.md for Firebase infrastructure (1pt)
- [X] MER-175 — Verify OpenAPI contract completeness for M3 (1pt, High)
- [X] MER-176 — Ensure BDD/Cucumber coverage for all M3 endpoints (2pt, High)
- [X] MER-223 — End-to-end WebSocket testing (FE <> BE) (3pt, High)
- [ ] MER-177 — Final M3 regression run and coverage verification (1pt, High — LAST)

---

## Notes

* Phase 1–3 are all FE tasks and can proceed independently of backend M3 blockers.
* Phase 3b (menu bar timer) is macOS-only, uses native NSStatusItem via platform channel (no third-party packages).
* Phase 4 is blocked by MER-144 through MER-154 (core M3 backend issues).
* MER-223 (WebSocket E2E testing) should be done before MER-177 (final regression) as it may surface issues.
* MER-177 (final regression) must be the very last item before closing MER-155.
