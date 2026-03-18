# ADR-033: API Client with Retry Logic and Rate-Limit Awareness

**Status:** Accepted
**Date:** 2026-03-18
**Deciders:** Mert Ertugrul

## Context

Network requests can fail transiently (timeouts, socket errors) or be throttled by the backend's rate limiter (HTTP 429 Too Many Requests). Without retry logic, transient failures surface as user-visible errors. Without rate-limit awareness, aggressive retries worsen throttling and may trigger longer bans.

## Decision

Implement a custom `ApiClient` wrapper with the following capabilities:

- **Exponential backoff retry**: Delays of 1s, 2s, 4s with a maximum of 2 retries for transient failures (`TimeoutException`, `SocketException`).
- **Rate-limit detection**: On 429 status, read the `Retry-After` header and wait before retrying. Do not count rate-limited attempts against the retry budget.
- **Firebase JWT injection**: A `tokenProvider` callback supplies the current auth token, injected as `Authorization: Bearer <token>` on every request.
- **Request correlation**: An `X-Request-Id` (UUID v4) header on every request enables end-to-end tracing with backend logs.

## Alternatives Considered

- **`retry` package or `dio` with interceptors**: Adds an external dependency with less control over rate-limit and correlation-header behavior.
- **No retry**: Poor UX on flaky mobile networks where transient failures are common.
- **Fixed-delay retry**: Inefficient — doesn't back off, risks overwhelming the server during degraded conditions.

## Consequences

- Transparent retry for transient failures improves perceived reliability on mobile networks
- Rate-limit awareness prevents retry storms and respects backend throttling policy
- `X-Request-Id` enables end-to-end request tracing across client and server logs
- Custom implementation means maintaining retry logic manually (but logic is ~80 lines, well-tested)
- Non-retryable errors (4xx except 429) fail fast without unnecessary delay

*Related: MER-270, MER-271, MER-252*
