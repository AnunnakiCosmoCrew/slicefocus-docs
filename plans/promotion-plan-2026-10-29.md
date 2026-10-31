# Promotion plan for 1.7.4: first weeks

*Created: 2026-10-29 | Release: [SliceFocusFE#641](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/641) | Founder check: public (every post below is a draft; the founder approves and posts it, Claude never does)*

## Where this starts

- 1.7.4 is live on iPhone and iPad (released 2026-10-29, 05:19 UTC) and on the Mac (approved by 2026-10-31, founder's report). A first launch opens on a guest account, so people can try the app without signing up.
- Founder's rule was to promote only once the Mac version is out. That is met, so the plan starts **Mon 2 Nov** (re-dated on 2026-10-31; the original Friday start passed while the Mac version was in review).
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
| 7 | **Show HN**, framed as how it is built (radial CustomPainter, server-owned timer shared by iPhone, Mac and lock screen) | A write-up and 3 h of replies | Gated: see "Show HN gate" below. Earliest the week of 23 Nov |

## Day 0, today (Thu 29 Oct): before anything is posted

All founder steps.

1. ~~**Run the guest test on a real iPhone**~~ Done by the founder on TestFlight build 45, the same build that went live.
2. ~~**Register `account_state` in GA4**~~ Done 2026-10-29: user-scoped custom dimension "Account state" on the slicefocus-b9034 property. GA4 does not fill it backwards, so the guest vs signed-in split starts from that day.
3. ~~**Make App Store campaign links**~~ Done 2026-10-29. Provider token `128684421`; one link per channel, the same App Store page with a label:
   - r/ProductivityApps: https://apps.apple.com/app/apple-store/id6761280632?pt=128684421&ct=reddit-pa-1102&mt=8
   - r/macapps: https://apps.apple.com/app/apple-store/id6761280632?pt=128684421&ct=macapps-1104&mt=8
   - r/pomodoro: https://apps.apple.com/app/apple-store/id6761280632?pt=128684421&ct=pomodoro-1110&mt=8
   - slicefocus.app download buttons: https://apps.apple.com/app/apple-store/id6761280632?pt=128684421&ct=site&mt=8 (goes in with the site change, week of 2 Nov)
   App Store Connect lists a campaign only once 5 different Apple Accounts have installed through it, and counts only people who share analytics with developers.
4. ~~**Read the sidebars**~~ Done 2026-10-29 (founder's screenshots). What they mean for us:
   - **r/ProductivityApps:** self-promotion once a month with at least 10 community karma in the sub; flair **Self Promotion** (a wrong flair is removed; "Feedback Wanted" posts judged low quality are removed); no asking for testers (that goes to r/betatesters), no surveys, no "vibe coded" apps. The June post was the last promotion, so Monday's post is allowed if the June post's karma still counts towards the 10.
   - **r/macapps:** 10 local karma before posting or promoting in comments; one app post per developer per 30 days; a main-feed post only for developers who qualify under the sub's "Trust vs. Transparency" rules and use its template, everyone else promotes in the monthly megathread ("The App Pile"); flair **Free** (fits: no paid tier); Mac App Store links only, no shortened or redirect links; disclose in every comment. So draft 2 becomes a megathread comment, after earning the karma.
   - **r/betatesters:** not needed now that the app is public. Keep it for testing a future beta.
5. ~~**Featuring nomination**~~ Submitted 2026-10-29 13:43 Türkiye time, on the founder's go: "SliceFocus 1.7.4: start without an account", App Enhancements, iPhone, iPad and macOS. Apple writes back only if it is picked. Nominate the next release at least 3 weeks ahead (Apple's own advice in the form).

## First week (re-dated 2026-10-31)

Times are Türkiye time. US Eastern is off summer time from Sun 1 Nov, so 8:00 ET is 16:00.

| Day | What | Who |
| --- | --- | --- |
| Sat 31 Oct – Mon 2 Nov | **Earn 10 karma in r/MacApps**: answer 3–5 Mac questions there with plain help, no mention of SliceFocus | Founder |
| Mon 2 Nov, 16:00 | **Reddit update post**, r/ProductivityApps (draft 1) with the 15 s hero clip; then the reply in the June thread. Stay for 2–3 h of comments | Founder |
| Tue 3 Nov | Answer every comment within a few hours. Note each reason someone gives for not trying or for dropping the app | Founder |
| Wed 4 Nov, 16:00 | **r/macapps November megathread comment** (draft 2), once the 10 karma is there; otherwise the day it is | Founder |
| Wed 4 Nov | Site copy: "free, no account needed to start" and the `ct=site` link on slicefocus.app, as a `slicefocus-web` issue for an agent; founder approves the wording | Founder approves |
| Thu 5 Nov | **First readout**: installs from the Reddit link, guest vs signed-in activation. Start the 2–3 disclosed replies a week in study and ADHD threads | Founder; Claude reads the dashboard |
| Mon 9 Nov | **Week-1 readout**; decide the r/pomodoro post and whether a guest-start clip re-take is worth it | Founder + Claude |

## After week 1

- **Tue 10 Nov:** r/pomodoro post (`ct=pomodoro-1110`), a disclosed maker's post on auto-advance and blocking only during work phases. Session comparison page live. Optional re-take of the hero clip showing guest start (`SliceFocusFE/tool/videos/run.sh`, needs the Mac unlocked).
- **Every Monday:** readout row added to the log below.
- **Next release:** nominate it for featuring at least 3 weeks before it ships (Apple's advice in the form).
- **Show HN gate (earliest week of 23 Nov):** at least 100 installs in the 1.7.4 cohorts, the lower end of the 95 % interval for "installed → completed a session" above the pre-1.7.4 rate, and a weekday when the founder can reply for 3 h. If the gate is not met, find where guests drop off and fix that first.

## Early numbers, before any promotion (read 2026-10-31)

- Week of 26 Oct (1.7.4 went live on the Thursday): 10 installs, 4 started a session, 3 completed one. Far too few to judge; it only shows the events arrive.
- The guest vs signed-in split is still empty. GA4 takes a day or two to fill a newly registered dimension; check again on Mon 2 Nov before the post.

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

**Flair:** Self Promotion. Before posting, check the 10 community karma (if the June post's karma doesn't count, comment helpfully there first). If someone asks whether it is "vibe coded", answer plainly how it is built and tested; the sub removes vibe-coded apps.

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
> App Store: https://apps.apple.com/app/apple-store/id6761280632?pt=128684421&ct=reddit-pa-1102&mt=8

**Media:** `slicefocus-web/hero.mp4` (15 s, plan a block and watch it run itself), uploaded as Reddit video.

**Reply in the June thread** (after the new post is up):

> Update for anyone who commented here: it now starts without an account, the timer moves between work and breaks on its own, and it can block apps and sites during work phases. I wrote up what changed and which of your comments drove it: <link to the new post>

### Draft 2: r/macapps November megathread, Tue 3 Nov

A comment in "The App Pile – November, 2026", posted once the account has 10 r/MacApps karma. It carries what the sub's transparency rules look for: who made it, the price, how it is built, what data it collects.

> **SliceFocus** – a 24-hour clock day planner with a Pomodoro timer that blocks distracting sites in Safari during work phases. Free. I'm the developer.
>
> - **Website blocking in Safari**, work phases only; breaks are free. A Safari web extension does it, with no proxy, VPN or profile.
> - **App shield:** switch to an app on your list during a work phase and a shield window covers it.
> - **Menu bar:** the timer stays in the menu bar while Focus Shield is on, so it keeps working with the window closed.
> - **Same timer as your iPhone:** start on the phone and the Mac shields too (needs signing in on both).
> - **Apple Calendar:** your events show next to your blocks; blocks can be exported to a calendar.
>
> Built in Flutter, with native Swift for the Safari extension, the shield and the menu bar; not Electron. Free, no ads, no subscription, no paid tier, and you can start without an account. Analytics are Firebase, with no tracking across apps; privacy policy: https://anunnakicosmocrew.github.io/slicefocus-docs/legal/privacy-policy
>
> Mac App Store: https://apps.apple.com/app/apple-store/id6761280632?pt=128684421&ct=macapps-1104&mt=8

**Main-feed version (only if the founder qualifies under the sub's transparency rules):** use the sub's own post template, flair **Free**, and fill it from the longer text below.

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
> Mac App Store: https://apps.apple.com/app/apple-store/id6761280632?pt=128684421&ct=macapps-1104&mt=8

**Media:** gallery of `slicefocus-web/screen-mac.png`, `screen-menubar.png`, `focus-shield/blocked-page.png` and `focus-shield/mac-shield-window.png`, or the Mac hero clip if the sub allows video.

Before posting: check every claim against the live build (Focus Shield on a guest account on the Mac, the menu bar behaviour, the calendar export).
