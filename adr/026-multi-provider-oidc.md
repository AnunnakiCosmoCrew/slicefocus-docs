# ADR-026: Multi-Provider OIDC (Google + Apple) with Auto-Provisioning

**Status:** Accepted
**Date:** 2026-03-18
**Deciders:** Mert Ertugrul

## Context

ADR-003 established stateless JWT authentication, but the actual identity layer is more nuanced. The app supports two OIDC issuers — Google and Apple — each with distinct behaviors. Apple only sends user profile data (name, email) on the very first login; subsequent tokens omit it. Additionally, desktop platforms (macOS) cannot rely on Firebase auth state streams because they are unreliable, requiring explicit state forcing after sign-in completes.

## Decision

Use `provider_subject` (the `sub` claim from the OIDC token) as the stable identity key rather than email. When the backend receives a valid token from an unknown subject, it auto-provisions a new user record — no separate registration flow exists.

Supported providers:
- **Google OAuth 2.0** — standard OIDC flow via Firebase Authentication
- **Apple Sign-In** — captures profile data (display name, email) on first authentication only and persists it immediately, since Apple will not send it again

On desktop platforms (macOS), the client explicitly forces auth state updates after sign-in completes, bypassing the unreliable Firebase `authStateChanges()` stream.

## Alternatives Considered

- **Single provider (Google only):** Simpler integration but excludes users who prefer Apple-ecosystem sign-in and limits App Store compliance on iOS/macOS.
- **Email as identity key:** Fragile — users can change their email, and Apple's "Hide My Email" relay addresses add complexity. The `sub` claim is immutable per provider.
- **Manual registration flow:** Adds friction. Auto-provisioning on first valid token provides a seamless onboarding experience with zero extra screens.

## Consequences

- Users can sign in with either Google or Apple from day one, broadening adoption
- Apple's one-time profile data delivery requires careful first-login handling; missing the capture means no display name until the user manually sets one
- Desktop (macOS) needs a platform-specific auth workaround, increasing platform-aware code in the auth layer
- The `provider_subject` key ensures identity stability regardless of email changes or provider-specific quirks

*Related: MER-153, MER-165, MER-166, MER-167, MER-168, MER-169, MER-183*
