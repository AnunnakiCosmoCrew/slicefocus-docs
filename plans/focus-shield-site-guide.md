# Focus Shield on slicefocus.app: what to explain, how, and where

**Status:** Sections 3.1 and 3.2 shipped 2026-10-23 ([slicefocus-web#29](https://github.com/AnunnakiCosmoCrew/slicefocus-web/pull/29), live at https://slicefocus.app/focus-shield/). Open: screenshots [slicefocus-web#27](https://github.com/AnunnakiCosmoCrew/slicefocus-web/issues/27), support FAQ and Chrome listing URL [slicefocus-web#28](https://github.com/AnunnakiCosmoCrew/slicefocus-web/issues/28), deep link [SliceFocusFE#624](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/624).
**Relates to:** ADR-057 (iOS Screen Time), ADR-059 (macOS Safari extension + app interruption), ADR-063 (Chrome extension), ADR-042 (marketing design language), SliceFocusFE#587 (in-app permission panel).

## 1. The problem

The site today gives Focus Shield one paragraph of "what" and one dense paragraph of "how" inside the features section of `index.html`. Everything a user needs to make it work on a given device is in that second paragraph, but nothing is actionable: no steps, no screenshots, no mention of the two settings that silently break it (Screen Time's *App & Website Activity* on iOS, *Always allow on every website* for the Safari extension), and no link to install the Chrome extension or to the in-app Settings path.

The app itself already knows what goes wrong (the `focusShieldPermission*` strings and the picker hint in `app_en.arb`). The site should say the same things once, in the same words, before the user hits them.

## 2. What the research says, and what it implies for us

### 2.1 Self-control tools: what users want from a blocker

- **Lyngs et al., CHI 2019, "Self-Control in Cyberspace"** reviewed 367 blocking apps and extensions through dual-systems theory: blocking works by removing the *cue* from System 1 so System 2 does not have to fight it in the moment. Users judge a blocker by whether it blocks *their* distractions, and by how it treats them when they try to get round it.
  Implication: the site must show that the block list is the user's own (presets are a starting point, not a verdict), and that breaks let everything back in. That is our answer to "will this fight me?".
- **Lyngs et al., IJHCS 2022, "The Goldilocks level of support"** (1,529 reviews of 334 tools): the top-rated tools give support that is "just right": enough to change behaviour, not so coercive that it triggers *psychological reactance* and abandonment. Tools combining a block with a reminder of the goal rated higher than pure blockers.
  Implication: lead with *why it is gentle*, not *how hard it blocks*. Our design already is Goldilocks by construction: work phase only, lifts at the break, the Mac "interrupts" rather than kills, the Chrome popup has a manual lift. Say so plainly. Also: the blocked page already names what you are working on (goal reminder). Mention it.
- **Lukoff, Lyngs & Alberts, CHI 2022 workshop, "Designing to support autonomy and reduce psychological reactance in digital self-control tools":** autonomy-supportive framing ("you chose this at the start of the session") beats controlling framing ("you are blocked").
  Implication: copy rule. Say "out of reach", "held back", "shielded"; never "banned", "locked out", "we block you". The app already enforces "interrupted, never blocked" on macOS; the site should follow the same rule.
- **Implementation intentions (Gollwitzer & Brandstätter 1997; Wieber, Gollwitzer et al. 2011 on attractive distractions):** already cited in our *Why it works* section. The Wieber paper is the relevant extension: for *attractive* distractions, a plain goal intention is not enough, an if-then plan is. Focus Shield is the if-then plan made mechanical: "if a work phase runs, then these are out of reach."
  Implication: one sentence connecting Focus Shield to the existing Gollwitzer bullet closes the loop between the science section and the feature.

### 2.2 Permission prompts: explain before the system asks

- **Tan et al., CHI 2014, "The effect of developer-specified explanations for permission requests"** (772 users): permission requests with an explanation are approved significantly more often; only ~19% of apps gave one.
- **Practitioner data (habitdoom, Family Controls in production):** authorization is the *midpoint*, not the end. ~35% of installs granted Screen Time access and then never picked a single app; that cohort retained far worse. Users also get stuck at the picker's two levels (tap a category to reveal its apps).
  Implication: the site's iOS walkthrough must run *through* the picker to "pick at least one thing", not stop at "tap Allow". And it must carry the two hints the app already shows: *App & Website Activity* must be on or the picker is empty (our own memory note from testing), and categories expand on tap.

### 2.3 How to write the instructions

- **Carroll's minimalism** (*The Nurnberg Funnel*, 1990; van der Meij & Carroll 1995): task-oriented chunks, start with the task, support error recognition and recovery, each chunk self-contained. Evidence: minimal manuals cut learning time roughly in half against a conventional manual.
  Implication: one card per device, each a short numbered procedure with a "If you don't see…" recovery line. No conceptual preamble inside the steps.
- **Mayer's multimedia principles** (segmenting, signaling, spatial contiguity, coherence): user-paced segments, visual cues on what matters, text beside the image it refers to, nothing decorative.
  Implication: one cropped screenshot per step where a system UI is involved (the Screen Time dialog, the picker, Safari's Extensions pane, the Chrome options page), with the relevant control ringed in the clay accent, caption directly under it. The menubar-shot on the current page already does exactly this (blur + ring); reuse that treatment.
- **Progressive disclosure (Nielsen; ADR-042's "one list, no accordion" rule):** the home page stays a showcase. Steps live on their own page; the feature beat links to it.

## 3. Proposed integration

### 3.1 Home page (`index.html`, the Focus Shield beat)

Keep the beat to the length of the other two. Replace the second paragraph with three short lines and a link:

> **How it behaves.** Only during work phases. Every break lets everything back in. On the Mac a listed app is interrupted with a note of what you are working on, not closed. You can lift the shield yourself at any time.
>
> **Where it runs.** iPhone and iPad through Screen Time (nothing to install). Mac: apps in the Mac app, websites in Safari through the bundled extension, or in Chrome through the SliceFocus Focus Shield extension.
>
> [Set up Focus Shield on your devices →](/focus-shield/)

Add one visual: a 3-up of the blocked states (iOS system shield, the Mac shield window, the Chrome/Safari blocked page). It shows the user the *outcome* before any setup, which is what the showcase is for.

In the *Why it works* list, add a fifth claim under the Gollwitzer one or as its continuation:

> For distractions that are genuinely tempting, a plain intention is not enough; a concrete if-then plan is. Focus Shield is that plan made automatic.
> Wieber, Gollwitzer et al., "If-then planning helps school-aged children ignore attractive distractions", 2011 (link to the Konstanz record or DOI).

Keep the existing "findings about attention in general, not studies of SliceFocus" note; it covers this too. Also add the Lyngs 2022 "Goldilocks" reference, phrased as a claim about blockers in general ("Blockers that support without coercing are the ones people keep using").

### 3.2 New page: `/focus-shield/index.html`

Same shell as the home page (same `styles.css`, cream column, serif headline). Structure, top to bottom:

1. **What it is, in 60 words.** The dual-systems sentence in plain language: the cue goes away, so you do not have to resist it. Work phase on, break off, your list.
2. **How to think about your list.** Presets (Social, News, Video, Shopping) as a start; prune to what actually pulls *you*. One line on why the list is per device (iOS selections are device-bound by Apple's design) so no one files that as a bug.
3. **Device cards** (one procedure each, Carroll style, each with a "Not working?" recovery line). Tabs or anchors, never an accordion that hides content from search.
   - **iPhone & iPad**
     1. Settings › Focus Shield › turn on *Block apps & websites*.
     2. When iOS asks for Screen Time access, tap *Allow*. (Screenshot: the system dialog.)
     3. Tap *Choose apps & websites*. Tap a category to open it and pick the apps inside; add websites at the bottom. Pick at least one. (Screenshot: the picker, with a category expanded.)
     4. Start a session. During work phases the chosen items show the Screen Time shield.
     *Not working?* The picker shows no apps while Settings › Screen Time › *App & Website Activity* is off. Turn it on, come back, tap *Try again*. (The same text the app shows.)
     *Running the iPhone app on a Mac?* Screen Time cannot be shared with it; use the Mac app instead.
   - **Mac: apps**
     1. Settings › Focus Shield › turn it on.
     2. *Add apps*: pick from the running and installed apps.
     3. Switching to a listed app during a work phase brings up the shield window with your current task and the time left; *Back to work* or *Take the break*.
     *Note:* SliceFocus stays in the menu bar when you close its window, so a session started on your iPhone still shields this Mac.
   - **Mac: websites in Safari**
     1. Settings › Focus Shield › *Safari extension* › *Enable in Safari*. (Screenshot: Safari › Settings › Extensions with SliceFocus ringed.)
     2. Tick the SliceFocus extension. When Safari asks for website access, choose *Always allow on every website*; with less than that, blocking cannot work.
     3. Back in SliceFocus, add websites under *Websites*. Subdomains are covered: `instagram.com` also covers `www.instagram.com`.
     *Not working?* The row in Settings says *Off* until the extension is on; it says *Couldn't check* if Safari is not installed. Rules can take a few seconds to apply after a phase starts.
   - **Chrome (any computer)**
     1. Install *SliceFocus Focus Shield* from the Chrome Web Store (link; "coming soon" badge until SliceFocusFE#593 ships, with the unpacked-install note hidden, that is a developer path).
     2. Click the toolbar icon, sign in with the same Google or email account you use in SliceFocus. Signed in with Apple only? Link a Google or email login in the app's Settings first.
     3. Open the options page and add your sites, or take a preset.
     4. Start a work phase in SliceFocus; the sites close until the phase ends. *Lift shield now* in the popup if you must; it returns with the next work phase.
     *Not working?* The extension needs an internet connection to see your session. A session started elsewhere can take up to a minute to reach Chrome.
4. **What Focus Shield cannot do.** Honest limits, in one list: Mac apps are interrupted, not closed; Safari and Chrome only, not every browser; lists do not sync between devices; on iOS a force-quit mid-session can leave the shield up until the planned end (ADR-057's accepted consequence) and how to clear it (open SliceFocus, or Screen Time settings). Honesty is part of the reactance budget: a user who finds the limit on their own trusts the rest less.
5. **Privacy.** Three lines: Apple's picker hands SliceFocus opaque tokens, so the app never learns which apps you chose; the Safari and Chrome extensions never read page content; the Chrome extension talks only to your SliceFocus account. Link the privacy policy.
6. **Support link** and the existing contact address.

### 3.3 Supporting changes

- **Screenshots:** eight crops, produced like the menubar shot (real capture, other content blurred, clay ring on the control). Source list: iOS Screen Time dialog, FamilyActivityPicker with a category open, the iOS shield, the Mac shield window, Safari Extensions pane, Safari website-access prompt, Chrome popup signed in, Chrome options page. Store them under `focus-shield/` in slicefocus-web.
- **Deep links:** none exist today. The iOS and Mac apps could register `slicefocus://settings/focus-shield` so the page's "Open Focus Shield settings" button lands in the right place; that is an FE issue, optional.
- **Copy source of truth:** the recovery lines must stay identical to the `focusShieldPermission*` strings in `app_en.arb`. Add a note at the top of the new page's HTML pointing at the arb keys, so a copy change in the app is mirrored.
- **Support page** (`slicefocus-docs/support.md`): add two FAQ entries that link to the new page ("Focus Shield isn't blocking anything", "I don't see my apps in the picker").
- **Chrome listing** (`slicefocus-chrome/store/listing.md`): Support URL becomes `https://slicefocus.app/focus-shield/`.
- **SEO:** the page is the only place the phrases "Screen Time app blocker Pomodoro", "Safari extension block websites during focus session" appear on the site; give it its own title and description, and an FAQ schema block for the recovery questions.

### 3.4 Copy rules for the page (from 2.1 and LUVITA.md)

- Autonomy framing: "out of reach", "held back", "shielded", "interrupted". Never "blocked you", "locked", "banned", "forbidden".
- Every sentence describes what ships today; nothing about Windows Chrome beyond "Chrome", nothing about speed.
- Steps use the exact labels from the app and the OS (italic), one action per step, no step without a visible result.
- Each card ends with recovery, not with praise.

## 4. Issues to open (suggested, all in slicefocus-web unless noted)

| # | Scope | Est. |
| --- | --- | --- |
| 1 | `/focus-shield/` page: structure, copy, limits, privacy; no screenshots yet | 3 |
| 2 | Home beat rewrite + link + the two *Why it works* additions | 1 |
| 3 | Eight screenshots, blur-and-ring treatment, captions | 3 |
| 4 | support.md FAQ entries + Chrome listing support URL (slicefocus-docs, slicefocus-chrome) | 1 |
| 5 | Optional, SliceFocusFE: `slicefocus://settings/focus-shield` deep link | 2 |

Founder check on 1 and 2: `public` (site copy).

## Sources

- Lyngs et al., "Self-Control in Cyberspace: Applying Dual Systems Theory to a Review of Digital Self-Control Tools", CHI 2019. https://arxiv.org/abs/1902.00157
- Lyngs et al., "The Goldilocks level of support: Using user reviews, ratings, and installation numbers to investigate digital self-control tools", IJHCS 166, 2022. https://ora.ox.ac.uk/objects/uuid:1b26d9f8-8a14-467a-97db-634e583036b5
- Lukoff, Lyngs & Alberts, "Designing to support autonomy and reduce psychological reactance in digital self-control tools", CHI 2022 workshop. https://ora.ox.ac.uk/objects/uuid:989e140b-383e-4c1a-a4bc-196bf99561a1
- Wieber, Gollwitzer et al., "If-then planning helps school-aged children ignore attractive distractions", 2011. https://www.socmot.uni-konstanz.de/publications/if-then-planning-helps-school-aged-children-ignore-attractive-distractions
- Tan et al., "The effect of developer-specified explanations for permission requests on smartphone user behavior", CHI 2014. https://blues.cs.berkeley.edu/?p=105
- Carroll, *The Nurnberg Funnel*, MIT Press 1990; overview: https://www.instructionaldesign.org/theories/minimalism/
- Mayer's multimedia principles, overview: https://twut.nd.edu/PDF/Principles-MultimediaLearning.pdf
- Practitioner: "FamilyControls in Production: The Edges" (35% picker drop-off). https://habitdoom.com/blog/shipping-familycontrols-ios
