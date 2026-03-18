# ADR-003: Stateless JWT authentication with Firebase/Google Identity

**Status:** Accepted
**Date:** 2026-02-20
**Deciders:** Mert Ertugrul

## Context

The app needs authentication that works across multiple platforms (macOS, iOS, Android) and supports social login (Google, Apple). The backend runs on Cloud Run which scales to zero and can have multiple instances.

## Decision

Use stateless JWT authentication. The frontend authenticates via Firebase Auth (Google/Apple OAuth), receives a JWT, and sends it as a Bearer token. The backend validates the JWT signature against Google's public keys — no session storage needed.

## Alternatives Considered

- **Session-based auth**: Requires sticky sessions or shared session store (Redis). Doesn't work with Cloud Run scale-to-zero. Adds infrastructure complexity.
- **Custom JWT issuance**: Build our own auth server. Massive security risk for a solo developer. Firebase handles token rotation, revocation, and provider integration.
- **API keys**: No user identity, no OAuth, no social login. Not suitable for a user-facing app.

## Consequences

- Zero session storage — backend is fully stateless, perfect for Cloud Run autoscaling
- IDOR protection returns 404 (not 403) to avoid leaking resource existence
- Firebase handles token refresh, social provider integration, and account management
- Backend trusts Firebase token claims — if Firebase is compromised, so is the app
- No server-side session revocation (must wait for token expiry or use Firebase's revocation API)
