# Promotion plan for 1.7.4: first weeks

*Created: 2026-10-29 | Release: [SliceFocusFE#641](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/641) | Founder check: public (every post below is a draft; the founder approves and posts it, Claude never does)*

## Where this starts

- 1.7.4 is live on iPhone, iPad and Mac (approved and released 2026-10-29, 05:19 UTC). A first launch opens on a guest account, so people can try the app without signing up.
- The Turkish and English (U.K.) listings and the new subtitle "Pomodoro Planner & App Blocker" are live.
- The plan agreed on 2026-10-26 still holds: the first session was the problem, 1.7.4 is the fix, and promotion should bring in a cohort we can measure before anything bigger.
- Not pursued: Setapp, Product Hunt, Turkish community posts (for now), LinkedIn (never).

## The options picked, in priority order

| Priority | Channel | What it takes | Why it is here |
| --- | --- | --- | --- |
| 1 | **Reddit update post**, r/ProductivityApps, plus a reply in the June thread | One post, 2–3 h of replies right after | The June post got a real conversation there. A "what I changed" story fits the room and brings the first guest cohort |
| 2 | **Apple featuring nomination**, App Store Connect | 20 min form | Free. Apple asks for notice, so file 1.7.4 now as an enhancement and every later release 2+ weeks ahead |
| 3 | **Mac communities**, r/macapps (or r/MacOSApps) | One post with flair and disclosure | The Safari website blocker, the app shield and the menu bar timer are Mac-only and the main difference from iPhone-only rivals |
| 4 | **Reuse the demo videos** from `slicefocus-web` | Choose clips; one optional re-take showing guest start | Posts with a clip of the clock running itself explain the app in 15 s |
| 5 | **ADHD and study audience**: r/pomodoro post (disclosed makers allowed); link-free, disclosed replies in r/ADHD, r/GetStudying, r/productivity | 2–3 replies a week, one post in week 2 | Small but well-matched users, and every conversation is user research. Never claim the app treats ADHD |
| 6 | **slicefocus.app**: say "free, no account needed to start" on the home page; finish the Session comparison page ([slicefocus-web#30](https://github.com/AnunnakiCosmoCrew/slicefocus-web/issues/30)) | Approving copy | Every post links here; comparison pages pay back slowly, so finish them early |
| 7 | **Show HN**, framed as how it is built (radial CustomPainter, server-owned timer shared by iPhone, Mac and lock screen) | A write-up and 3 h of replies | Gated: see "Show HN gate" below. Earliest the week of 16 Nov |

## Day 0, today (Thu 29 Oct): before anything is posted

All founder steps.

1. **Run the guest test on a real iPhone** against the live build (#641). If a fresh install cannot start as a guest, nothing below goes out.
2. **Register `account_state` in GA4** (Admin → Custom definitions → user-scoped dimension `account_state`). GA4 does not fill custom dimensions backwards, so without it Friday's cohort has no guest vs signed-in split.
3. **Make App Store campaign links** in App Store Connect (App Analytics → Acquisition → Campaigns), one per channel: `ct=reddit-pa-1030`, `ct=macapps-1103`, `ct=pomodoro-1110`, `ct=site`. They look like `https://apps.apple.com/app/apple-store/id6761280632?pt=<provider token>&ct=<campaign>&mt=8`.
4. **Read the sidebars** of r/ProductivityApps and r/macapps: karma minimum, how often a developer may post, required flair, promo megathread. Adjust the dates below if a rule says so.
5. **File the featuring nomination** (notes below).

## First week

Times are Türkiye time. US Eastern moves off summer time on Sun 1 Nov, so 8:00 ET is 15:00 on Friday and 16:00 from Monday.

| Day | What | Who |
| --- | --- | --- |
| Thu 29 Oct | Day 0 list above | Founder |
| Fri 30 Oct, 15:00 | **Reddit update post** (draft 1) with the 15 s hero clip; then the reply in the June thread. Stay for 2–3 h of comments | Founder |
| Sat 31 Oct – Sun 1 Nov | Answer every comment within a few hours. Note each reason someone gives for not trying or for dropping the app | Founder |
| Mon 2 Nov | **First readout** (see "What to measure"). Start the 2–3 disclosed replies a week in study and ADHD threads, found by hand | Founder; Claude reads the dashboard and writes the readout |
| Tue 3 Nov, 16:00 | **r/macapps post** (draft 2), gallery of Mac screenshots or the Mac hero clip | Founder |
| Wed 4 Nov | Site copy: "free, no account needed to start" on slicefocus.app, as a `slicefocus-web` issue for an agent; founder approves the wording | Founder opens / approves |
| Thu 5 Nov | **Week-1 readout**; decide the week-2 post (r/pomodoro) and whether a guest-start clip re-take is worth it | Founder + Claude |

## After week 1

- **Week 2 (9–13 Nov):** r/pomodoro post, a disclosed maker's post focused on auto-advance and blocking only during work phases. Session comparison page live. Optional re-take of the hero clip showing guest start (`SliceFocusFE/tool/videos/run.sh`, needs the Mac unlocked).
- **Every Monday:** readout row added to the log below.
- **Next release:** nominate it for featuring at least 2 weeks before it ships.
- **Show HN gate (earliest week of 16 Nov):** at least 100 installs in the 1.7.4 cohorts, the lower end of the 95 % interval for "installed → completed a session" above the pre-1.7.4 rate, and a weekday when the founder can reply for 3 h. If the gate is not met, find where guests drop off and fix that first.

## What to measure

From the **Activation funnel** card in `cosmocrew-metrics` (GA4 events from SliceFocusFE#640), by weekly install cohort, split by `account_state`:

| Step | Event |
| --- | --- |
| Installed | `first_open` |
| Created a slice | `slice_created` (source: canvas, list, template, suggestion) |
| Started a session | `session_started` (kind: slice, quick) |
| **Completed a session (activation)** | `timer_session_completed` |
| Signed in from a guest | `login` / `sign_up` with `account_state` moving from `guest` to `signed_in` |

Plus, each week:
- **Installs per channel:** App Store Connect → App Analytics → Campaigns (the `ct=` links), and the Web Referrer source.
- **Day-1 and day-7 return** for the 1.7.4 cohorts, compared with the pre-1.7.4 cohorts.
- **Ratings count** (the rating prompt fires after the third completed session).
- **Weekly users** against the goal of 20 (3 auto-advanced blocks a week).

Read cohorts as intervals, not single numbers: tens of users per week cannot separate small changes. About 100 installs is enough to see a jump like 15 % → 30 %.

### Log

| Week of | Installs (by channel) | Activation guest / signed in | Day-1 / day-7 | Ratings | Weekly users | Note |
| --- | --- | --- | --- | --- | --- | --- |
| 26 Oct | | | | | | 1.7.4 live 29 Oct |

## Featuring nomination: notes for the form

Type: App Enhancements. Points to make:
- Start without an account: the first launch opens on today's 24-hour clock, and signing in later keeps everything.
- One focus timer across iPhone, iPad, Mac and the lock screen, moving from work to break on its own.
- On the Mac, Focus Shield blocks chosen websites in Safari and covers chosen apps, only during work phases.
- New Turkish App Store listing; English (U.K.) as the primary language.
- Relevant section: Mac App Store "Work" (focus and organisation apps).

## Drafts (founder approves, edits and posts)

### Draft 1: r/ProductivityApps update, Fri 30 Oct

Bracketed lines are for the founder to fill from the June comments. Claude cannot read Reddit, so it cannot quote them; tying changes to real comments is what makes this an update rather than a re-post. Delete any line that has no matching comment.

**Title:** I posted my 24-hour planner here in June and got 29 comments. Here's what I changed.

**Body:**

> I make SliceFocus, so I'm biased.
>
> In June I shared it here: a day planner where your whole day is one 24-hour clock, and a Pomodoro timer runs the blocks you draw on it. It got 29 comments. I went back through every one, and here is what's different four months later:
>
> **You can try it without an account.** Open it and you're on today's clock. If you sign in later (Apple, Google or email), everything you made as a guest comes with you.
> [June: "<quote of the comment that asked about sign-in>", if there was one]
>
> **The timer runs itself.** Work rolls into the break and back without a tap, on the lock screen too. It's the same timer on iPhone and Mac: start on one, see it on the other.
>
> **Focus Shield.** Pick the apps and sites that pull you away. They're blocked during work phases only and open again on every break. iPhone uses Screen Time; on the Mac a Safari extension blocks the sites and a shield window covers the apps.
>
> [June: "<comment>" → <what changed>]
>
> Smaller things: the timer collapses to a mini bar, a break no longer shows "Work" on the lock screen, and the create-slice dialog no longer breaks when the keyboard opens.
>
> What hasn't changed: it's free, with no ads and no subscription. It's Apple-only (iPhone, iPad, Mac) and the interface is in English.
>
> One question for anyone who tried it in June and stopped: what made you stop? And if you try it now, tell me where the first five minutes felt awkward. That's the part I'm working on.
>
> App Store: <campaign link, ct=reddit-pa-1030>

**Media:** `slicefocus-web/hero.mp4` (15 s, plan a block and watch it run itself), uploaded as Reddit video.

**Reply in the June thread** (after the new post is up):

> Update for anyone who commented here: it now starts without an account, the timer moves between work and breaks on its own, and it can block apps and sites during work phases. I wrote up what changed and which of your comments drove it: <link to the new post>

### Draft 2: r/macapps, Tue 3 Nov

Use the flair the sidebar asks for (self-promotion / free). If r/macapps limits developer posts or sends them to a megathread, post there or in r/MacOSApps instead.

**Title:** SliceFocus: a 24-hour clock planner for Mac with a Pomodoro timer that blocks distracting sites in Safari [free, I'm the developer]

**Body:**

> I'm the developer, so take this with that in mind.
>
> SliceFocus shows your day as one 24-hour clock. You draw time blocks on it, start one, and a Pomodoro timer runs it: work, break, work, without you tapping anything.
>
> The Mac parts I think this sub will care about:
>
> - **Website blocking in Safari** during work phases only. Breaks are free. A Safari web extension does it; there's no proxy, VPN or profile.
> - **App shield:** switch to an app on your list during a work phase and a shield window covers it.
> - **Menu bar:** the timer stays in the menu bar while Focus Shield is on, so it keeps working with the window closed.
> - **Same timer as your iPhone:** start a session on the phone and the Mac shields too (needs signing in on both; trying it on the Mac alone doesn't).
> - **Apple Calendar:** your events show next to your blocks, and blocks can be exported to a calendar.
>
> Honest notes: it's built in Flutter, with native Swift for the Safari extension, the shield and the menu bar. It's not Electron. It's free, with no ads and no subscription, and you can start without an account. There's also a Chrome extension for blocking, but it needs an account.
>
> I'd like to know what a Mac user expects from a focus blocker that this doesn't do.
>
> Mac App Store: <campaign link, ct=macapps-1103>

**Media:** gallery of `slicefocus-web/screen-mac.png`, `screen-menubar.png`, `focus-shield/blocked-page.png` and `focus-shield/mac-shield-window.png`, or the Mac hero clip if the sub allows video.

Before posting: check every claim against the live build (Focus Shield on a guest account on the Mac, the menu bar behaviour, the calendar export).
