---
layout: default
title: Privacy Policy — SliceFocus
---

# Privacy Policy

**SliceFocus**
*Last updated: June 25, 2026*

AnunnakiCosmoCrew ("we", "us", or "our") operates the SliceFocus mobile and desktop application (the "App"). This Privacy Policy explains how we collect, use, and protect your information when you use our App.

By using SliceFocus, you agree to the collection and use of information in accordance with this policy.

---

## 1. Information We Collect

### 1.1 Account Information

When you sign in, we collect:

- **Email address** — provided by your Google or Apple account during sign-in
- **Full name** — provided by your authentication provider
- **Authentication provider identifier** — a unique ID from Google or Apple used to link your account

We do not collect or store your password. Authentication is handled entirely by Google Sign-In or Sign in with Apple via Firebase Authentication.

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

We use **Firebase Analytics** to understand how the App is used so we can improve it. We log a small set of usage events:

- **App opens** — when the App is launched
- **Focus session completions** — including the number of completed cycles and total focus minutes for the session

These analytics events contain **no name, email, or schedule content**. Firebase Analytics assigns a pseudonymous, app-generated instance identifier to associate events from the same installation; this identifier is not linked to your account profile by us. Analytics collection is **disabled in debug/development builds** and only active in production releases.

### 1.7 Website Analytics (slicefocus.app)

Our marketing website ([slicefocus.app](https://slicefocus.app)) uses **Google Analytics 4** to measure visits and how visitors reach the App Store. We record page views and clicks on the "Download on the App Store" buttons, along with a campaign label derived from the link you arrived through (for example, a Reddit or Hacker News post). This website analytics:

- uses cookies to count visits and recognise a returning browser within a session;
- collects **no name, email, or account information**;
- does **not** track you across other websites and is **not** used for advertising or profiling;
- does not store your IP address (Google Analytics 4 does not log IP addresses).

We cannot link an individual website visitor to an individual App Store download. This analytics applies only to the website, not to the App.

---

## 2. How We Use Your Information

We use the collected information to:

- **Provide the App's core functionality** — display your schedule, run Pomodoro timers, and sync data across your devices
- **Send push notifications** — notify you of focus session phase transitions, session completions, and scheduled time block reminders
- **Maintain security** — audit sensitive operations (account linking, preference changes), enforce rate limits, and correlate requests for debugging
- **Improve reliability** — identify and fix crashes via Firebase Crashlytics
- **Improve the product** — understand aggregate usage (app opens, session completions) via Firebase Analytics
- **Monitor service health** — track aggregated usage metrics to ensure the backend is performing well

We do **not** use your data for advertising, profiling, or selling to third parties.

---

## 3. Data Storage and Security

### 3.1 Where Your Data Is Stored

- **Backend data** (account, slices, sessions, preferences) is stored in a PostgreSQL database hosted by [Neon](https://neon.tech) in the EU region
- **Authentication** is managed by [Firebase Authentication](https://firebase.google.com/products/auth) (Google Cloud infrastructure)
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
| Firebase Authentication | Google | Sign-in and identity management | Email, OAuth tokens |
| Firebase Cloud Messaging | Google | Push notification delivery | Device tokens, notification payloads |
| Firebase Crashlytics | Google | Crash reporting | Crash logs, device info, app version |
| Firebase Analytics | Google | Product usage analytics | Usage events (app opens, session completions), pseudonymous app-instance ID |
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
- **Request account deletion** — contact us at the email below to request full deletion of your account and all associated data

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

**Email:** merty.ertugrul@gmail.com

---

*This privacy policy is hosted at [https://anunnakicosmocrew.github.io/slicefocus-docs/legal/privacy-policy](https://anunnakicosmocrew.github.io/slicefocus-docs/legal/privacy-policy)*
