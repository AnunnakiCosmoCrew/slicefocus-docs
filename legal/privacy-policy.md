---
layout: default
title: Privacy Policy — SliceFocus
---

# Privacy Policy

**SliceFocus**
*Last updated: October 27, 2026*

SliceFocus is a CosmoCrew app. CosmoCrew is the consumer-apps brand of Luvita Teknoloji Ltd. Şti. ("Luvita", "we", "us", or "our"), which operates the SliceFocus mobile and desktop application (the "App") and is responsible for the personal data described in this policy (the data controller). This Privacy Policy explains how we collect, use, and protect your information when you use our App.

By using SliceFocus, you agree to the collection and use of information in accordance with this policy.

---

## 1. Information We Collect

### 1.1 Account Information

**Guest accounts.** You can use SliceFocus without signing in. The first time you open the App, Firebase Authentication creates an anonymous account for you. A guest account holds no email address and no name: it is identified only by a random account identifier that Firebase generates. The schedule and focus data you create (section 1.2) are stored under that identifier on our servers, so that the timer, notifications and lock-screen timer work as they do for any account. We also record when the account was last used, so that we can delete guest accounts nobody uses any more (section 5).

**Signed-in accounts.** When you sign in, with Google, with Apple or with an email address, we collect:

- **Email address** — provided by Google or Apple, or the address you sign up with
- **Full name** — provided by your sign-in provider, if it supplies one
- **Authentication provider identifier** — a unique ID from Google, Apple or Firebase used to link your account

**From guest to signed in.** When a guest signs in, the guest account becomes the signed-in account and keeps all its data. If that sign-in already belongs to an existing SliceFocus account, the guest's data is added to the existing account (items that overlap existing ones are left out, and the App tells you which) and the guest account is then deleted.

We do not store your password. Sign-in, including email-and-password sign-in, is handled by Firebase Authentication.

### 1.2 Schedule and Productivity Data

When you use the App, we store:

- **Time slices** — task names, scheduled start and end times, and color labels you assign to your daily schedule
- **Focus sessions** — Pomodoro session records including start time, end time, number of completed cycles, total focus minutes, and session status (running, completed, cancelled)
- **User preferences** — theme (light/dark), soundscape selection, notification settings, and timer duration preferences (work, short break, long break, cycles before long break)

### 1.3 Device Information

To deliver push notifications, we collect:

- **Device token** — a Firebase Cloud Messaging (FCM) token that identifies your device for push notification delivery
- **Platform** — whether you are using iOS, Android, or macOS

### 1.4 Crash and Error Data

We use Firebase Crashlytics to collect:

- **Crash reports** — unhandled exceptions and stack traces
- **App diagnostics** — app version, operating system version, and device model

Crash data collection is **disabled in debug/development builds** and only active in production releases.

### 1.5 Usage Metrics

We collect anonymous, aggregated metrics for service health monitoring:

- Total focus sessions started, completed, and cancelled (counts only, no user identifiers)
- API request performance (response times, error rates)

These backend metrics contain **no personally identifiable information**.

### 1.6 Product Analytics

We use **Firebase Analytics** to understand how the App is used so we can improve it, in particular how many new users reach their first completed focus session. We log a small set of usage events:

- **App opens** — when the App is launched
- **Sign-in steps** — when the sign-in screen is shown, and when you sign in or create an account, with the method used (Apple, Google or email)
- **Planning and focus** — when you create a time slice (with where you created it: on the dial, in the list, from a template or from a suggestion), when you start a focus session, and when you complete one (with the number of completed cycles and total focus minutes)
- **Screens** — when you open your session history, analytics, planned vs actual, or trends
- **Prompts** — when the App asks the system to show the App Store rating request, and when a suggestion to sign in is shown, accepted or dismissed
- **Account state** — whether the installation is used as a guest, signed in, or signed out

These analytics events contain **no name, email, or schedule content** (no task names or times). Firebase Analytics assigns a pseudonymous, app-generated instance identifier to associate events from the same installation; this identifier is not linked to your account profile by us. Analytics collection is **disabled in debug/development builds** and only active in production releases.

### 1.7 Website Analytics (slicefocus.app)

Our marketing website ([slicefocus.app](https://slicefocus.app)) uses **Google Analytics 4** to measure visits and how visitors reach the App Store. We record page views and clicks on the "Download on the App Store" buttons, along with a campaign label derived from the link you arrived through (for example, a Reddit or Hacker News post). This website analytics:

- uses cookies to count visits and recognise a returning browser within a session;
- collects **no name, email, or account information**;
- does **not** track you across other websites and is **not** used for advertising or profiling;
- does not store your IP address (Google Analytics 4 does not log IP addresses).

We cannot link an individual website visitor to an individual App Store download. This analytics applies only to the website, not to the App.

### 1.8 Chrome Extension (SliceFocus Focus Shield)

If you install the **SliceFocus Focus Shield** Chrome extension, it signs you into your SliceFocus account (Google or email) and reads your active focus session from our servers to know when a work phase runs. It stores your sign-in session and your block list on your device (the block list syncs through your Chrome profile). It registers your browser for push messages with Firebase Cloud Messaging so it can react when a phase starts or ends. It blocks the sites on your list by browser rules and does **not** read, record or send your browsing history or page content. It uses no analytics or advertising, and we do not sell your data. Uninstalling the extension removes its local data; sign out in the popup first to also remove the push registration from our servers.

---

## 2. How We Use Your Information

We use the collected information to:

- **Provide the App's core functionality** — display your schedule, run Pomodoro timers, and sync data across your devices
- **Send push notifications** — notify you of focus session phase transitions, session completions, and scheduled time block reminders
- **Maintain security** — audit sensitive operations (account linking, preference changes), enforce rate limits, and correlate requests for debugging
- **Improve reliability** — identify and fix crashes via Firebase Crashlytics
- **Improve the product** — understand aggregate usage (for example, how many new users complete their first focus session) via Firebase Analytics
- **Monitor service health** — track aggregated usage metrics to ensure the backend is performing well

We do **not** use your data for advertising, profiling, or selling to third parties.

---

## 3. Data Storage and Security

### 3.1 Where Your Data Is Stored

- **Backend data** (account, slices, sessions, preferences) is stored in a PostgreSQL database hosted by [Neon](https://neon.tech) in the EU region
- **Authentication**, including guest accounts, is managed by [Firebase Authentication](https://firebase.google.com/products/auth) (Google Cloud infrastructure)
- **Push notification tokens** are stored in our database and transmitted to Firebase Cloud Messaging for delivery
- **Crash reports** are stored in Firebase Crashlytics (Google Cloud infrastructure)
- **Analytics events** are processed and stored by Firebase Analytics (Google Cloud infrastructure)
- **Local preferences** (theme, timer settings, offline queue) are stored on your device using platform-native storage and are not transmitted unless explicitly synced

### 3.2 Security Measures

- All API communication is encrypted via HTTPS/TLS
- Authentication uses industry-standard JWT tokens validated against Google's public key infrastructure
- All database queries are scoped by user ID — you can only access your own data
- The backend runs in a non-root container on Google Cloud Run with no static API keys (OIDC + Workload Identity Federation)
- Sensitive operations are logged in an audit trail for security compliance

---

## 4. Third-Party Services

SliceFocus uses the following third-party services that may process your data:

| Service | Provider | Purpose | Data Shared |
|---------|----------|---------|-------------|
| Firebase Authentication | Google | Sign-in, guest accounts and identity management | Email and OAuth tokens; for a guest account, only an anonymous identifier |
| Firebase Cloud Messaging | Google | Push notification delivery | Device tokens, notification payloads |
| Firebase Crashlytics | Google | Crash reporting | Crash logs, device info, app version |
| Firebase Analytics | Google | Product usage analytics | Usage events (section 1.6), account state (guest or signed in), pseudonymous app-instance ID |
| Google Analytics 4 | Google | Website (slicefocus.app) visit + App Store click analytics | Page views, App Store button clicks, campaign label; pseudonymous cookie ID. No IP stored. |
| Google Sign-In | Google | OAuth authentication | Email, name, Google account ID |
| Sign in with Apple | Apple | OAuth authentication | Email, name, Apple ID |
| Neon | Neon Inc. | Database hosting | All backend-stored data (encrypted at rest) |
| Google Cloud Run | Google | Backend hosting | API requests and responses |

Each third-party service is governed by its own privacy policy:
- [Google Privacy Policy](https://policies.google.com/privacy)
- [Apple Privacy Policy](https://www.apple.com/legal/privacy/)
- [Neon Privacy Policy](https://neon.tech/privacy-policy)

---

## 5. Data Retention

- **Account and productivity data** is retained as long as your account is active
- **Guest accounts** and all their data are deleted automatically after **90 days without use**. A guest account cannot be opened on another device or after you delete the App: installing the App again starts a new guest account, and the old one is deleted after 90 days. Sign in to keep your data
- **A guest account that signs in to an existing account** is deleted as soon as its data has been added to that account
- **Focus session history** is retained indefinitely to support productivity analytics and reporting
- **Device tokens** are retained until the device is unregistered or the token is refreshed
- **Audit logs** are retained for security compliance purposes
- **Crash reports** are retained according to Firebase Crashlytics' default retention policy (90 days)
- **Analytics events** are retained according to Firebase Analytics' configured retention policy
- **Local device data** is retained until you uninstall the App

---

## 6. Your Rights

You have the right to:

- **Access** your data — all your schedule and session data is visible within the App
- **Update** your data — you can edit your profile name, slices, and preferences at any time
- **Delete** your data — you can delete individual slices and focus sessions within the App
- **Delete your account** — Settings › Account › Delete account deletes your account and all associated data, for guest and signed-in accounts alike. You can also contact us at the email below to request deletion

If you are in the European Economic Area (EEA), you have additional rights under GDPR including the right to data portability, restriction of processing, and the right to lodge a complaint with a supervisory authority.

---

## 7. Children's Privacy

SliceFocus is not directed at children under the age of 13. We do not knowingly collect personal information from children. If you believe a child has provided us with personal data, please contact us and we will delete it.

---

## 8. Changes to This Policy

We may update this Privacy Policy from time to time. We will notify you of significant changes by posting the new policy in the App or on this page. The "Last updated" date at the top of this policy indicates when it was last revised.

---

## 9. Contact Us

If you have any questions about this Privacy Policy or wish to exercise your data rights, please contact us at:

**Email:** contact@cosmocrew.dev

**Who we are:** Luvita Teknoloji Ltd. Şti. · MERSİS 0609147476900001 · Bodrum, Muğla, Türkiye

---

*This privacy policy is hosted at [https://anunnakicosmocrew.github.io/slicefocus-docs/legal/privacy-policy](https://anunnakicosmocrew.github.io/slicefocus-docs/legal/privacy-policy)*

© 2026 CosmoCrew — a brand of [Luvita Teknoloji Ltd. Şti.](https://luvita.tr/)
