# SliceFocus Launch Roadmap

*Created: 2026-03-14 | Last updated: 2026-03-14 | Migrated from Linear: 2026-03-17*

---

## Current State

| Milestone | Progress | Status |
| -- | -- | -- |
| M1 - The "Visual Core" (MVP) | 41/41 | **Complete** |
| M2 - The "Focus Engine" | 20/20 | **Complete** |
| M3 - The "Sync & Alert" Layer | 25/25 | **Complete** |
| M5 - User Testing & Cosmetics | 18/19 | **Complete** |
| M4 - The "Refinement & Analytics" | 12/17 | **In Progress** (all BE done, 5 FE remain) |

**Overall: 223/228 issues closed (98%). 5 open issues remain (0 BE + 4 FE + 1 Infra).**

---

## Work Queue — Pick from top, work down

### Launch-Critical (High Priority)

| # | Title | Side | Pts | Status | Depends On |
| -- | -- | -- | -- | -- | -- |
| 1 | ~~Add "actual vs planned" report endpoint~~ | BE | 5 | **Done** | — |
| 2 | Actual vs planned report UI | FE | 5 | Backlog | #1 |
| 3 | Sync user preferences with backend | FE | 3 | Backlog | User preferences API |
| 4 | ~~Milestone quality gate: tests, docs~~ | BE | 3 | **Done** | All M4 BE done |

### Launch-Important (Medium Priority)

| # | Title | Side | Pts | Status | Depends On |
| -- | -- | -- | -- | -- | -- |
| 5 | Cloud Run min-instances=1 | Infra | 1 | Todo | — |
| 6 | ~~Add productivity trends endpoint~~ | BE | 3 | **Done** | — |
| 7 | Productivity trends visualization | FE | 5 | Backlog | #6 |
| 8 | ~~Add OAuth2 account linking flow~~ | BE | 5 | **Done** | — |
| 9 | OAuth2 account linking UI in settings | FE | 3 | Backlog | #8 |
| 10 | Configure Apple Sign-In | FE | — | **Blocked** | Apple Dev Portal |

### Post-Launch (Low Priority / Deferrable)

| # | Title | Side | Pts | Status | Depends On |
| -- | -- | -- | -- | -- | -- |
| 11 | ~~Add soundscape catalog endpoint~~ | BE | 2 | **Done** | — |
| 12 | Soundscape picker UI | FE | 3 | Backlog | #11 |
| 13 | ~~Add CI pipeline quality gates~~ | BE | 2 | **Done** | — |
| 14 | API-First Auth Flow (User Story) | Story | 5 | Todo | #8, #10 |

**Total remaining: ~25 points (1 Infra + 4 FE + 1 blocked)**

---

## Roadmap Phases

### Phase 1 - Core Preferences and Analytics (Launch Blockers) - BE COMPLETE

**Target: Mar 14 - Mar 18** | BE done, FE ready to start

### Phase 2 - Production Hardening (Launch Day) - BE COMPLETE

**Target: Mar 19 - Mar 20** | BE done, FE ready to start

### Phase 3 - Auth and Identity (Post-Launch Sprint) - BE COMPLETE

**Target: Mar 21 - Mar 28** | BE done, FE ready to start

### Phase 4 - Polish (Post-Launch) - BE COMPLETE

**Target: Mar 29+** | BE done, FE ready to start

---

## Launch Readiness - What's Already Working

* Full Pomodoro lifecycle (work, short break, long break)
* Pie chart time visualization with slice CRUD
* Real-time WebSocket sync across devices
* Push notifications (FCM)
* Menu bar timer (macOS), lock screen timer (mobile)
* Session history, stats, and analytics dashboard
* Multi-platform FE (Flutter: macOS, iOS, Android)
* Cloud Run + Neon PostgreSQL deployment with CI/CD
* User profile management (display name editing)
* Theme selection (light/dark)
* Quick-start focus from timeline
* User preferences API (theme, soundscape, notifications)
* Actual vs planned productivity report
* Weekly/monthly productivity trends
* OAuth2 account linking (Google/Apple)
* Soundscape catalog
* CI quality gates (Checkstyle)

## Launch Criteria

* **Minimum viable launch** (after Phase 1): preferences sync + actual-vs-planned report + quality gate pass — **BE READY**
* **Recommended launch** (after Phase 2): add cold-start fix + trends for a polished production experience — **BE READY, pending FE + infra**
