# Design Note 005: Snapshot vs. Stream

## Decision

v1 uses **snapshot semantics exclusively**. Event streams and transition modeling are deferred to future versions.

## Context

Operational state can be modeled in two fundamentally different ways:

| Model | Description |
|---|---|
| **Snapshot** | A point-in-time observation of current state. Each observation is independent. |
| **Stream** | A sequence of state transitions over time. Each event represents a change. |

Both models are valuable. The question is which to start with.

## Why Snapshots First

### Simplicity

Snapshots are conceptually simpler:

- No ordering requirements between observations
- No need to handle missed events or replay
- No connection state or subscription management
- Each response is complete and self-contained

### Compatibility

The vast majority of existing health endpoints use a request/response pattern that produces snapshot-like output:

- HTTP health checks return current state
- Kubernetes probes poll and receive point-in-time responses
- Status pages display current condition, not event history

Starting with snapshots means the standard can immediately interpret and normalize existing endpoints through adapters.

### Protocol Fit

v1 is centered on HTTP (request/response). Snapshots map naturally to HTTP semantics:

- A GET request returns a snapshot
- Caching and freshness (RFC 7234) apply naturally
- No persistent connection required

Stream semantics would require:

- Server-Sent Events, WebSockets, or similar persistent protocols
- Connection management, reconnection, and event ordering
- A fundamentally different consumption model

### Incremental Adoption

Organizations adopting the standard can start with snapshot endpoints and later add stream capabilities without redesigning their initial implementation.

## Future Stream Path

The architecture is deliberately designed to support future stream extensions:

- The core model concepts (Subject, Condition, Timing, Provenance, etc.) work in both models
- A future "event" layer could define condition transitions as events referencing core model concepts
- Snapshot semantics would remain valid — a stream is conceptually a sequence of snapshots with transition metadata

Possible future stream approaches:

- Server-Sent Events (SSE) for push-based condition updates
- WebSocket channels for bidirectional state negotiation
- Webhook callbacks for condition change notifications

These are **not part of v1** but the architecture does not preclude them.

## References

- [core-model.md: Snapshot Semantics](../../spec/core-model.md#snapshot-semantics)
- [ARCHITECTURE.md](../../ARCHITECTURE.md)
