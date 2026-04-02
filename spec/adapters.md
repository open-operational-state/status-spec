# Adapters

> **Status:** Draft specification — Phase 3.

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## Purpose

Adapters translate native or legacy target formats into the core model and profiles. They enable the ecosystem to interpret existing operational-state endpoints without requiring targets to change their implementations.

## Why Adapters Matter

The real world is full of existing health, status, and readiness endpoints. Broad interpretability requires the ability to ingest these existing formats — even imperfectly — through adapter logic. The standard should not require every target to rewrite its operational-state endpoints to be useful.

Adapters are what make the standard **practically relevant from day one**, because they allow monitors to interpret existing endpoints using core model semantics immediately.

---

## Adapter Specification Structure

Each adapter specification is a **structured prose document with mapping tables**. Adapter specifications are human-readable and mechanically translatable — they define the mapping precisely enough that a developer can implement the adapter from the spec alone, but they do not contain executable logic or pseudo-code.

An adapter specification must include the following sections:

### 1. Input Pattern Definition

How to identify that a given response matches this adapter's input format.

- **Content type** — expected media type(s) (e.g., `application/health+json`, `application/json`)
- **Structural markers** — distinctive fields, structures, or patterns that identify the format (e.g., presence of a `status` field with value `pass`/`fail`/`warn`)
- **Source context** — where this format is typically encountered (e.g., Spring Boot Actuator, Kubernetes probes)
- **Disambiguation** — how to distinguish this format from similar formats that a different adapter handles

### 2. Mapping Table

A tabular mapping from source format fields to core model concepts.

Format:

| Source Field / Pattern | Core Model Concept | Mapping Notes |
|---|---|---|
| _(source field name or pattern)_ | _(core model concept)_ | _(transformation rules, assumptions, edge cases)_ |

The mapping table must cover:

- Every core model concept, indicating whether it is mapped, unmapped, or inferred
- How values are transformed (e.g., `"pass"` → orderable condition value indicating operational)
- What assumptions are made when source data is ambiguous

### 3. Lossiness Classification

How much information is preserved in the adapter translation, using the following classification:

| Classification | Definition |
|---|---|
| **Lossless** | All source information maps cleanly to the core model. The original can be reconstructed from the core model representation. |
| **Partially lossy** | Some source information cannot be represented in the core model and is documented as lost. The core model representation is semantically correct but incomplete. |
| **Heuristic** | The mapping relies on pattern-based matching. The adapter recognizes structural patterns in the source and maps them to core model concepts based on observed conventions, not explicit declarations. Example: inferring "component" from a nested key name in a JSON health response. |
| **Inferred** | Core model fields are derived from indirect signals that were not intended to convey that meaning. The source does not contain the information, but the adapter derives it from context. Example: inferring provenance type from whether the response came from a service's own endpoint vs. an external monitor. |

A single adapter may have different lossiness classifications for different core model concepts. For example, Subject mapping may be lossless while Provenance mapping is inferred.

### 4. Limitations

Explicit documentation of what the adapter **cannot** reliably determine from the source format:

- Core model concepts that are not representable
- Ambiguous mappings where multiple interpretations are possible
- Edge cases where the mapping may produce incorrect results
- Conditions under which the adapter should decline to produce output

---

## Adapter-to-Profile Relationship

Adapters produce **core model output**, not profile-specific output. However, the output may be sufficient to satisfy one or more profiles depending on how much information the source format provides.

An adapter specification should indicate which profiles its output can typically satisfy:

| Profile | Satisfiable? | Notes |
|---|---|---|
| Liveness | Yes / No / Partial | _(explanation)_ |
| Readiness | Yes / No / Partial | _(explanation)_ |
| Health | Yes / No / Partial | _(explanation)_ |
| Status | Yes / No / Partial | _(explanation)_ |

This allows consumers to know what level of operational insight they can expect from adapted output.

---

## v1 Adapter Targets

The following adapter targets are prioritized for v1 specification. They are ordered by ubiquity and practical impact:

### Priority 1 — Plain HTTP Status Code

**Input:** HTTP response with status code only (no body, or body not conforming to any known format).

This is the most common "health check" pattern in the wild — a 200 means healthy, a 503 means unhealthy. Despite its simplicity, this adapter is essential because it is the universal fallback.

- **Lossiness:** Heuristic / Inferred for most concepts
- **Profile coverage:** Liveness only

### Priority 2 — Health-Check Draft Responses

**Input:** JSON responses conforming to `draft-inadarei-api-health-check` format (`application/health+json`).

This serves as a **meta-adapter**: translating the prior-art format into core model semantics. It also validates the dual-lineage architectural decision by demonstrating that the prior art can be losslessly mapped.

- **Lossiness:** Partially lossy (provenance is not explicitly present)
- **Profile coverage:** Liveness, Readiness, Health

### Priority 3 — Spring Boot Actuator

**Input:** JSON responses from Spring Boot Actuator `/actuator/health` endpoints.

Spring Boot Actuator is one of the most widely deployed structured health endpoints. Its format is close to the health-check draft but has framework-specific conventions (status values: `UP`, `DOWN`, `OUT_OF_SERVICE`, `UNKNOWN`; component details structure).

- **Lossiness:** Partially lossy
- **Profile coverage:** Liveness, Readiness, Health

### Priority 4 — Kubernetes Probe Conventions

**Input:** HTTP responses consumed by Kubernetes liveness, readiness, and startup probes.

Kubernetes probes typically consume HTTP status codes (200 = pass, non-200 = fail) with optional body content. The adapter must understand the Kubernetes probe context (which probe type informs which profile).

- **Lossiness:** Heuristic (Kubernetes probes do not mandate a body format)
- **Profile coverage:** Liveness, Readiness

---

## Adapter Registry Concept

As the ecosystem grows, the set of available adapters will expand. The conceptual model for adapter management:

- **Registered adapters** — adapters whose specifications are maintained within the `status-spec` or `status-conformance` repositories
- **Community adapters** — adapter specifications contributed by the community but not officially maintained
- **Implementation-specific adapters** — adapters that exist only within specific tooling implementations

The exact registry mechanism (how adapters are listed, discovered, and versioned) is deferred to later phases. The conceptual model ensures that the architecture supports a growing adapter ecosystem.

---

## Adapter Development Guidance

When creating a new adapter specification:

1. Start with the input pattern definition — can you reliably identify this format?
2. Map each core model concept — work through the mapping table systematically
3. Be honest about lossiness — partially lossy or heuristic is perfectly acceptable
4. Document limitations explicitly — consumers need to know what they cannot trust
5. Indicate profile coverage — which profiles can the adapted output satisfy?
6. Provide examples — at least one complete input → core model output example

---

## References

- [ARCHITECTURE.md](../ARCHITECTURE.md) — Layer 4 definition
- [core-model.md](core-model.md) — the model that adapters target
- [Conformance adapters](https://github.com/open-operational-state/status-conformance/blob/main/adapters/) — adapter validation in conformance repo
- [Glossary](https://github.com/open-operational-state/governance/blob/main/GLOSSARY.md) — authoritative terminology
