# ADR-044: Web analytics on slicefocus.app (GA4 + Apple campaign tokens)

**Status:** Accepted
**Date:** 2026-06-25
**Deciders:** Mert Ertugrul

## Context

The launch-push epic ([#309](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/309))
is firing its distribution moments (Reddit, then Show HN) and its Definition of
Done requires a dashboard showing progress toward 100 downloads / 20 WAUs. But the
marketing site **slicefocus.app was completely uninstrumented**: no analytics
snippet, and the App Store links carried no campaign attribution. There was no way
to see how many people a post sent to the site, how many tapped "Get the app," or
how many downloaded — the launch funnel was invisible.

Two hard constraints shape the design:

1. **The App Store identity boundary.** On iOS, a web visitor (analytics client ID)
   and the resulting install (app instance ID) cannot be joined by any free tool.
   The funnel can only be measured as aggregate counts across two systems, not as a
   stitched per-user path.
2. **Audience is partly EU / Türkiye (KVKK / GDPR)** and GA4 uses cookies, so consent
   handling is in scope.

The app itself already reports product telemetry via Firebase Analytics
([ADR-040](040-firebase-analytics-telemetry.md)); this decision covers the **website**,
a separate data-processing context.

## Decision

Instrument slicefocus.app with **Google Analytics 4 via gtag.js**, plus **Apple App
Store campaign tokens**, to make the download funnel visible:

- **Dedicated GA4 web data stream** (`G-VBK98EW3WQ`) in the SliceFocus Firebase
  project, separate from the auto-generated app build streams, so marketing-site
  traffic does not pollute app analytics.
- **`page_view`** is collected automatically (top of funnel).
- **`app_store_click`** custom event on all three App Store links (header / hero /
  footer), tagged with the link location and sent with `transport_type: 'beacon'`
  so the hit survives the navigation to the App Store.
- **Per-channel attribution via a dynamic `ct` token.** On load, the site reads the
  inbound `?utm_source` and stamps every App Store link with a matching Apple
  campaign token (`utm_source=reddit_pomodoro` → `ct=reddit_pomodoro`; `hn`; default
  `web`). App Store Connect → App Analytics then splits **downloads** by channel.
  Links remain bare in source; the token is applied at runtime.
- **Deferred consent, with hooks in place.** Ship now with a Consent Mode v2
  `default` call (`analytics_storage: 'granted'`) and a short footer privacy note;
  GA4 does not store IP addresses. A full consent banner is deferred — it is a
  one-line flip to `denied` plus a banner calling `gtag('consent','update',…)`,
  to be added before any real scale or paid advertising.

The funnel is therefore read as: **post → `page_view` (channel via UTM) →
`app_store_click` → download (App Store Connect, by `ct` token).**

## Alternatives Considered

- **Reuse an existing app web data stream.** Rejected: the `slice_focus_fe (web)`
  build stream exists, but routing marketing-site events through it would mix
  website visitors into the app's user counts and muddy both.
- **Cloudflare Web Analytics (cookieless, no banner).** Rejected: privacy-simple and
  we are already on Cloudflare, but it lives outside Firebase and cannot cleanly
  model the `app_store_click` → per-channel-download funnel that is the whole point.
- **Full consent banner before launch.** Deferred: at this scale (tens of visitors)
  a default-deny banner would gut the very launch data it exists to protect; the
  practical risk is negligible and GA4 stores no IPs. Revisit before scale / ads.
- **Paid mobile measurement partner (Branch / AppsFlyer) for true per-user
  attribution.** Rejected: the only thing that defeats the iOS identity boundary, but
  far too heavy for a solo launch. Aggregate counts are sufficient.

## Consequences

- **Easier:** the launch funnel is now measurable — GA4 for visits + store clicks,
  App Store Connect for downloads — and Reddit vs HN vs organic can be compared,
  which is what makes the keep-pushing-vs-pivot decision data-driven.
- **Constraint (by design):** an individual visitor cannot be tied to an individual
  install; we report aggregates only. Apple's `ct` attribution also undercounts —
  it credits a download only when the user installs directly from the tagged link,
  not if they later search the store.
- **Dependency:** per-channel splitting requires the **posted links to carry
  `?utm_source=…`** (e.g. `slicefocus.app/?utm_source=hn`). Untagged links fall
  through to `ct=web`.
- **Privacy:** the website's analytics + cookie use is now disclosed in the privacy
  policy (§1.7) and the site footer.
- **Debt accepted:** no consent banner yet. Tracked as a follow-up; must land before
  meaningful EU/TR scale or any paid acquisition.
