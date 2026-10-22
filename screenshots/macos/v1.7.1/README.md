# macOS App Store Screenshots — v1.7.1

App Store Connect slot: **Mac, 2560x1600**. Composited canvases: a caption over the
app window (1440x870 debug window on a 2560x1600 Retina display) on a light gradient.
Made by the pipeline in `SliceFocusFE/tool/screenshots/` (SliceFocusFE#608) against
the dev backend with a seeded demo account.

| # | File | Caption | Screen |
|---|------|---------|--------|
| 1 | `01-day-canvas.png` | Your whole day, at a glance | Day canvas: a founder's weekday on the 24h pie, timeline below |
| 2 | `02-timer.png` | Focus with a Pomodoro timer | Running session on the current block, expanded timer (WORK, cycle 1/4) |
| 3 | `03-settings-focus-shield.png` | Focus Shield: block the sites you choose | Settings, Focus Shield on: Safari extension on, 14 websites chosen |
| 4 | `04-block-list.png` | Your list, your rules | Block list sheet: website field, Quick add presets, social and video sites |
| 5 | `05-actual-vs-planned.png` | Your plan vs. what happened | Actual vs Planned over the last 7 full days, 86% adherence |
| 6 | `06-analytics.png` | See where your focus goes | Analytics, this week: sessions, focus minutes, daily chart, by category |
| 7 | `07-history.png` | Every session, on record | Session history by date, one day expanded |
| 8 | `08-templates.png` | Start from a template | Template picker with the starter templates |
| 9 | `09-fill-gap.png` | Fill the gaps with a tap | Gap-fill sheet on a free slot of the day |

Notes:
- `08-templates.png` was shot with the picker's provider-scope fix applied locally
  (not in 1.7.1): in 1.7.1 the "My templates" row reads "Your templates aren't
  available right now." on every account.
- The app ran in UTC for the shoot, so Actual vs Planned shows the planned times
  (the report's `toLocal()` shifts them for users outside UTC).
