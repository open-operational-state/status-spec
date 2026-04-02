# Profiles

> **Status:** Conceptual definition — Phase 2 (not yet normative).

## Purpose

Profiles specialize the core model for concrete use cases within web-service scope. They define which core model concepts are required, optional, or constrained in a given context.

Profiles are the mechanism by which the standard supports multiple levels of operational-state richness — from a minimal liveness signal to a comprehensive service status dashboard — without forcing every implementation to support the full model.

## What Profiles Do

Profiles **narrow and contextualize** the core model. They:

- **Select** which core model concepts are required vs. optional
- **Constrain** how those concepts are used (e.g., which condition vocabulary is valid)
- **Contextualize** the meaning of concepts within a specific operational scenario
- **Define** condition vocabularies appropriate to the use case

## What Profiles Must Not Do

Profiles **must not redefine core semantics**. The meaning of "Condition," "Subject," "Provenance," etc. is defined by the core model and is invariant across profiles.

Specifically, a profile must not:

- Change the definition of a core concept
- Introduce new semantic categories that have no basis in the core model
- Override core model invariants (e.g., snapshot consistency, DAG acyclicity)
- Require serialization-specific behaviors (profiles are representation-agnostic)

---

## Profile Composability

**A single resource may declare conformance to multiple profiles simultaneously. Profiles are not mutually exclusive.**

This decision is locked based on real-world observation:

- Kubernetes, Spring Boot, and other platforms blur the boundaries between liveness, readiness, and health
- Forcing exclusivity creates artificial fragmentation and increases the number of endpoints a target must expose
- Composability enables a single rich endpoint to satisfy multiple consumer needs

### Composability Rules

1. When a resource declares multiple profiles, it must satisfy the requirements of **all** declared profiles.
2. If two profiles define different condition vocabularies, the resource's condition value must be valid in **all** declared vocabularies, or the resource must provide profile-qualified condition values.
3. Profile declarations are additive — declaring an additional profile adds constraints, never removes them.

---

## v1 Profile Definitions

v1 defines four profiles, ordered from least to most comprehensive. Each profile maps to a real-world use case and a specific level of operational-state richness.

### Liveness

**Question answered:** _"Is this service reachable and accepting connections?"_

| Aspect | Specification |
|---|---|
| **Required concepts** | Subject, Condition |
| **Optional concepts** | Timing |
| **Typical depth** | Minimal — no components, no dependencies |
| **Use case** | Load balancer health checks, service mesh liveness probes |
| **Condition vocabulary direction** | Binary (reachable / unreachable), orderable |

Liveness is the shallowest profile. It provides a fast, binary signal suitable for infrastructure-level routing decisions. Liveness checks should be inexpensive to produce and safe to invoke frequently.

A liveness endpoint should **not** fail due to downstream dependency failures. A service that is running, listening, and capable of processing requests is "live" even if a backend database is down. Downstream health belongs in Readiness or Health profiles.

### Readiness

**Question answered:** _"Is this service ready to handle requests?"_

| Aspect | Specification |
|---|---|
| **Required concepts** | Subject, Condition |
| **Optional concepts** | Timing, Dependencies |
| **Typical depth** | Shallow — may include critical dependency status |
| **Use case** | Orchestrator readiness gates, deployment rollout checks |
| **Condition vocabulary direction** | Binary or ternary (ready / not-ready / initializing), orderable |

Readiness extends liveness with awareness of whether the service can usefully process requests — not just that it's running, but that its critical dependencies are available and initialization is complete.

Readiness differs from Liveness in one key way: a service may be live (reachable) but not ready (still warming caches, waiting for config, or experiencing a dependency outage).

### Health

**Question answered:** _"What is the service's overall operational condition?"_

| Aspect | Specification |
|---|---|
| **Required concepts** | Subject, Condition, Timing |
| **Optional concepts** | Evidence, Components, Provenance |
| **Typical depth** | Medium — may include component-level detail |
| **Use case** | API health endpoints, monitoring dashboards, incident response |
| **Condition vocabulary direction** | Multi-valued (operational / degraded / partial outage / major outage / down), orderable |

Health is the most common profile for machine-readable operational state. It provides enough detail for monitoring systems to make informed alerting and routing decisions.

Health differs from Readiness in that it provides **ongoing operational insight**, not just a deployment-time gate. A service may be "ready" but "degraded" due to elevated error rates.

### Status

**Question answered:** _"What is the comprehensive operational state of this service and its entire dependency tree?"_

| Aspect | Specification |
|---|---|
| **Required concepts** | Subject, Condition, Timing, Evidence, Components, Dependencies, Provenance |
| **Optional concepts** | Scope (regional/zonal) |
| **Typical depth** | Full — complete component and dependency tree |
| **Use case** | Rich operational dashboards, status pages, incident investigation |
| **Condition vocabulary direction** | Rich multi-valued, may include both orderable and categorical values |

Status is the most comprehensive profile. It provides a complete picture of a service's operational state, including all components, dependencies, evidence, and provenance.

Status endpoints are expected to be more expensive to produce and may require authentication. They are not intended for high-frequency infrastructure polling.

---

## Profile Hierarchy

The four v1 profiles form a **conceptual hierarchy** based on required concepts:

```
Status  ⊃  Health  ⊃  Readiness  ⊃  Liveness
```

Each profile is a superset of the one below it in terms of required concepts. This means:

- A resource conforming to the Health profile automatically provides enough information to satisfy Liveness and Readiness consumers
- A resource conforming to Status automatically satisfies all other profiles
- A resource may declare the most specific applicable profile(s), or all profiles it satisfies

This hierarchy does not mean the profiles are identical — they differ in **expected semantics**, **condition vocabularies**, and **consumer behavior**. A Liveness consumer should not be expected to parse component trees.

---

## Profile-to-Serialization Relationship

Profiles and serializations are **orthogonal concerns**:

- A profile defines **what** must be communicated
- A serialization defines **how** it is communicated on the wire

One profile may have **multiple serializations** (e.g., a Health profile may be expressed in both the health-response serialization and the service-status serialization). Each serialization must map all required profile concepts to wire-level fields.

A serialization that cannot represent a required concept for a given profile **cannot** claim conformance to that profile.

---

## Profile Versioning

Profiles evolve independently of the core model and independently of each other.

- Profile versions are distinct from core model versions and serialization versions
- A profile version defines the set of required/optional concepts and condition vocabulary rules at that version
- Breaking changes to a profile (removing required concepts, changing condition semantics) require a new major version

---

## References

- [ARCHITECTURE.md](../ARCHITECTURE.md) — Layer 2 definition
- [core-model.md](core-model.md) — the model that profiles specialize
- [Glossary](https://github.com/open-operational-state/governance/blob/main/GLOSSARY.md) — authoritative terminology
- [Design Note 002: Profile Boundaries](../design-notes/002-profile-boundaries.md) — rationale for v1 profile distinctions
