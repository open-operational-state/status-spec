# Profiles

> **Status:** Draft specification — Phase 3.

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## Purpose

Profiles specialize the core model for concrete use cases within web-service scope. They define which core model concepts are required, optional, or constrained in a given context.

Profiles are the mechanism by which the standard supports multiple levels of operational-state richness — from a minimal liveness signal to a comprehensive service status dashboard — without forcing every implementation to support the full model.

## What Profiles Do

Profiles **narrow and contextualize** the core model. They:

- **Select** which core model concepts are required vs. optional
- **Constrain** how those concepts are used (e.g., which condition vocabulary is valid)
- **Contextualize** the meaning of concepts within a specific operational scenario
- **Define** condition vocabularies appropriate to the use case (see [Condition Vocabularies](condition-vocabularies.md))

## What Profiles Must Not Do

Profiles **MUST NOT redefine core semantics**. The meaning of "Condition," "Subject," "Provenance," etc. is defined by the core model and is invariant across profiles.

Specifically, a profile MUST NOT:

- Change the definition of a core concept
- Introduce new semantic categories that have no basis in the core model
- Override core model invariants (e.g., snapshot consistency, DAG acyclicity)
- Require serialization-specific behaviors (profiles are representation-agnostic)

---

## Profile Composability

**A single resource MAY declare conformance to multiple profiles simultaneously. Profiles are not mutually exclusive.**

This decision is locked based on real-world observation:

- Kubernetes, Spring Boot, and other platforms blur the boundaries between liveness, readiness, and health
- Forcing exclusivity creates artificial fragmentation and increases the number of endpoints a target must expose
- Composability enables a single rich endpoint to satisfy multiple consumer needs

### Composability Rules

1. When a resource declares multiple profiles, it MUST satisfy the requirements of **all** declared profiles.
2. If two profiles define different condition vocabularies, the resource's condition value MUST be valid in **all** declared vocabularies, or the resource MUST provide profile-qualified condition values.
3. Profile declarations are additive — declaring an additional profile adds constraints, never removes them.

---

## Required and Optional Concepts Per Profile

The following table defines which core model concepts are required, recommended, or optional for each v1 profile. This table is **normative**.

| Concept | Liveness | Readiness | Health | Status |
|---|---|---|---|---|
| **Subject** | MUST | MUST | MUST | MUST |
| **Condition** | MUST | MUST | MUST | MUST |
| **Timing** | MAY | MAY | MUST | MUST |
| **Evidence** | — | — | SHOULD | MUST |
| **Components** | — | — | MAY | MUST |
| **Dependencies** | — | MAY | MAY | MUST |
| **Provenance** | — | — | SHOULD | MUST |
| **Scope** | — | — | MAY | MAY |

**Interpretation:**

- **MUST** — the concept MUST be present in a conformant response for this profile
- **SHOULD** — the concept SHOULD be present; implementations that omit it MUST document why
- **MAY** — the concept is optional; its absence carries no conformance implications
- **—** — the concept is not expected for this profile; its presence is permitted but has no profile-defined semantics

---

## v1 Profile Definitions

v1 defines four profiles, ordered from least to most comprehensive.

### Liveness

**Question answered:** _"Is this service reachable and accepting connections?"_

**Condition vocabulary:** [Liveness Vocabulary](condition-vocabularies.md#liveness-vocabulary) — `alive` / `unreachable` (orderable)

**Requirements:**

- A Liveness response MUST include Subject and Condition
- A Liveness response MUST use a value from the Liveness vocabulary
- A Liveness endpoint SHOULD be inexpensive to produce and safe to invoke frequently
- A Liveness endpoint SHOULD NOT fail due to downstream dependency failures — a service that is running, listening, and capable of processing requests is `alive` even if a backend database is unavailable

**Typical consumers:** Load balancers, service mesh liveness probes, basic uptime monitors.

### Readiness

**Question answered:** _"Is this service ready to handle requests?"_

**Condition vocabulary:** [Readiness Vocabulary](condition-vocabularies.md#readiness-vocabulary) — `ready` / `initializing` / `not-ready` (orderable)

**Requirements:**

- A Readiness response MUST include Subject and Condition
- A Readiness response MUST use a value from the Readiness vocabulary
- A Readiness response MAY include critical dependency conditions to explain a `not-ready` state

**Relationship to Liveness:** A service MAY be `alive` (Liveness) while simultaneously `not-ready` (Readiness). This is expected during startup, dependency outages, or draining scenarios.

**Typical consumers:** Orchestrator readiness gates, deployment rollout checks, traffic routing decisions.

### Health

**Question answered:** _"What is the service's overall operational condition?"_

**Condition vocabulary:** [Health Vocabulary](condition-vocabularies.md#health-vocabulary) — orderable (`operational` / `degraded` / `partial-outage` / `major-outage` / `down`) + categorical (`maintenance` / `unknown`)

**Requirements:**

- A Health response MUST include Subject, Condition, and Timing
- A Health response MUST use a value from the Health vocabulary (or a valid extension value)
- A Health response SHOULD include Evidence for non-operational conditions (degraded, partial-outage, major-outage, down)
- A Health response SHOULD include Provenance
- A Health response MAY include Component and Dependency conditions

**Aggregation guidance:** When a Health response includes component conditions, the parent subject's condition SHOULD be computed using **worst-of** aggregation across the orderable condition values of its components. Categorical values (e.g., `maintenance`) SHOULD be reported separately and MUST NOT participate in worst-of calculations. Implementations MAY use alternative aggregation strategies (majority quorum, weighted importance) but MUST document the strategy used.

**Typical consumers:** API health endpoints, monitoring dashboards, incident response tooling.

### Status

**Question answered:** _"What is the comprehensive operational state of this service and its entire dependency tree?"_

**Condition vocabulary:** [Status Vocabulary](condition-vocabularies.md#status-vocabulary) — inherits full Health vocabulary plus extension values

**Requirements:**

- A Status response MUST include Subject, Condition, Timing, Evidence, Components, Dependencies, and Provenance
- A Status response MUST use a value from the Health vocabulary (or a valid extension value)
- A Status response MUST include condition statements for all known components and dependencies
- A Status response MAY include Scope (regional/zonal) information

**Relationship to other profiles:** A Status response that satisfies all MUST requirements automatically provides enough information to satisfy Health, Readiness, and Liveness consumers. Implementations declaring the Status profile SHOULD also declare all subordinate profiles.

**Typical consumers:** Rich operational dashboards, status pages, incident investigation, capacity planning.

---

## Profile Hierarchy

The four v1 profiles form a **conceptual hierarchy** based on required concepts:

```
Status  ⊃  Health  ⊃  Readiness  ⊃  Liveness
```

Each profile is a superset of the one below it in terms of required concepts. This means:

- A resource conforming to the Health profile automatically provides enough information to satisfy Liveness and Readiness consumers
- A resource conforming to Status automatically satisfies all other profiles
- A resource MAY declare the most specific applicable profile(s), or all profiles it satisfies

This hierarchy does not mean the profiles are identical — they differ in **condition vocabularies**, **expected semantics**, and **consumer behavior**. A Liveness consumer SHOULD NOT be expected to parse component trees.

---

## Profile-to-Serialization Relationship

Profiles and serializations are **orthogonal concerns**:

- A profile defines **what** MUST be communicated
- A serialization defines **how** it is communicated on the wire

One profile MAY have **multiple serializations** (e.g., a Health profile MAY be expressed in both the health-response serialization and the service-status serialization). Each serialization MUST map all required profile concepts to wire-level fields.

A serialization that cannot represent a required concept for a given profile MUST NOT claim conformance to that profile.

---

## Profile Declaration

A resource MUST declare which profile(s) it implements. The mechanism for declaration is serialization-specific:

- In JSON serializations, a `profiles` field MUST be present containing an array of profile identifiers
- In HTTP headers, profile information MAY be conveyed via Link header parameters
- In discovery documents, profile information MUST be associated with each resource entry

Profile identifiers for v1 are: `liveness`, `readiness`, `health`, `status`.

---

## Profile Versioning

Profiles evolve independently of the core model and independently of each other.

- Profile versions are distinct from core model versions and serialization versions
- A profile version defines the set of required/optional concepts and condition vocabulary rules at that version
- Breaking changes to a profile (removing required concepts, changing condition semantics) MUST result in a new major version

---

## References

- [ARCHITECTURE.md](../ARCHITECTURE.md) — Layer 2 definition
- [core-model.md](core-model.md) — the model that profiles specialize
- [condition-vocabularies.md](condition-vocabularies.md) — condition values per profile
- [Glossary](https://github.com/open-operational-state/governance/blob/main/GLOSSARY.md) — authoritative terminology
- [Design Note 002: Profile Boundaries](../design-notes/002-profile-boundaries.md) — rationale for v1 profile distinctions
