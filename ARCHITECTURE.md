# Architecture

## Overview

The Open Operational State standard is built around a six-layer extensible architecture. This document defines those layers, their responsibilities, and how they relate to each other.

The architecture is designed to:

- Separate **meaning** from **wire representation**
- Support **multiple serializations** without forcing one universal body format
- Enable **incremental adoption** through adapters and profiles
- Allow **future extension** to non-web systems without breaking the core model
- Treat **discovery** and **capabilities** as first-class concerns

The six-layer model is considered **stable for v1** and should not be collapsed, reordered, or removed without a formal governance decision.

---

## The Six Layers

```
┌─────────────────────────────────────────┐
│  6. Capabilities / Negotiation          │
├─────────────────────────────────────────┤
│  5. Discovery                           │
├─────────────────────────────────────────┤
│  4. Adapters                            │
├─────────────────────────────────────────┤
│  3. Serializations                      │
├─────────────────────────────────────────┤
│  2. Profiles                            │
├─────────────────────────────────────────┤
│  1. Core Model                          │
└─────────────────────────────────────────┘
```

The layers are ordered from most fundamental (bottom) to most interaction-oriented (top). Lower layers are more stable; upper layers are more context-dependent.

---

## Layer 1: Core Model

### Responsibility

Provide stable, canonical semantics for operational-state meaning across all representations, profiles, and system classes.

The Core Model is the **normative semantic foundation** of the standard; all other layers must map to it.

### What It Defines

The core model is the transport-agnostic semantic center of the standard. It represents:

- **Subject** — the thing being described (a service, endpoint, component, dependency)
- **Condition** — normalized operational state at a point in time
- **Timing** — when the condition was observed, reported, or last changed
- **Evidence** — the basis on which the condition is asserted
- **Scope** — what part of the system the condition applies to
- **Dependencies / Components** — hierarchical or compositional relationships
- **Provenance** — how the condition was produced (self-reported, externally observed, derived, manually declared)

The core model represents a **snapshot of operational condition at a point in time**. Future extensions may model condition transitions or event streams, but v1 treats each observation as an independent snapshot.

A condition must be **internally consistent within a snapshot**. A single snapshot must not contain contradictory condition statements for the same subject at the same scope.

### Design Principles

- The core model is where **long-term semantic durability** lives
- It must be expressible across different serializations without losing meaning
- It must not be tied to a specific JSON structure, HTTP status code, or endpoint convention
- Changes to the core model are the most consequential and require the most scrutiny

### What It Does Not Define

- Wire-level field names or JSON structure (that is Serializations)
- Domain-specific field requirements (that is Profiles)
- How to find an endpoint (that is Discovery)
- How to translate a legacy format (that is Adapters)

---

## Layer 2: Profiles

### Responsibility

Specialize the core model for concrete use cases within web-service scope.

### What Profiles Do

Profiles narrow and contextualize the core model for specific operational scenarios. Profiles **must not redefine core semantics** — they select, constrain, and contextualize existing concepts.

They define:

- Which core model concepts are **required** vs. **optional** in context
- What **semantics** apply in that context
- What **valid interpretations and constraints** exist
- **Condition vocabularies** appropriate to the use case

### Expected v1 Profile Distinctions

For v1, profiles are expected to remain within web-service scope. They may distinguish among concepts such as:

- **Liveness** — is the service reachable and accepting connections?
- **Readiness** — is the service ready to handle requests?
- **Health** — what is the service's overall operational condition?
- **Status** — rich, structured information about the service's state and components

### Relationship to Core Model

Profiles are **specializations**, not replacements. Every profile maps back to core model concepts.

---

## Layer 3: Serializations

### Responsibility

Define wire-level representations of operational-state data, decoupled from semantic meaning.

### Why Serializations Are Separate

Meaning and wire shape are distinct concerns. The same operational condition can be represented as:

- A JSON object with one structure
- A JSON object with a different structure
- A non-JSON format
- A minimal HTTP response (status code + headers only)

The architecture must support **multiple normative serializations** rather than one forced universal body format.

### Key Structural Decision

The old health-check draft (`draft-inadarei-api-health-check`) and the newer service-status draft (`draft-dallariva-web-service-status-json`) cannot be collapsed into a single strict universal wire format without structural conflicts.

The correct response is:

- One core model
- Multiple normative serializations / profiles
- Clear mappings between them
- Internal normalization, external representation flexibility

A single hybrid payload that pretends to be fully compliant with both drafts at once is **not** the chosen direction.

### Key Constraint

A serialization must be **losslessly mappable** to the core model within its declared capability. If a serialization cannot represent a core model concept, it must declare that limitation explicitly rather than silently dropping or distorting information. This prevents ambiguous or inconsistent adapter behavior downstream.

A serialization **must not introduce new semantics** that cannot be represented in the core model. This prevents divergence between implementations and hidden meaning in wire formats.

### What Serializations Define

- Field names and structure for a given wire format
- Required vs. optional fields in that format
- Mapping from serialization fields back to core model concepts
- Content types and media type registration (where applicable)
- Declared limitations (which core model concepts are not representable)

---

## Layer 4: Adapters

### Responsibility

Bridge existing or foreign systems into the standard model.

### Why Adapters Exist

The real world is full of existing health, status, and readiness endpoints that predate this standard. Adapters are the mechanism by which the ecosystem can ingest:

- Existing framework health endpoints (e.g., Spring Boot Actuator, ASP.NET health checks, Rails health endpoints)
- Vendor-specific health/status formats
- Legacy conventions (plain HTTP status codes, simple text bodies)
- Platform-specific patterns (Kubernetes probes, cloud provider health checks)

### What Adapters Do

An adapter translates a native or legacy format into the core model and, optionally, into a specific profile.

Adapters define:

- Input format recognition (how to identify what is being adapted)
- Mapping rules (how to translate fields/concepts to the core model)
- Confidence or fidelity indicators (how complete or reliable the mapping is)
- Lossiness classification — the degree of information preservation:
  - *Lossless* — all source information maps cleanly to the core model
  - *Partially lossy* — some information cannot be represented and is documented as lost
  - *Heuristic* — mapping relies on inference or pattern matching
  - *Inferred* — core model fields are populated from indirect evidence
- Limitations (what the adapter cannot infer)

### Adapter Philosophy

Adapters are essential to the goal of **broad interpretability**. The standard should not require every target to rewrite its operational-state endpoints. Where possible, existing endpoints should be interpretable — even if imperfectly — through adapter logic.

---

## Layer 5: Discovery

### Responsibility

Allow monitors to locate available operational-state resources and determine what they represent.

### Why Discovery Is a First-Class Layer

Monitors should not have to guess where operational-state resources live or what they represent. Hard-coded path conventions (e.g., always checking `/.well-known/health`) are a starting point but not sufficient.

### What Discovery Covers

- **Resource location** — where operational-state endpoints exist
- **Resource relations** — how endpoints relate to each other (e.g., a shallow liveness check vs. a rich health endpoint)
- **Profile awareness** — which profile(s) a resource implements
- **Public vs. authenticated** — which resources require authentication
- **Depth indication** — which resources offer richer vs. shallower information

### Discovery Mechanisms

The standard should support:

- **Link-based discovery** — HTTP Link headers, link relations
- **Predictable resources** — well-known paths as a baseline
- **Optional DNS bootstrap** — TXT records for domain-level discovery
- **Inline references** — resources linking to related resources

### DNS Position

DNS may assist with discovery and domain ownership verification, but it is **not** the primary protocol surface. HTTP is normative for the resource itself.

Discovery **must not depend solely on DNS**. Any resource discoverable via DNS must also be discoverable through HTTP-based mechanisms. This prevents DNS from becoming a mandatory dependency for conformant implementations.

### Content Negotiation

Content negotiation may be supported, but it should not be the sole or primary discovery path. Separate resources are simpler and more practical initially.

---

### Boundary with Capabilities

Discovery and Capabilities are complementary but distinct: **Discovery identifies resources; Capabilities describe how those resources can be interacted with.** A discovery mechanism locates an endpoint; capabilities metadata describes what profiles, serializations, and auth modes that endpoint supports.

---

## Layer 6: Capabilities / Negotiation

### Responsibility

Communicate what a target supports and how a monitor can interact with it.

### Why Capabilities Are a Separate Layer

As the ecosystem supports multiple profiles, serializations, and auth modes, monitors need a way to understand what is available before attempting to consume a resource. Without this, extensibility becomes opaque or fragile.

### What Capabilities Cover

- Which **profile(s)** the target supports
- Which **serialization(s)** are available
- What **authentication modes** apply
- What **evidence depth** is available (e.g., self-reported only vs. component-level detail)
- What **condition vocabularies** the target uses

### Negotiation

Negotiation is the process by which a monitor and target determine the best interaction mode. This may happen through:

- Capabilities documents or endpoints
- HTTP headers
- Discovery metadata
- Content negotiation (where supported)

### Design Constraint

Capabilities should be discoverable without requiring a monitor to attempt every possible interaction mode through trial and error.

Capabilities metadata should be **relatively stable** and not vary per-request. This supports caching, predictable monitoring behavior, and reliable negotiation.

---

## Cross-Cutting Concerns

### Authentication and Security

Authentication is a first-class concern across all layers:

- **Public resources should exist where possible** — they lower adoption friction and enable basic interoperability
- **Authenticated resources should extend, not replace, public ones** — deeper detail behind auth, not hidden basics
- Discovery must indicate which resources require authentication
- Capabilities may describe available authentication modes
- The standard should work with realistic auth approaches (API keys, tokens, mutual TLS) rather than assuming all resources are public

### Evidence and Provenance

Condition information may originate from different sources. The architecture preserves room to represent provenance:

- **Self-reported** — the target claims its own condition
- **Externally observed** — a monitor observes the condition
- **Derived** — computed from other data
- **Manually declared** — set by a human operator

This provenance awareness belongs in the model but does **not** merge synthetic testing into the operational-state spec.

### Extensibility

The architecture is intentionally designed for extension:

- New **profiles** can be added for new domains or use cases
- New **serializations** can be defined for new wire formats
- New **adapters** can bridge new legacy or vendor formats
- New **discovery mechanisms** can be introduced
- The core model can gain new concepts without breaking existing profiles

v1 scopes narrowly (web services only). The architecture designs broadly (future system classes can fit).

---

## Relationship to Prior Art

This architecture is informed by, but not normatively derived from, two prior Internet-Drafts:

- **`draft-inadarei-api-health-check`** — *Health Check Response Format for HTTP APIs*. An expired individual Internet-Draft defining a JSON response format for health checks. Represents the "health-response lineage."

- **`draft-dallariva-web-service-status-json`** — *Service Status Resource Format for Web Services*. An Internet-Draft defining a status resource format. Represents the "service-status lineage."

These are treated as **influences and compatibility targets**, not as normative foundations. The architecture acknowledges both lineages and provides a framework for supporting both through separate serializations and profiles, rather than forcing a single hybrid format.

See [PRIOR-ART.md](PRIOR-ART.md) for detailed analysis.
