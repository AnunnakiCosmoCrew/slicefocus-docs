# Step B drafts: pages for five moments of need

*Written: 2026-11-06 | Status: wording approved by the founder on 2026-11-06; the pages are being built as a pull request for slicefocus.app | Founder check: public (site copy) | Belongs to [name-strategy.md](name-strategy.md), step B*

## What these are

Four new pages for slicefocus.app, one for each moment when someone needs this kind of app. The fifth moment, keeping distractions away, already has its page: the Focus Shield guide at [slicefocus.app/focus-shield/](https://slicefocus.app/focus-shield/).

Each page answers one question plainly and shows how to do it in the app. They carry the signature: the name with "by CosmoCrew", the phrase "24-hour clock", and the site's closing line.

## How they were written

- **Every claim about the app was checked** against the app's code on 2026-11-06. One claim on the live website goes further than the app does and is left out; see "Settled" below.
- **The words follow how people search.** The App Store's own search suggestions (read 2026-11-06, United States) include "study planner", "study timer pomodoro", "focus timer for studying", "time blocking planner", "time blocking free", "pomodoro timer free", "daily schedule planner", "app blocker for productivity" and "safari website blocker". "Free" keeps appearing, so each page says the app is free near the top. The suggestions come from a public Apple address that is not documented, so they are a guide to wording and nothing more.
- **The pages are written for people.** Google's guidance on [helpful, people-first content](https://developers.google.com/search/docs/fundamentals/creating-helpful-content) asks for pages that answer the reader's question from first-hand knowledge. There are no tricks aimed at search engines or AI assistants.
- **No health claims.** The pages never say the app treats ADHD or anything else.

## Settled: what "runs itself" means

Settled with the founder on 2026-11-06. What runs by itself is the Pomodoro inside a session: once a session is started, it goes focus, break, focus, break, a long break after the fourth round, and then the next set, with no taps. The code agrees.

What does not happen by itself is the move from one planned block on the clock to the next. Each block's session is started by the user, and if a session is still running when its block's planned time is up, the app stretches that block.

These drafts say exactly that. The home page and the three comparison pages said more than that in eight places; they were reworded to the rounds on 2026-11-06 ([slicefocus-web#50](https://github.com/AnunnakiCosmoCrew/slicefocus-web/pull/50)), and the signature's description now reads "a Pomodoro timer that runs its rounds by itself".

## Page 1: plan a study day

- **Address:** slicefocus.app/study-day/
- **Page title:** Plan a study day on a 24-hour clock | SliceFocus
- **Picture:** the iPhone day view (`screen-day.png`)

**Headline:** Plan your study day on one clock.

See your whole study day on a 24-hour clock, block by block. When you start a block, a Pomodoro timer runs the work and break rounds for you. Free on iPhone and Mac.

**How to do it**

1. **Start from the Student Day template.** Tap Browse templates and choose Student Day. It lays out study from 09:00 to 12:00, lunch, study from 14:00 to 17:00, a break, and an evening review from 19:00 to 21:00.
2. **Make it yours.** Rename each Study block to its subject, and move or resize blocks to fit your lectures. To add a block, press and sweep along the clock.
3. **Start a block when its time comes.** The timer runs 25 minutes of work and a 5-minute break, and moves between them by itself. After four rounds the break is 15 minutes. You can change these lengths in Settings.
4. **Put the phone down.** The countdown stays on the iPhone lock screen. With Focus Shield on, the apps and sites you picked stay out of reach during work rounds and come back on every break.
5. **Look back in the evening.** Actual vs Planned shows which blocks you kept.

**Good to know**

- Your Apple Calendar events, such as lectures, show next to your blocks.
- You can start without an account. Signing in keeps your iPhone and Mac in step.
- The app is in English, for iPhone, iPad and Mac.

Free on iPhone and Mac. No ads, nothing to buy, no sign-up to start.

## Page 2: time-block a work day

- **Address:** slicefocus.app/time-blocking/
- **Page title:** Time blocking on a 24-hour clock | SliceFocus
- **Picture:** the Mac window with a planned day (`screen-mac.png`)

**Headline:** Block out your day on a clock you can see.

Time blocking means giving every part of the day a job before the day starts. SliceFocus does it on a 24-hour clock, so free time and crowded hours show at a glance. Free on iPhone and Mac.

**How to do it**

1. **Draw your blocks.** Press and sweep along the clock to make a block. Times snap to five minutes. Or start from a template: Maker Schedule, 9-to-5 + Gym or Deep Work Sprint.
2. **See your meetings.** Apple Calendar events show round the clock next to your blocks. You can also send your blocks to a calendar you choose.
3. **Fill the gaps.** Tap an empty stretch and the app suggests something that fits the time.
4. **Colour by kind of work.** Give blocks a category, such as Deep Work or Meetings. The categories are yours to name.
5. **Run the day.** Start a block and its timer runs the work and break rounds. The time left shows in the Mac menu bar and on the iPhone lock screen.
6. **Check the plan against the day.** Actual vs Planned lays the focus time you logged over the blocks you planned.

**Good to know**

- Plans change. Edit a block's times and its neighbours adjust.
- A day you like can be saved as your own template.
- Each day has its own plan.

Free on iPhone and Mac. No ads, nothing to buy, no sign-up to start.

## Page 3: a Pomodoro timer that runs itself

- **Address:** slicefocus.app/pomodoro/
- **Page title:** A Pomodoro timer that runs itself | SliceFocus
- **Picture:** the focus timer (`screen-focus.png`), and the menu bar (`screen-menubar.png`)

**Headline:** Start once. The timer runs the rest.

A timer that waits for a tap at the end of every round breaks the rhythm it is meant to keep. SliceFocus moves from work to break and back by itself, on every device you use. Free on iPhone and Mac.

**What it does**

- **Runs the rounds for you.** 25 minutes of work, a 5-minute break, and a 15-minute break after four rounds. You can change every length in Settings, and turn the automatic move off if you want to tap.
- **Stays in sight.** The countdown shows on the iPhone lock screen and in the Mac menu bar, with no window open.
- **One timer on every device.** Signed in on your iPhone and your Mac, you start on one and see it on the other. Pause on one, and the other follows.
- **Tells you once.** One notification when each round ends.
- **Sound if you want it.** White, brown or pink noise that pauses with the timer.
- **Works with or without a plan.** Start a timer on a block from your day, or tap Start Focus Session for one on its own. If you are still working when a block's planned time is up, the timer keeps going and the block stretches to match.

**Good to know**

- A Mac window that is open but not in front can take up to a minute to show a change. The lock screen and the menu bar follow within seconds.
- You can start without an account. One timer across devices needs signing in on each.

Free on iPhone and Mac. No ads, nothing to buy, no sign-up to start.

## Page 4: keep distractions away (already live)

The Focus Shield guide at [slicefocus.app/focus-shield/](https://slicefocus.app/focus-shield/) is this page. No new page is proposed. The four new pages link to it.

## Page 5: see where the day went

- **Address:** slicefocus.app/planned-vs-actual/
- **Page title:** Planned vs actual: see where your day went | SliceFocus
- **Picture:** the Actual vs Planned screen (App Store screenshot 4)

**Headline:** See where your day actually went.

A plan shows what you meant to do. SliceFocus also keeps what you did, and lays one over the other. Free on iPhone and Mac.

**What you can look at**

- **Actual vs Planned.** The focus time you logged, laid over the blocks you planned, day by day.
- **Session history.** Every session you finished.
- **Trends.** Focus minutes per day, and by week and month, split by category.
- **A weekly recap.** An optional summary of your focus week every Monday.

**Good to know**

- "Actual" means time you spent with the timer running. The app does not watch what you do in the background, so time without the timer is not counted.
- Calendar events you imported are not counted as planned blocks here.
- If you sign in, your history follows you to your other devices.

Free on iPhone and Mac. No ads, nothing to buy, no sign-up to start.

## What building them takes

- Each page uses the look of the existing Focus Shield guide: the same header with "by CosmoCrew", the same footer, one picture and the App Store button.
- The home page's footer gains links to the four pages, and each page links to the others and to the Focus Shield guide.
- Each page's App Store button carries its own label, so installs from each page can be counted.
- One pull request, opened only after the wording is approved.
