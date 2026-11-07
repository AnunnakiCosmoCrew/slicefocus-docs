# Hero video for slicefocus.app: brief, storyboard, approach

*Created: 2026-10-23 | Owner: founder | Issue: [SliceFocusFE#622](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/622) | Supersedes the hero row of [site-feature-videos.md](site-feature-videos.md) (that take, a static capture in which only the timer panel changed, was rejected: a still image does the same).*

Design rules: [ADR-042](../adr/042-minimal-sincere-marketing-design-language.md), amended on 2026-10-23 to allow time compression and restrained, eased framing moves for the hero.

## 1. The founder's brief (verbatim)

Task: Design and Implement a SaaS Hero Video for SliceFocus
You are a senior product designer, motion designer, and frontend engineer specializing in premium SaaS landing pages. Your task is to create and integrate a short, silent, automatically looping product demonstration for SliceFocus, a productivity application developed by Luvita Teknoloji. The result should feel comparable in quality to product demonstrations found on websites such as Screen Studio, Framer, Linear, Cursor, and other premium software products.

1. Product Context. Product: SliceFocus. Website: https://slicefocus.app. Platforms: iOS and macOS. SliceFocus is a productivity and time-management application built around planning your day and automatically executing focus and break sessions. Its core value proposition is: Plan once. Focus throughout your day. Key capabilities include: plan daily activities using a visual, 24-hour schedule; view activities in a pie-chart-style timeline or list; automatically transition between focus sessions, breaks, and subsequent activities; continue focus cycles even when the iPhone is locked; track ongoing sessions using iOS Live Activities; synchronize productivity workflows between iPhone and Mac; start an immediate focus session without configuring an entire day; review productivity analytics. Important: these features are product context, not a requirement to showcase all of them in the video. The hero video must communicate the essential product experience immediately.

2. First: Inspect the Existing Project. Before designing or implementing anything: explore the website repository, its architecture, and existing components; review the current homepage, hero section, typography, colors, layout, and visual identity; locate any existing product screenshots, application recordings, promotional assets, or design resources; inspect available application source code or documentation if accessible; identify the most recognizable and visually distinctive SliceFocus interaction; understand existing build and deployment conventions. Do not invent UI screens or functionality that contradicts the real application. Reuse actual product visuals whenever possible. Do not redesign the entire website.

3. Objective. Create a premium-quality, silent product hero demonstration that communicates what SliceFocus does to a first-time visitor within approximately five seconds. The video should answer three questions visually: What is SliceFocus? How does it work? Why would I want to use it? It must be understandable without sound, captions, or prior knowledge of the product. The result should resemble a polished software product demonstration, not a traditional advertisement or tutorial.

4. Proposed Narrative. Use the following as a starting point, but improve it if you discover a more compelling workflow. Scene 1, Plan Your Day (0–4 s): show the actual SliceFocus interface; a user creates or selects a focus activity; the daily schedule updates to reflect the planned session; prefer showing the distinctive visual scheduling interface if it is available. Scene 2, Start Focusing (4–8 s): the user starts the planned activity; the application transitions into its active focus state; a focus timer becomes visible; keep movements smooth and purposeful. Scene 3, Automatic Progression (8–12 s): demonstrate SliceFocus automatically progressing from a focus session into a break or the next scheduled activity; use accelerated time or an elegant transition rather than making the visitor watch a timer count down; this should communicate the primary differentiator: the user does not need to manually manage every Pomodoro transition. Scene 4, Continue Anywhere (12–16 s, optional): if visual assets and space permit, briefly show the iPhone Live Activity or Mac companion interface; only include this scene if it improves understanding without overwhelming the viewer. Scene 5, Loop: return seamlessly to the initial state; the transition should feel natural rather than like an abrupt video restart.

5. Motion Design Principles. Total duration ideally 12–18 seconds; silent playback with no audio track; autoplay and infinite looping; demonstrate one coherent workflow; start with the application interface, not an introductory logo animation; avoid generic AI-generated backgrounds, floating icons, particle effects, and unnecessary motion graphics; use real application UI wherever possible; prefer smooth cursor or touch interactions; use subtle easing and restrained transitions; maintain readable UI text at normal desktop viewing sizes; avoid excessive typing animations; avoid long loading states; do not show artificial interactions that misrepresent the product; make the final result feel clean, confident, modern, and credible. The animation should support the product's visual identity rather than compete with it.

6. Visual Direction. Aim for a visual standard similar to https://screen.studio, https://www.framer.com, https://linear.app, https://cursor.com. Study their use of application framing, timing, transitions, typography, composition, and presentation. Do not copy their visual identities. SliceFocus should remain recognizable as its own product. The hero should emphasize the distinctive scheduling interface and the effortless transition between focus and break sessions. Avoid turning the hero into a collection of generic productivity illustrations.

7. Implementation Approach. Select the most appropriate technical implementation after inspecting the repository. Possible approaches: a real screen recording, carefully edited and optimized; a deterministic animation using existing product UI assets; a programmatically rendered video using a suitable motion framework such as Remotion; a lightweight HTML/CSS/SVG animation based on the actual application interface. Do not automatically add a heavy animation framework. Choose the simplest maintainable solution that delivers high visual quality. If the project is based on Astro, preserve its architecture and avoid introducing unnecessary client-side JavaScript. Prefer an actual optimized video asset for a self-contained demonstration. Use a programmatic animation when it meaningfully improves quality, maintainability, or visual accuracy. Do not assume you can obtain real screen recordings unless source assets, simulators, or recording tools are available.

8. Website Integration. Integrate the result directly into the existing SliceFocus homepage hero. Requirements: responsive desktop and mobile presentation; muted autoplay; seamless looping; inline playback on iOS; no visible player controls in normal presentation; a high-quality poster image; minimal layout shift; reasonable loading performance; support for prefers-reduced-motion; graceful fallback if the video cannot play.

## 2. Storyboard (one workflow, about 16 s, silent, seamless loop)

The real macOS app, signed in to the demo account on the dev backend (the App Store screenshot seed: a founder's weekday, five categories, 14 days of history). The day is full except for an open gap at the current time, where the story starts. Pomodoro on Classic (25 min work, 5 min break): the viewer should recognise "25:00".

| Time | Scene | On screen | Camera |
| --- | --- | --- | --- |
| 0.0–1.0 | hold | Today's plan on the 24-hour canvas, the clock hand inside the open gap, the block list below. | Top framing: date, canvas, the first rows. Still. |
| 1.0–3.2 | **plan** | Press on the ring at the gap's start, sweep clockwise 90 minutes; the selection wedge grows under the pointer. | Still. |
| 3.2–5.4 | plan | The Create Slice dialog: "Deep work" typed in under a second, the Deep Work category chip, Save. The dialog closes and the wedge sits on the ring in its colour, the row in the list. | Still. |
| 5.4–7.4 | **start** | The new row's play control. The timer panel opens below the list: the block's name, 25:00, WORK, Cycle 1/4, and the digits start counting. | Eased pan down with a push-in to about 1.12x, so the ring's live wedge, its row and the timer panel share the frame. |
| 7.4–10.4 | **automatic progression** | The real countdown compressed: 24:58 to 00:04 in about 2.5 s, the digits visibly racing (eased in and out, no hard speed cut), then real time from 00:03. | Hold. |
| 10.4–12.4 | handoff | 00:03, 00:02, 00:01, 00:00, and SHORT BREAK 05:00 takes over by itself; the break counts for 1.5 s. Nobody clicked. | Hold. |
| 12.4–13.6 | hint | The break compressed to 00:03, then 00:00 and WORK 25:00, Cycle 2/4: the next handoff is just as automatic. | Hold, then ease back towards 1.0x. |
| 13.6–15.0 | rest | The timer counting, the plan behind it. | Ease out to the opening framing. |
| 15.0–15.8 | **loop** | Cross-dissolve to the first frame (the open gap, no timer), so the restart is a soft reset, not a cut. | Opening framing. |

Scene 4 (the iPhone) is left out: the existing phone-and-Mac take has the Mac visibly lagging, and a phone next to the Mac in the last three seconds would compete with the handoff rather than add to it. "One app for iPhone and Mac" is the line under the hero and the devices clip in the Features section.

Rules kept from ADR-042: the real UI only, silent, no captions or burned-in text, no bezels, no particles, no cursor theatre (the gestures are the app's own touch events; no pointer is painted), no hype. Added for the hero (ADR-042, 2026-10-23 amendment): time compression of real footage and eased framing moves of at most 1.15x.

## 3. Technical approach

**A real screen recording, composed with ffmpeg only.** The product's own pixels are the whole point (ADR-042), the capture pipeline already exists (`tool/videos/` in SliceFocusFE, from SliceFocusFE#614), and the framing moves the brief asks for are within ffmpeg's reach: `zoompan` with eased expressions for the push-in and pan, `setpts` on cut segments for the speed ramps, `xfade` for the loop. A Remotion or HTML render of screenshots was the fallback if the moves could not be made smooth; it was not needed.

- **Capture.** `tool/videos/videos_test.dart`, scene `hero` (markers `hero:start`, `hero:dialog`, `hero:saved`, `hero:started`, `hero:b1`, `hero:b2`, `hero:end`), driven by `flutter drive` against the dev backend. `seed.py --open-now` clears a 2-hour gap from the current quarter hour so the block is drawn over the current time and can be started as is (no ADR-034 start adjustment on screen). The window is 1100x860 points on the built-in Retina display (2200x1720 px recorded with `screencapture -v`), title bar cropped; a 1440-wide output from a 1100-point capture puts the app's 700-point column at 64 % of the frame, so the UI text reads at the site's desktop hero width.
- **Compose.** `tool/videos/hero/compose.sh <raw.mov> <markers> <out-dir>`: cuts the segments at the marker times, applies the speed ramps (stepped 1x, 4x, 16x, 64x, 256x and back, each step a few frames, which reads as an eased ramp at 30 fps), runs the camera as one `zoompan` with eased keyframes, and closes the loop with a short `xfade`. Output: `hero.mp4` (H.264, 1440x900, 30 fps, `yuv420p`, `+faststart`, no audio, at most 2 MB), `hero.webm` (VP9, at most 1.5 MB), `hero-poster.jpg` and `.webp` from the frame where the plan is visible and the timer has just started.
- **Site.** The video carries only app pixels; the page frames it: at the desktop breakpoint `.hero-media` keeps rounded corners and the soft `--shadow` round the video (as it already does round the phone screenshot), on `--bg`. That way the cream, and the dark theme's near-black, come from the page's own tokens instead of being baked into the file. The autoplay, pause button, reduced-motion and Save-Data behaviour from slicefocus-web#23 stay as they are.
- **Review.** Frames extracted at 1 s intervals and checked for window chrome, seed seams (overlapping blocks, "Fill it?" lines, the clock hand off the live block), the cycle-jump artefact (SliceFocus#455: a fresh slice and session per take), unreadable text, an abrupt loop.

## 4. What shipped (2026-10-23)

The founder approved the cut on 2026-10-23 after two notes (hold on SHORT BREAK before the loop; no flash of unrelated rows while the list scrolls). Files: [`videos/v1.7.1/`](../videos/v1.7.1/) `hero.mp4`, `hero.webm`, `hero-poster.jpg`, `hero-poster.webp`.

| | |
| --- | --- |
| Length | 15.7 s, silent, seamless loop (0.6 s dissolve) |
| hero.mp4 | H.264, 1440x900, 30 fps, BT.709, no audio, 0.97 MB |
| hero.webm | VP9, same frame, 0.79 MB |
| Source | one real take on the demo account, Classic Pomodoro, 2026-10-23 12:12 to 12:38 UTC |

Where it differs from the storyboard above, and why:

- **One handoff, not two.** The take ended before the break did, so the loop closes on SHORT BREAK counting (05:00 to 04:57, held for about 2 s), not on the next WORK.
- **No 00:00 on screen.** When the work phase runs out, the timer card holds 00:00 for about 2 s while the server hands the session over, and shows an Advance button (the manual fallback) and the next cycle's number early. Those frames are cut: 00:01 goes straight to SHORT BREAK 05:00, which is the app's own phase change. Shown, the button would suggest a click is needed, which is the opposite of the point.
- **Push-in 1.08x, not up to 1.15x.** At 1.12x the timer card's Stop and Pause buttons left the frame.
- **No iPhone scene.** As planned in section 2.
- **A mouse pointer stood still in the take** (parking it failed; screen video always films it). It was painted over with the flat strip beside it (`compose.sh --hide-pointer`); `run.sh` now refuses to record while the pointer is inside the recorded rectangle.
- **The new block is Deep Work blue** next to two other blue blocks, so it reads by its sweep and its row more than by its colour. A retake without the category tap would keep the dialog's default yellow, which stands out on the canvas.

Rebuild from the raw take (kept on the founder's Mac, `~/Desktop/site-videos/hero-v2/raw/`):

```bash
tool/videos/hero/compose.sh ~/Desktop/site-videos/hero-v2/raw/mac.mov \
  ~/Desktop/site-videos/hero-v2/raw/markers.txt ~/Desktop/site-videos/hero-v2/clips \
  --hide-pointer 1436:846:54:70
```

## 5. Version 3: a feature tour with visible taps (2026-11-02)

The founder's verdict on the shipped cut (2026-11-02): it does not show Focus Shield, the lock screen, the session across devices, calendar export or categories, and the taps are not emphasised. Issue: [SliceFocusFE#658](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/658). ADR-042 amended again (2026-11-02): a tour is allowed, taps are marked with an expanding ring, a real still may stand in where a moment cannot be recorded.

| Time | Beat | On screen | Source |
| --- | --- | --- | --- |
| 0–6.2 | plan | the open day, the sweep on the ring (ring at the press), Create Slice, "Deep work", the Deep Work chip (ring), Save (ring) | the 2026-10-23 Mac take |
| 6.2–8.8 | start | the row's play control (ring), the timer opens on 25:00 | same |
| 8.8–15.1 | auto-advance | the countdown racing, 00:01, SHORT BREAK by itself, held 2 s | same |
| 15.1–18.3 | one session, every device | iPhone next to the Mac, both on the same running "Deep work" session; the phone's expand tap (ring) | a `both` take: the server starts the session, the phone and the Mac show it |
| 18.3–21.3 | lock screen | the iPhone locked, the Live Activity counting, the Mac still counting beside it | same take, the simulator locked on the `phone:lock` marker |
| 21.3–24.1 | Focus Shield | the real block page, "Shielded by SliceFocus until 19:45", slow push-in | `slicefocus-web/focus-shield/blocked-page.png` |
| 24.1–29.6 | categories | Settings (ring), Manage Categories (ring), + (ring), "Writing", Save (ring), the new category in the list | a Mac take, scene `categories` |
| 29.6–32.6 | calendar | Settings, "Add slices to calendar" (ring), the switch turns on and its options appear | same take, scene `calendar` |
| 32.6–33.2 | loop | dissolve to the first frame | |

Pipeline additions (SliceFocusFE#658): `filmedTap` in the test prints `CLIP:<scene>:tap x y`; `run.sh` keeps the coordinates with the time, starts the phone's session on the server (`seed.py --start-block`) on the `prepare` line, and locks the simulator on `phone:lock` (the Simulator app is absent from this Xcode, so the lock button is pressed through the iOS Simulator tool and a flag file tells `run.sh` to go on); `compose.sh` takes several takes and a still, draws the rings before the camera move, composes the phone beside the Mac on the site's cream, and closes the loop as before.

### What shipped in v3 (2026-11-02)

Files: [`videos/v1.7.1/`](../videos/v1.7.1/) `hero.mp4` (H.264, 1440x900, 30 fps, BT.709, no audio, 1.71 MB at crf 22), `hero.webm` (VP9, 1.47 MB at crf 35), `hero-poster.jpg` / `.webp`. 33.1 s, a 0.6 s loop dissolve. Eight shots: plan (6.2 s), start (2.6 s), the countdown and the handoff (6.1 s), the phone beside the Mac (3.2 s), the lock screen (2.4 s), Focus Shield (2.8 s), categories (6.0 s), calendar (4.5 s). Ten tap rings.

Where it differs from the table above:

- **The lock beat is the site's earlier lock-screen clip** (`beat3-lockscreen.mp4`, 2.35 s of it), the phone alone on the cream: on the simulator the Live Activity's countdown renders as dashes (`24:––`) and the first locks show the "Allow Live Activities" sheet. That clip's session is "Exercise", not the Mac's "Deep work", which is why the Mac is not beside it.
- **The phone beside the Mac** comes from a take in which the server started the session (`seed.py --start-block`), so both devices show the same "Deep work 24:3x" in step; the phone's tap opens its timer.
- **The 00:00 frame** is cut from the handoff as in v2.
- **Two takes were junk** before the first good one: the app now signs in as a guest at launch (ADR-064), so the demo driver signs the demo account in before launch; and two macOS "find devices on local networks" prompts sat over the recorded area until the founder answered them.

### What shipped in v3.1 (2026-11-07)

The v3 cut blinked: every tap ring ended with a solid black square for one frame. The founder noticed it on 2026-11-06, before spending on promotion ([SliceFocusFE#661](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/661)). The cause was in the pipeline, not the takes: ImageMagick stored the ring's last, fully transparent frame as a 1-bit grey PNG without alpha, which ffmpeg overlaid as opaque black. Reviewing the recomposed hero with the founder led to three more changes, all in [SliceFocusFE#662](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/pull/662); the files shipped in [slicefocus-web#53](https://github.com/AnunnakiCosmoCrew/slicefocus-web/pull/53), merged 2026-11-07.

- **No flash.** Ring frames are written as 8-bit RGBA (`PNG32:`), and `compose.sh` checks that each decodes as `rgba` before any overlay.
- **No shake.** The slow push-ins used `zoompan`, which crops on whole pixels, so a 3 % zoom over 7 s stepped the picture 1–2 px every few frames. A `camera()` helper now maps the eased source rectangle with `perspective` and scales with lanczos.
- **Slower round the taps.** Rings last 0.67 s instead of 0.4 s (`SF_RING_FRAMES`), each tap sits in a moment at normal speed, and the categories and calendar beats run at 1.0–1.25x instead of 1.2–1.6x.
- **Focus Shield has its cause.** At 22 s a browser's new tab with shortcut tiles (Instagram, YouTube, TikTok, News, Weather), a tap on the Instagram tile, the tab becomes Instagram with `instagram.com` in the address bar, and the real block page loads where the site would be (`hero/browser-frame.sh`). The three tiles carry the sites' own favicons, the founder's call: third-party marks on the public site, shown as sites a user blocks.

Files: [`videos/v1.7.1/`](../videos/v1.7.1/) `hero.mp4` (1,768,865 bytes, crf 22), `hero.webm` (1,356,021 bytes, crf 38), `hero-poster.jpg` / `.webp`, replacing the v3 copies. 38.6 s, same takes (2026-10-23 and 2026-11-02), 1440x900, 30 fps, BT.709, no audio. Checked: a per-frame scan for a large black area appearing for one frame found 7 such frames in v3 and none here; slit scans over every camera move show smooth edges.
