# Serializations

> **Status:** Draft specification — Phase 3.

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## Purpose

Serializations define wire-level representations of operational-state data, decoupled from semantic meaning. They specify how core model concepts are expressed in a concrete format that can be transmitted, parsed, and consumed.

## Key Principle

**Meaning and wire shape are distinct concerns.** The same operational condition can be represented in multiple ways — different JSON structures, different content types, different levels of detail. The architecture supports **multiple normative serializations** rather than one forced universal body format.

---

## Serialization Requirements

Every serialization specification MUST include:

1. **Content type** — the media type identifying this serialization
2. **Field mapping** — how each core model concept maps to wire-level fields, with exact field names and JSON structure
3. **Required vs. optional fields** — which fields MUST be present and which MAY be omitted
4. **Core model coverage** — which core model concepts this serialization can represent
5. **Declared limitations** — which core model concepts are **not** representable in this serialization, documented explicitly rather than silently dropped
6. **Lossless mapping proof** — a demonstration that the serialization can round-trip through the core model within its declared coverage without information loss

## Serialization Constraints

Two architectural constraints govern all serializations:

1. **Lossless within scope** — a serialization MUST be losslessly mappable to the core model within its declared capability. If a serialization cannot represent a core model concept, it MUST declare that limitation explicitly rather than silently dropping or distorting information.

2. **No semantic invention** — a serialization MUST NOT introduce new semantics that cannot be represented in the core model. If a wire format needs a concept that doesn't exist in the core model, the core model SHOULD be extended first. This prevents divergence between implementations and hidden meaning in wire formats.

## Field Name Consistency

All serializations MUST use consistent field names for the same core model concepts:

- The condition concept MUST be expressed as `condition` (not `status`, `state`, or `health`)
- The subject concept MUST be expressed as `subject` (not `service`, `resource`, or `target`)
- Profile declarations MUST use the field name `profiles`

This ensures that consumers can rely on consistent naming across serializations, even when the overall document structure differs.

---

## v1 Serializations

v1 defines three serializations:

| Serialization | Content Type | Profile Coverage | Specification |
|---|---|---|---|
| **Health Response** | `application/health+json` | Liveness, Readiness, Health | [serializations/health-response.md](serializations/health-response.md) |
| **Service Status** | `application/status+json` | All profiles (optimized for Status) | [serializations/service-status.md](serializations/service-status.md) |
| **HTTP Status Only** | *(no body)* | Liveness only | [serializations/http-status-only.md](serializations/http-status-only.md) |

---

## Non-JSON Serializations

**v1 decision: non-JSON serializations are out of scope for v1.** The architecture supports future addition of non-JSON serializations (XML, Protobuf, CBOR, etc.), but v1 focuses exclusively on JSON-based wire formats.

The core model is deliberately transport-agnostic to ensure this extension remains possible without breaking changes.

---

## Serialization Selection

When a target supports multiple serializations, the consumer needs a way to select the appropriate one. This intersects with the Capabilities and Discovery layers:

- **Separate resources** — different serializations served at different URLs (simplest, RECOMMENDED as the primary approach for v1)
- **Content negotiation** — HTTP `Accept` header to select serialization at a single URL (MAY be supported but SHOULD NOT be the sole mechanism)

---

## References

- [ARCHITECTURE.md](../ARCHITECTURE.md) — Layer 3 definition
- [PRIOR-ART.md](../PRIOR-ART.md) — the drafts that inform serialization design
- [core-model.md](core-model.md) — the model that serializations represent
- [Design Note 001: Dual Serialization Strategy](../design-notes/001-dual-serialization-strategy.md) — rationale for two serializations
