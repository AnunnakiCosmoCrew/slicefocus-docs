# ADR-064: New users start as guests on a silent anonymous account

**Status:** Accepted (founder, 2026-10-26). The list of account-only features under *Decision* §5 is the default to confirm before the app issue starts.
**Date:** 2026-10-26
**Deciders:** Mert Ertugrul
**Amends:** [ADR-026](026-multi-provider-oidc.md) (auto-provisioning no longer needs an email), [ADR-003](003-stateless-jwt-auth.md) (a Firebase anonymous token is a valid credential)
**Relates to:** [ADR-062](062-server-owned-focus-timeline-clients-render.md) (the server owns the focus timeline), [ADR-063](063-chrome-focus-shield-state.md) (the Chrome extension's own sign-in)

## Context

Every new user meets a sign-in screen before they can start a timer or place a block. The app's numbers after install are poor: 15 % of new users ever finish one focus session, 3 % come back on day 1 and 2 % on day 7 (n ≈ 98, mostly before 1.7). The cross-category iOS average for day 1 is about 25 % (vendor data). The sign-in screen is the main suspect for the day-1 drop.

Apple's App Review Guideline 5.1.1(v) says: "If your app doesn't include significant account-based features, let people use it without a login." Show HN's rules ask that people can try a product "without barriers such as signups". The promotion research of 2026-10-26 found no promotion channel worth scaling until the first session works, because every launch would decay to nothing.

A guest cannot simply run the app on the device alone. Since ADR-062 the server owns the focus timeline: it decides each WORK to BREAK handover and moves the lock-screen Live Activity forward by APNs push. iOS runs no app code while the phone is locked. So a device-only timer would freeze the Live Activity at 00:00, which is the moment the product is built around.

## Decision

1. **A first launch with no Firebase user signs in anonymously** (`FirebaseAuth.signInAnonymously`). There is no screen. The home page opens on today's canvas. The login page stays, reached from Settings and from the prompts in §4.
2. **The backend treats an anonymous Firebase token as a guest user.** It recognises the `firebase.sign_in_provider` claim `anonymous`. It provisions a user by its subject alone, with no email and a guest flag. `users.email` becomes nullable, unique where present. Everything else (slices, sessions, the server timeline, Live Activities, pushes, preferences) works for a guest exactly as for anyone else.
3. **Upgrading keeps the same account.** The app links the chosen credential (Apple, Google, email) to the anonymous Firebase user with `linkWithCredential`. The Firebase uid, and so the backend's subject, does not change. On the first request whose token carries an email for a guest user, the backend fills in the email and name and clears the guest flag. No data moves.
4. **When the credential already has an account, the existing account wins and the guest's data is merged into it.** Before switching, the app keeps the guest's ID token. It signs in to the existing account, then calls a merge endpoint with the guest token. The backend checks that the token is an anonymous one for a guest user. It moves the guest's data under these rules:
   - categories merge by name
   - focus-session history moves
   - slices move, except ones that overlap a slice the account already has on that date; those are skipped and counted
   - the guest's preferences are dropped
   - a running guest session moves only if the account has none; otherwise it is completed

   The backend then deletes the guest user and its Firebase anonymous account. The app shows what moved and what was skipped.
5. **Sign-in is offered at moments of value, never as a gate.** It is offered after the third completed focus session ("keep your history"), when the user opens the Mac app or the Chrome extension, and when they open trends or planned vs actual. Each prompt shows at most once per moment and can be dismissed. Default split, to confirm:
   - **Guests get** the timer and auto-advance, the canvas, categories, templates and suggestions, the Focus Shield on that device, the Live Activity and notifications, Calendar and Reminders, and the last 7 days of history.
   - **An account adds** the same day on every device (iPhone, Mac, Chrome), full history, trends and planned vs actual, and data that survives a reinstall.
6. **Guest accounts with no use for 90 days are deleted**, with their data and their Firebase anonymous account. The account page tells a guest this, and says that deleting the app deletes a guest's data.
7. **The privacy policy names guest accounts** before the release ships: what is stored without an email, the 90-day rule, and how to delete it. The App Store privacy label is checked against it.

## Alternatives Considered

- **Device-only guest mode.** The app would run a local timer and keep data on the phone until sign-in. Rejected for three reasons:
  - It brings back a second owner of the timeline, which ADR-062 removed after several cycle bugs.
  - It freezes the lock-screen Live Activity at 00:00 at the moment a new user is judging the product.
  - It needs an upload-and-merge of every local record at sign-in.

  The offline work (local cache and queued writes) remains a separate need.
- **Keep the sign-in screen, make it lighter** (Sign in with Apple first, fewer fields). Rejected: it is still a wall before any value, and Guideline 5.1.1(v) points the other way.
- **Sign-in after a few onboarding screens.** Rejected for the same reason: the user still has to sign in before the first timer.
- **Discard guest data on a credential conflict instead of merging.** Simpler, but it punishes exactly the users who come back on a second device. Kept only as the fallback when the merge fails.

## Consequences

- **Easier:**
  - A new user reaches the first timer in one tap.
  - Show HN, a public TestFlight link and an App Store featuring nomination become possible.
  - The whole server path stays as it is.
- **Harder:**
  - The backend stores data for people without an email. That needs the privacy line, the cleanup job and care in any code that assumes `email` is present (profile display, audit logs, account linking in `AccountLinkingService`, the email fallback in `UserService.resolveCurrentUser`).
  - Merging is the riskiest piece: it writes across slices, sessions and categories in one transaction, so it needs its own tests.
  - A Mac opened before the iPhone is linked starts its own guest account. The two only meet after sign-in, through the merge.
- **Abuse:** anyone can mint accounts. The per-user rate limit applies. Firebase App Check and Firebase's own anonymous sign-up limits are the next lines if guest creation is abused.
- **Founder steps:** enable the Anonymous provider in the Firebase console for every environment, and review the privacy policy wording and the App Store privacy label.
- **Measurement:** the activation funnel should split guests from signed-in users, so the release shows whether the wall was the cause.

## Rollout

Epic [SliceFocus#457](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/457), for the release after 1.7.3:

| Issue | What | Waits for |
| --- | --- | --- |
| [SliceFocus#458](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/458) | Backend accepts anonymous tokens and provisions guests (§2) | nothing |
| [SliceFocus#459](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/459) | Fill in the account on link; merge endpoint (§3, §4) | #458 |
| [SliceFocus#460](https://github.com/AnunnakiCosmoCrew/SliceFocus/issues/460) | Delete guests after 90 days without use (§6) | #458 |
| [SliceFocusFE#630](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/630) | Silent anonymous sign-in at first launch; guest account page (§1) | #458 |
| [SliceFocusFE#631](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/631) | Link from a guest, or merge into an existing account (§3, §4) | #459, #630 |
| [SliceFocusFE#632](https://github.com/AnunnakiCosmoCrew/SliceFocusFE/issues/632) | Sign-in prompts at moments of value; account-only features (§5) | #630 |
| [slicefocus-docs#14](https://github.com/AnunnakiCosmoCrew/slicefocus-docs/issues/14) | Privacy policy and App Store privacy label (§7) | ships before the release |
