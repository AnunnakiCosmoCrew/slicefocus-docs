# Feature videos for slicefocus.app (1.7.1 feature set)

*Created: 2026-10-22 | Owner: founder | Implements: site videos for the 1.7.1 features (Focus Shield on iPhone and Mac, auto-advance, one timer across devices) and the hero loop that [SliceFocus#311](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/311) specified but never shipped (the hero still shows the poster placeholder).*

Governed by [ADR-042](../adr/042-minimal-sincere-marketing-design-language.md): the real product doing one real thing, silent, no burned-in text, no bezels, no effects.

## 1. What the evidence says, and what we take from it

Academic (learning from video) and industry (marketing video, web performance, accessibility) sources agree on more than they disagree. The rules below are the ones that survived both.

| Rule | Backing | Applied here |
| --- | --- | --- |
| **One idea per clip, short.** Engagement is highest under 1 minute and falls with length; the largest video-engagement study (6.9 M MOOC sessions) found drop-off after a few minutes and best engagement in the shortest segments. Mayer's *segmenting* principle: user-paced chunks beat one long walkthrough. | [Guo, Kim & Rubin 2014](https://www.cs.rochester.edu/hci/pubs/pdfs/edX-MOOC-video-production-and-engagement_LAS-2014.pdf); [Wistia State of Video](https://wistia.com/blog/optimal-video-length) (videos under 1 min: ~50 % engagement, highest of all bands); [Mayer's principles](https://edtechbooks.org/designing_transmedia_learning_environments/principles_of_multimedia_design) | One clip per feature, **8–15 s**, looping. No "tour" video. |
| **Cut everything that is not the point** (Mayer's *coherence*): decorative text, extra UI, cursor theatre. | [Adobe on Mayer's 12 principles](https://elearning.adobe.com/2019/12/12-principles-of-multimedia-learning-relevant-to-video-from-richard-e-mayers-work) | Fresh demo data, one screen, the single interaction that is the feature. Nothing else moves. |
| **Point at the essential thing** (Mayer's *signaling*) without adding noise. | same | Signalling comes from framing and the moment the clip starts on, not from overlays. The clip opens ≤1 s before the change (the handoff, the block page, the shield window). |
| **Design for sound off.** Marketing clips are watched muted; the motion must carry the meaning. | [Wistia](https://wistia.com/blog/optimal-video-length); [Swarmify product-video practice](https://swarmify.com/blog/product-video-best-practices/) | Silent files, no audio track at all (smaller, and no autoplay-with-sound risk). |
| **Show the product, not a trailer.** Apple's App Preview rules (15–30 s, in-app footage only, no people or marketing intros) are the strictest published spec for app video and match ADR-042. | [App Store Connect preview specs](https://www.developer.apple.com/help/app-store-connect/reference/app-information/app-preview-specifications); [AppRadar](https://appradar.com/academy/preview-video) | Same captures serve the site and, cut to 15–30 s, an App Preview later. |
| **Don't autoplay without consent, and if you do, keep it silent, in view, and stoppable.** NN/g: users dislike video that starts on its own; WCAG 2.2.2 requires a way to pause anything that moves for more than 5 s; the a11y project and Section 508: when in doubt, don't. web.dev: autoplay only as a GIF replacement. | [a11yproject](https://www.a11yproject.com/posts/never-use-auto-play/); [Section508](https://www.section508.gov/blog/avoid-auto-playing-content/); [web.dev lazy-loading video](https://web.dev/articles/lazy-loading-video) | Hero loop is the **only** autoplay (muted, silent file, pause button, stops when `prefers-reduced-motion`, `saveData`, or out of view). Feature clips play when the visitor opens the feature, and pause when it closes. |
| **Performance: the poster is the LCP, the video never is.** Poster image, `preload="none"`, lazy loading, a small file. | [web.dev video performance](https://web.dev/learn/performance/video-performance) | Posters as WebP/JPEG, hero poster `fetchpriority="high"`; each clip ≤1.5 MB MP4 and ≤1 MB WebM. |
| **Text alternative for video-only media** (WCAG 1.2.1) and no flashing (2.3.1). | WCAG 2.2 | Every clip has a one-line `<figcaption>` saying what happens, in HTML, not burned in. That also satisfies "no captions" in ADR-042. |

Numbers to ignore: "video lifts conversions 80 %", "65 % leave in 10 s", "1.7-second hook". They circulate without a primary study ([summary of the vendor claims](https://funnl.ai/why-product-videos-arent-converting-and-how-to-fix/)); the plan does not depend on them.

## 2. The clips

Same demo account and seed as the App Store screenshots (`tool/screenshots/seed.py`, dev backend, founder's Thursday, five categories, 14 days of history). Pomodoro set to **1-minute work / 1-minute break** in the demo account so a phase boundary happens inside a clip at real speed; no time-lapse.

| # | Clip | Replaces | Platform / capture | Storyboard (silent loop) | Length |
| --- | --- | --- | --- | --- | --- |
| H | **Hero: it runs itself** | the `hero-poster` placeholder (#311 never shipped) | Mac window, 1440×900 debug build | #311 storyboard: block draining → auto-advance handoff → next block drains → second handoff → hold on the full day → loop. Two 1-min phases give two boundaries in 2 min; trim to 15 s around them with one jump cut. | 15 s |
| 1 | Plan on the radial canvas | beat1 (keep if still accurate) | iPhone simulator | press-and-sweep a block, name it, release | 8–10 s |
| 2 | Pomodoro with auto-advance | beat2 | iPhone simulator | timer at 0:04 → 0:00 → BREAK appears without a tap → break counts | 10 s |
| 3 | **One timer, all devices** | new | Mac window + iPhone simulator recorded at once, composed side by side | tap Start on the phone → the Mac timer appears within a second; phone locks → both keep counting | 12 s |
| 4 | **Focus Shield on the Mac: Safari** | new | founder's Mac, store/TestFlight build (dev-signed extension is not registered with Safari) | session running → open a listed site in Safari → the SliceFocus block page → break starts → page loads | 12 s |
| 5 | **Focus Shield on the Mac: apps** | new | founder's Mac | switch to a listed app → shield window comes to the front → back to work | 8 s |
| 6 | **Focus Shield in Chrome** | new, ships when the Web Store listing is live | founder's Mac, store build of the extension | listed site in Chrome → blocked page → break → loads | 10 s |
| 7 | Focus Shield on iPhone | new | **real iPhone** (the Screen Time picker does not run in the simulator), QuickTime capture | Settings → Focus Shield on → picker → choose an app → session → app shows the Screen Time shield | 15 s |
| 8 | Lock screen timer | beat3 (keep) | – | – | – |
| 9 | Planned vs actual, Trends | beat5, beat6 (keep; re-record only if the seed data looks better) | iPhone simulator | – | 8 s |

Clips 1–3 and 9 come from the scripted pipeline (section 3). Clips 4–7 need the founder's own Mac and iPhone; the pipeline supplies the demo account, the recording script and the encoder, the founder presses record.

## 3. Pipeline: `tool/videos/` in SliceFocusFE

Extends `tool/screenshots/` rather than duplicating it.

- `videos_test.dart`: an integration test like `screenshots_test.dart` that prints `CLIP:<name>:start` and `CLIP:<name>:end` around each scene and performs the interactions (drag-to-create, start session, wait for a boundary). It reuses the same fakes (UTC, EventKit stub, notification stub).
- `run.sh <macos|ios> <out>`: seeds (or reuses) the demo account, starts one recording per platform, runs the test, and cuts clips from the timestamps:
  - macOS: `screencapture -v -R <x,y,w,h>` on the debug window rectangle (the window is placed at a fixed origin by `SF_SCREENSHOT_WINDOW`; add `SF_SCREENSHOT_ORIGIN`), no cursor (`-C` off).
  - iOS: `xcrun simctl io <udid> recordVideo --codec h264 <file>` (status bar overridden to 9:41 as in screenshots).
  - Clip 3: both recorders at once, then `ffmpeg hstack` on a brand-cream canvas.
- `encode.sh <raw.mov> <name>`: trim by the logged timestamps, scale (phone 720×1560, Mac 1440×900), 30 fps, strip audio, `libx264 -crf 23 -profile high -pix_fmt yuv420p -movflags +faststart` and `libvpx-vp9 -crf 33 -b:v 0`, loop-trim so the last frame matches the first, poster from the frame just before the change (`-ss`), WebP + JPEG. Fails if a file is over budget.
- `README.md`: one-command rerun; the founder-recorded clips go through `encode.sh` only.

Prerequisites on the Mac: `brew install ffmpeg` (the `ffmpeg`/`ffprobe` in `/usr/local/bin` are x86_64 leftovers and fail with "bad CPU type"); Screen Recording permission for the terminal; the demo credentials in the environment.

## 4. Site integration: slicefocus-web

- **Hero**: `<video autoplay muted loop playsinline preload="metadata" poster="hero-poster.webp">` with `<source webm>` then `<source mp4>`, a visible pause/play button (WCAG 2.2.2), `fetchpriority="high"` on the poster. A few lines of JS: do not autoplay (show the poster and the button) when `matchMedia('(prefers-reduced-motion: reduce)')` or `navigator.connection?.saveData`; pause when the hero leaves the viewport (IntersectionObserver), resume on return.
- **Feature clips**: keep the `<details>` accordion. On `toggle` open: `video.play()` (muted loop); on close: pause and reset. `controls` stay for keyboard users; `preload="none"`, `loading="lazy"`, WebM + MP4 sources, WebP poster with JPEG fallback, `<figcaption>` describing the clip in one line. Clip 6 waits for the Web Store listing.
- **Reduced motion**: the existing `@media (prefers-reduced-motion: reduce)` block also disables the autoplay class; videos still play on an explicit click.
- **Budget**: the page already ships ~15 MB of MP4 at `preload="none"`, so nothing loads until opened; keep it that way, and add no third-party player.

## 5. Sessions and order

| Step | Repo | Who | Model | Depends on |
| --- | --- | --- | --- | --- |
| A. `tool/videos/` pipeline, encoder, README; produce clips H, 1, 2, 3, 9 from the dev demo account | SliceFocusFE | agent session | Sonnet 5.5 · medium | demo credentials in env; `brew install ffmpeg` |
| B. Site markup, JS and CSS for hero autoplay + accordion playback, reduced motion, posters; wire the clips as they land | slicefocus-web | agent session | Sonnet 5.5 · medium | none (uses the current beats until A delivers) |
| C. Record clips 4, 5, 6, 7 on the founder's Mac and iPhone; run `encode.sh` | – | founder, with the A session's script | – | A merged; store build installed; Chrome listing live for 6 |
| D. Review the clips against ADR-042 and the storyboard, replace the posters, publish | slicefocus-web | founder + agent | Sonnet 5.5 · low | A, B, C |

A and B run in parallel (different repos). C is founder time, about an hour with the script. D is a short PR.

## 6. Decisions for the founder

1. Hero autoplay: ADR-042 says yes (silent loop, GIF replacement). Confirmed by this plan unless you say otherwise.
2. One-line HTML `<figcaption>` under each clip: yes (accessibility text alternative, not burned-in text).
3. Clip 7 needs your iPhone and a QuickTime recording; the alternative is to leave iPhone Focus Shield as a screenshot. Default: record it.
