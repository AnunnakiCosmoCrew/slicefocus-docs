# ADR-007: WebSocket with STOMP for real-time sync

**Status:** Accepted
**Date:** 2026-03-01
**Deciders:** Mert Ertugrul

## Context

Users need to see focus session state (timer, phase) update in real-time across devices (e.g., start Pomodoro on Mac, see countdown on phone).

## Decision

Use Spring WebSocket with STOMP message broker for real-time session events. Backend publishes events to user-specific topics; frontends subscribe via STOMP.

## Alternatives Considered

- **Server-Sent Events (SSE)**: Simpler, unidirectional. Sufficient for our use case (server → client only) but less ecosystem support in Flutter. No built-in topic/subscription model.
- **Firebase Realtime Database/Firestore**: Would bypass the backend entirely. Loses server-side validation and business logic. Creates two sources of truth.
- **Polling**: Simple but wastes bandwidth, adds latency (1-5 seconds), and doesn't scale well with many connected clients.
- **gRPC streaming**: Overkill for this use case. Adds Protobuf compilation step and complexity.

## Consequences

- Real-time updates across all connected devices
- STOMP provides topic-based pub/sub out of the box
- Simple in-memory broker works for single instance but doesn't work across multiple Cloud Run instances
- Will need external broker (Redis pub/sub or RabbitMQ) before horizontal scaling
- JWT authentication on WebSocket CONNECT frames
- Reconnection logic needed on frontend for network interruptions
