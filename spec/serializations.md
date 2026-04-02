# Serializations

> **Status:** Conceptual definition — Phase 2 (not yet normative).

## Purpose

Serializations define wire-level representations of operational-state data, decoupled from semantic meaning. They specify how core model concepts are expressed in a concrete format that can be transmitted, parsed, and consumed.

## Key Principle

**Meaning and wire shape are distinct concerns.** The same operational condition can be represented in multiple ways — different JSON structures, different content types, different levels of detail. The architecture supports **multiple normative serializations** rather than one forced universal body format.

---

## What a Serialization Defines

Every serialization specification must include:

1. **Content type** — the media type identifying this serialization (e.g., `application/health+json`)
2. **Field mapping** — how each core model concept maps to wire-level fields, with exact field names and JSON structure
3. **Required vs. optional fields** — which fields must be present and which may be omitted
4. **Core model coverage** — which core model concepts this serialization can represent
5. **Declared limitations** — which core model concepts are **not** representable in this serialization, documented explicitly rather than silently dropped
6. **Lossless mapping proof** — a demonstration that the serialization can round-trip through the core model within its declared coverage without information loss

## Serialization Constraints

Two architectural constraints govern all serializations:

1. **Lossless within scope** — a serialization must be losslessly mappable to the core model within its declared capability. If a serialization cannot represent a core model concept, it must declare that limitation explicitly rather than silently dropping or distorting information.

2. **No semantic invention** — a serialization must not introduce new semantics that cannot be represented in the core model. If a wire format needs a concept that doesn't exist in the core model, the core model should be extended first. This prevents divergence between implementations and hidden meaning in wire formats.

---

## Dual-Lineage Strategy

The v1 standard defines two primary serializations, each informed by a prior-art Internet-Draft lineage. This reflects the locked architectural decision that the two prior drafts cannot be collapsed into a single universal wire format without structural conflicts.

### Health-Response Serialization

**Lineage:** Informed by `draft-inadarei-api-health-check-06` (_Health Check Response Format for HTTP APIs_).

**Content type direction:** `application/health+json` (or a namespace under the standard's authority).

**Characteristics:**

- Flat, single-resource-oriented structure
- Component-level detail via a `checks` object keyed by component name
- Status vocabulary: a small, binary-ish set (the prior draft uses `pass` / `fail` / `warn`)
- Observation-oriented: includes per-check timing, values, and units
- Links for related resources

**Core model mapping (conceptual — exact field names are Phase 3):**

| Core Model Concept | Mapping Direction |
|---|---|
| Subject | Represented by the resource itself (implicit) + service identifier |
| Condition | Top-level status field + per-component status |
| Timing | Per-component observation time |
| Evidence | Per-component observed values and units |
| Scope | Implicit: whole-service at top level, component-specific in checks |
| Components | Checks object with component-keyed entries |
| Dependencies | Within checks, components typed as external dependencies |
| Provenance | Not directly represented — implied as self-reported |

**Declared limitations:**

- No explicit provenance field (assumes self-reported)
- Limited scope expressiveness (no regional / zonal)
- No dependency vs. component semantic distinction in the wire format

### Service-Status Serialization

**Lineage:** Informed by `draft-dallariva-web-service-status-json-00` (_Service Status Resource Format for Web Services_).

**Content type direction:** `application/status+json` (or a namespace under the standard's authority).

**Characteristics:**

- Rich, multi-level structure
- Component-level status with criticality classification
- Geographic scope identification
- Structured incident reporting
- Richer metadata model

**Core model mapping (conceptual — exact field names are Phase 3):**

| Core Model Concept | Mapping Direction |
|---|---|
| Subject | Explicit service identification with metadata |
| Condition | Overall health indicators + per-component status |
| Timing | Report timestamps + per-component observation times |
| Evidence | Incident details, component-level observations |
| Scope | Geographic scope, regional identification |
| Components | Component list with criticality classification |
| Dependencies | Potentially within components or as a separate concept |
| Provenance | Not directly represented as a first-class field |

**Declared limitations:**

- No explicit provenance field
- More opinionated structure that may constrain certain use cases

---

## Minimal Serialization

The core model supports the concept of operationally useful state communicated with **no body at all**.

### HTTP-Status-Code-Only

A minimal "serialization" may consist of:

- HTTP response status code (e.g., 200 = operational, 503 = unavailable)
- Optional HTTP headers conveying timing or profile information

**Core model mapping:**

| Core Model Concept | Mapping Direction |
|---|---|
| Subject | Implicit: the resource being requested |
| Condition | Derived from HTTP status code |
| Timing | HTTP `Date` header if present |
| All other concepts | Not representable |

**Declared limitations:** A minimal serialization conveys only a coarse-grained condition and cannot represent full operational state. It is the most lossy "serialization" — it can represent only Subject and Condition at a binary level. It is useful as an adapter target for existing HTTP health checks but should not be considered a conformant serialization for profiles beyond Liveness. Consumers must not treat a minimal serialization as equivalent to a richer one that happens to return the same condition.

---

## Non-JSON Serializations

**v1 decision: non-JSON serializations are out of scope for v1.** The architecture supports future addition of non-JSON serializations (XML, Protobuf, CBOR, etc.), but v1 focuses exclusively on JSON-based wire formats.

The core model is deliberately transport-agnostic to ensure this extension remains possible without breaking changes.

---

## Serialization Selection

When a target supports multiple serializations, the consumer needs a way to select the appropriate one. This intersects with the Capabilities and Discovery layers:

- **Separate resources** — different serializations served at different URLs (simplest, recommended as the primary approach for v1)
- **Content negotiation** — HTTP `Accept` header to select serialization at a single URL (may be supported but should not be the sole mechanism)

---

## References

- [ARCHITECTURE.md](../ARCHITECTURE.md) — Layer 3 definition
- [PRIOR-ART.md](../PRIOR-ART.md) — the drafts that inform serialization design
- [core-model.md](core-model.md) — the model that serializations represent
- [Design Note 001: Dual Serialization Strategy](../design-notes/001-dual-serialization-strategy.md) — rationale for two serializations
