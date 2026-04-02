# Capabilities / Negotiation

> **Status:** Draft specification — Phase 3.

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

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
| **Available serializations** | Which wire format(s) are available (`application/health+json`, `application/status+json`) |
| **Authentication modes** | What auth is required or available (none, API key, bearer token, etc.) |
| **Evidence depth** | How much evidence the target provides (condition only, component-level, full dependency tree) |
| **Condition vocabularies** | Which condition vocabularies the target uses in its responses |

---

## Capabilities Delivery

Capabilities information MAY be communicated through three mechanisms:

### 1. Discovery Document (Primary)

The [discovery document](discovery.md#discovery-document-format) serves as the primary capabilities declaration. Each resource entry in the discovery document communicates profiles, serialization, and authentication requirements.

This is the RECOMMENDED approach for v1.

### 2. HTTP Headers

Individual operational-state responses MAY include capabilities information in HTTP headers:

```http
X-OOS-Profiles: health, readiness, liveness
X-OOS-Serialization: application/health+json
```

- Lightweight — no separate request required
- Limited in expressiveness
- Suitable for simple cases (single profile, single serialization)

**Header definitions:**

- `X-OOS-Profiles` — comma-separated list of profile identifiers. OPTIONAL.
- `X-OOS-Serialization` — content type of the response. Redundant with `Content-Type` but explicitly signals standard awareness. OPTIONAL.

### 3. Embedded in Responses

The `profiles` field already present in serialization responses serves as inline capabilities declaration. Any operational-state response that includes a `profiles` array communicates its capabilities directly.

---

## Default Assumptions

**In the absence of explicit capabilities metadata, consumers MUST assume minimal support:**

- Single profile (Liveness)
- Single serialization (whatever format the resource returns)
- No authentication required
- Minimal evidence depth (condition only, no components or dependencies)

These defaults prevent monitors from guessing inconsistently. A target that wants to communicate richer capabilities MUST provide explicit capabilities metadata via discovery documents or response headers.

---

## Negotiation

Negotiation is the process by which a monitor and target determine the best interaction mode.

### v1 Recommendation

For v1, the RECOMMENDED approach is **discovery-guided + separate resources**: different profiles and serializations are served at different URLs, and the discovery layer helps monitors find the right one.

| Approach | How It Works | v1 Recommendation |
|---|---|---|
| **Discovery-guided** | Monitor reads discovery document, selects the resource matching its needs | RECOMMENDED |
| **Direct** | Monitor requests a known resource and accepts whatever format is returned | Acceptable for simple cases |
| **Content negotiation** | Monitor sends `Accept` header; target responds with best match | MAY be supported, SHOULD NOT be sole mechanism |

### Content Negotiation

If a target supports content negotiation:

- The target SHOULD accept both `application/health+json` and `application/status+json` in the `Accept` header
- If no `Accept` header is provided, the target SHOULD respond with its default serialization
- If the requested content type is not available, the target SHOULD respond with 406 Not Acceptable

---

## Capabilities Stability

Capabilities metadata SHOULD be **relatively stable** and SHOULD NOT vary per-request. This supports:

- **Caching** — monitors MAY cache capabilities for reasonable periods
- **Predictability** — monitoring behavior can rely on capabilities remaining consistent within a TTL
- **Reliable negotiation** — monitors can make interaction decisions based on cached capabilities

Capabilities MAY change when a target deploys new versions, adds new profiles, or changes authentication requirements — but these are infrequent events relative to operational-state polling frequency.

---

## Design Constraints

1. **Discoverable without trial and error** — monitors SHOULD be able to determine available interaction modes without probing every possible combination
2. **Simple for simple cases** — a target with a single health endpoint MUST NOT need to implement complex capabilities declaration
3. **Scalable for complex cases** — a target with multiple profiles, serializations, and auth modes SHOULD have a clean way to declare all of them
4. **Cacheable** — capabilities information SHOULD be safe to cache for reasonable periods

---

## References

- [ARCHITECTURE.md](../ARCHITECTURE.md) — Layer 6 definition
- [discovery.md](discovery.md) — the layer that locates resources (capabilities describes them)
- [profiles.md](profiles.md) — the profiles that capabilities declare
- [serializations.md](serializations.md) — the serializations that capabilities declare
