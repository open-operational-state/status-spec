# Capabilities / Negotiation

> **Status:** Conceptual definition — Phase 2 (not yet normative).

## Purpose

Capabilities describe what a target supports. Negotiation covers how a monitor and target determine the best interaction mode.

## Why This Layer Exists

As the ecosystem supports multiple profiles, serializations, and authentication modes, monitors need a way to understand what is available **before** attempting to consume a resource. Without this layer, extensibility becomes opaque or fragile — monitors would have to probe endpoints through trial and error.

---

## What Capabilities Describe

A capabilities declaration communicates:

| Aspect | Description |
|---|---|
| **Supported profiles** | Which profile(s) the target supports (Liveness, Readiness, Health, Status) |
| **Available serializations** | Which wire format(s) are available (health-response, service-status, etc.) |
| **Authentication modes** | What auth is required or available (none, API key, bearer token, mutual TLS, etc.) |
| **Evidence depth** | How much evidence the target provides (self-reported only, component-level, full dependency tree) |
| **Condition vocabularies** | Which condition vocabularies the target uses in its responses |

---

## Capabilities Document Concept

A capabilities document is a machine-readable description of what a specific operational-state resource (or set of resources) supports.

### Content

A capabilities document declares:

- The profile(s) available at each resource
- The serialization(s) each resource can produce
- Whether authentication is required and what modes are supported
- The evidence depth available
- The condition vocabulary in use

### Relationship to Discovery

Capabilities metadata may be:

- Embedded within a discovery document
- Available at a separate, dedicated capabilities endpoint
- Partially conveyed via HTTP headers on individual resources

The exact delivery mechanism will be specified in Phase 3, but the conceptual model supports all three approaches.

---

## Capabilities Delivery Mechanisms

### 1. Standalone Capabilities Document

A dedicated resource that describes the capabilities of one or more operational-state endpoints.

- Discoverable via the discovery layer
- Self-contained and cacheable
- Most suitable for targets with multiple operational-state resources

### 2. HTTP Headers

Capabilities information conveyed in HTTP response headers of individual operational-state resources.

- Lightweight — no separate request required
- Limited in expressiveness — only a subset of capabilities can be communicated via headers
- Suitable for simple cases (single profile, single serialization)

### 3. Embedded in Discovery

Capabilities metadata included within discovery documents or inline references.

- Eliminates the need for a separate capabilities request
- Combines "where" and "what" in a single document
- Must remain lightweight to avoid bloating discovery responses

---

## Capabilities Stability Contract

Capabilities metadata should be **relatively stable** and not vary per-request. This supports:

- **Caching** — monitors can cache capabilities and avoid redundant requests
- **Predictability** — monitoring behavior can rely on capabilities remaining consistent within a TTL
- **Reliable negotiation** — monitors can make interaction decisions based on cached capabilities

Capabilities may change when a target deploys new versions, adds new profiles, or changes authentication requirements — but these are infrequent events relative to operational-state polling frequency.

---

## Default Assumptions

**In the absence of explicit capabilities metadata, consumers must assume minimal support:**

- Single profile (the lowest applicable profile — typically Liveness)
- Single serialization (the format returned by the resource)
- No authentication required
- Minimal evidence depth (condition only, no components or dependencies)

This prevents monitors from guessing inconsistently. A target that wants to communicate richer capabilities must provide explicit capabilities metadata.

---

## Negotiation

Negotiation is the process by which a monitor and target determine the best interaction mode.

### Negotiation Approaches

| Approach | How it Works | Suitability |
|---|---|---|
| **Capabilities-first** | Monitor reads capabilities, then requests the appropriate resource | Best for complex targets with multiple options |
| **Content negotiation** | Monitor sends `Accept` header; target responds with the best match | Simple, HTTP-native, limited expressiveness |
| **Discovery-guided** | Monitor uses discovery to find the resource that matches its needs | Works when separate resources serve different profiles/serializations |
| **Direct** | Monitor requests a known resource and accepts whatever format is returned | Simplest, no negotiation overhead |

### v1 Recommendation

For v1, the recommended approach is **discovery-guided + separate resources**: different profiles and serializations are served at different URLs, and the discovery layer helps monitors find the right one. This is simpler and more practical than complex content negotiation.

Content negotiation may be supported as an additional mechanism but should not be the sole or primary negotiation path.

---

## Design Constraints

1. **Discoverable without trial and error** — monitors should be able to determine available interaction modes without probing every possible combination
2. **Simple for simple cases** — a target with a single health endpoint should not need to implement complex capabilities declaration
3. **Scalable for complex cases** — a target with multiple profiles, serializations, and auth modes should have a clean way to declare all of them
4. **Cacheable** — capabilities information should be safe to cache for reasonable periods

---

## References

- [ARCHITECTURE.md](../ARCHITECTURE.md) — Layer 6 definition
- [discovery.md](discovery.md) — the layer that locates resources (capabilities describes them)
- [profiles.md](profiles.md) — the profiles that capabilities declare
- [serializations.md](serializations.md) — the serializations that capabilities declare
