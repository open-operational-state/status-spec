# Core Model

> **Status:** Conceptual definition — Phase 2 (not yet normative).

## Purpose

The core model is the stable, transport-agnostic semantic center of the Open Operational State standard. It defines the canonical concepts that all profiles and serializations reference.

The core model is where **long-term semantic durability** lives. Wire formats and serializations may evolve, but the meaning of operational-state concepts defined here should remain durable across versions and representations.

## Scope

The core model defines **what operational state means**, independent of:

- how it is represented on the wire (that is Serializations)
- what domain-specific constraints apply (that is Profiles)
- how endpoints are located (that is Discovery)
- how legacy formats are translated (that is Adapters)

The core model is the **normative semantic foundation** of the standard; all other layers must map to it.

---

## Snapshot Semantics

The core model represents a **snapshot of operational condition at a point in time**. Each observation produces an independent, immutable snapshot.

### Invariants

1. **Immutability** — once a snapshot is produced, its content does not change. Subsequent observations produce new snapshots.
2. **Internal consistency** — a single snapshot must not contain contradictory condition statements for the same subject at the same scope.
3. **Point-in-time** — a snapshot describes the state of the world as understood at a specific moment. It does not represent a time range or a transition.
4. **Independence** — snapshots are independent of each other. The core model does not define ordering, diffing, or transition semantics between snapshots. Future extensions may model condition transitions or event streams, but v1 treats each snapshot as standalone.

### What a Snapshot Contains

A snapshot contains one or more **condition statements** — each binding a subject, at a scope, to a condition, with associated timing, evidence, and provenance. A snapshot may also contain component and dependency relationships between subjects.

---

## Core Concepts

The core model defines seven canonical concepts. Each concept has a precise semantic role that must be preserved across all serializations and profiles.

### 1. Subject

**Definition:** The entity whose operational state is being described.

A subject is the "thing" about which a condition statement is made. In v1 scope, subjects are web services, APIs, websites, web applications, or their constituent parts.

#### Identity

A subject must be **uniquely identifiable** within a snapshot and **stably identifiable** across snapshots.

- **Within a snapshot:** no two condition statements may refer to the same subject at the same scope without being contradictory. Subject identity must be sufficient to distinguish one subject from another.
- **Across snapshots:** a consumer must be able to determine whether two subjects in different snapshots refer to the same entity. This enables comparisons, trending, and change detection over time. Subject identity must be comparable across snapshots to enable longitudinal analysis. The standard does not define how identity is maintained over time (e.g., via registry, convention, or stable naming), but the identity value itself must be stable enough that two snapshots referring to the same entity use the same identity.

The core model defines that subjects have an **identity** but does not mandate a specific identifier format (URI, UUID, string, etc.). Serializations define how identity is expressed on the wire. Profiles may constrain what types of identifiers are valid.

#### Subject Types

The core model recognizes conceptual distinctions among subjects, but does not enforce a rigid taxonomy:

- **Service** — a top-level web service or API
- **Component** — a constituent part of a service (see [Components and Dependencies](#5-components-and-dependencies))
- **Dependency** — an external system that a service relies on (see [Components and Dependencies](#5-components-and-dependencies))

These distinctions are contextual. An entity that is a "dependency" from one service's perspective may be a "service" from its own.

---

### 2. Condition

**Definition:** The normalized operational state of a subject at a point in time.

Condition is the semantic core of any operational-state statement. It answers the question: _"What is the state of this thing right now?"_

#### Condition Values and Vocabularies

Condition values are drawn from a **condition vocabulary** — a defined set of allowed values. The core model defines the **structure** of condition vocabularies, not the specific values themselves.

Structural requirements for condition vocabularies:

- **Profile-scoped** — each profile defines its own condition vocabulary appropriate to its use case. The core model does not mandate a single universal vocabulary.
- **Extensible** — vocabularies must support extension without breaking existing consumers. The mechanism for extension (namespace-qualified values, URI-based values, registered extensions, etc.) will be defined in Phase 3.
- **Enumerable** — condition values must come from a known, finite set within a given vocabulary. Free-form strings are not valid condition values.

#### Comparability Model

Condition values within a vocabulary may be **orderable** or **categorical**:

- **Orderable** — values have a defined severity or health ordering (e.g., `operational` > `degraded` > `down`). This supports alerting thresholds, aggregation, and worst-of calculations.
- **Categorical** — values represent distinct states without inherent ordering (e.g., `maintenance`, `migrating`, `provisioning`).

A condition vocabulary must declare which model it uses. A single vocabulary may contain both orderable and categorical values, provided the orderable subset is clearly identified.

The comparability model has downstream implications for:

- **Alerting** — can a monitor compare current condition to a threshold?
- **Aggregation** — can a service's overall condition be derived from component conditions?
- **UI** — can conditions be sorted or ranked?
- **Adapters** — can a foreign format's values be mapped to an ordered scale?

#### Condition Uniqueness

**For a given Subject and Scope within a snapshot, exactly one Condition must be defined.** If a subject has different conditions at different scopes, those are separate condition statements. If a subject has multiple condition sources (e.g., self-reported and externally observed), those are distinguished by provenance, not by duplicating the condition. This constraint prevents conflicting adapter outputs and ambiguous normalization.

---

### 3. Timing

**Definition:** Temporal metadata associated with a condition statement.

Timing records **when** something happened. The core model distinguishes three temporal concepts:

- **Observation time** — when the condition was observed or measured. This is the moment the check was performed.
- **Report time** — when the condition was communicated or made available. This may differ from observation time if there is caching or delayed publication.
- **State-change time** — when the condition last transitioned to its current value. This enables duration calculations ("has been degraded for 45 minutes").

#### Requirements

- Observation time is the most fundamental timing concept and should be present when known.
- Report time and state-change time are supplementary. Not all systems can provide them.
- All timestamps use a common representation (ISO 8601 / RFC 3339). Serializations define the exact wire format.
- Profiles may require or make optional specific timing fields based on the use case.

---

### 4. Evidence

**Definition:** The basis on which a condition is asserted.

Evidence describes **why** a condition has a particular value — what checks, observations, measurements, or inputs informed the condition statement.

#### Role

Evidence increases the **interpretability** of a condition. A bare condition ("degraded") is less useful than a condition with evidence ("degraded because database response time exceeds 500ms").

#### Structure

The core model defines evidence as having:

- **Type** — the kind of check or observation (e.g., connectivity test, response time measurement, error rate threshold, manual assessment)
- **Detail** — supplementary information about the check result (e.g., observed value, threshold, error message)

Evidence is **optional** at the core model level. Profiles may require evidence for certain condition values (e.g., a Health profile may require evidence for degraded or failed conditions).

#### Relationship to Provenance

Evidence describes the basis for a condition; provenance describes how that evidence was obtained and by whom. A self-reported condition may have rich evidence (the service checked its own database). An externally observed condition may have different evidence (a monitor measured response time). These are complementary, not redundant. Maintaining this distinction now will prevent conflation in serializations and adapters later.

---

### 5. Components and Dependencies

**Definition:** Structural relationships between subjects within a snapshot.

A service may report condition not only for itself, but for its constituent parts (components) and the external systems it relies on (dependencies).

#### Distinction

- **Component** — an internal constituent part of the subject. The component exists within the subject's operational boundary. Examples: a microservice within an application, a database shard, a worker pool.
- **Dependency** — an external system or service that the subject relies on but does not own or operate. Examples: a third-party API, a cloud provider service, a DNS provider.

This distinction is **contextual and relative** — a dependency from one subject's perspective may be a service with its own components from another perspective.

#### Graph Structure

The component/dependency model forms a **directed acyclic graph (DAG)**.

**Invariants:**

1. **Acyclicity** — cyclic dependencies are not permitted within a single snapshot. If A depends on B, B must not depend on A (directly or transitively) within the same snapshot.
2. **Direction** — edges point from the subject to its components or dependencies.
3. **No mandatory depth** — the core model does not mandate a minimum or maximum depth. A snapshot may report only a top-level service with no components, or it may report arbitrarily deep component trees.

#### Component and Dependency Conditions

Each component or dependency is itself a **subject** with its own condition, timing, evidence, and provenance. This enables recursive condition modeling — a service can be "degraded" because one of its components is "down."

Profiles may define rules for how component and dependency conditions relate to the parent subject's condition (e.g., worst-of aggregation, majority quorum, or explicit override).

---

### 6. Provenance

**Definition:** How a condition was produced — the origin or method by which the condition statement was generated.

Provenance answers the question: _"Who says this is the condition?"_

#### Provenance Types

The core model defines four provenance types:

| Type | Description | Example |
|---|---|---|
| **Self-reported** | The target asserts its own condition. The entity being described produced the condition statement. | A web service's `/health` endpoint returns `{"status": "pass"}`. |
| **Externally observed** | A monitor or external system determined the condition based on its own observations. | An uptime monitor reports the service as unreachable after probe failures. |
| **Derived** | The condition is computed from other data — aggregation, inference, or transformation of other condition statements or metrics. | A dashboard computes "degraded" from component-level conditions using worst-of aggregation. |
| **Manually declared** | A human operator has explicitly set the condition, overriding automated sources. | An operator marks a service as "maintenance" during a planned deployment. |

#### Semantics

- Provenance is **per-condition-statement**, not per-snapshot. A single snapshot may contain conditions with different provenances (e.g., a self-reported service condition alongside externally observed dependency conditions).
- Multiple provenances may exist for the same subject/scope if they come from different sources. Conflict resolution rules are not defined by the core model but may be addressed by profiles or consumer implementations.
- Provenance does not imply **trust**. A self-reported condition is not inherently more or less trustworthy than an externally observed one. Trust and confidence are consumer-level concerns, not core model semantics.

---

### 7. Scope

**Definition:** What part of the system a condition statement applies to.

Scope delineates the **boundary** of a condition's applicability.

#### Usage

A condition statement always applies at a scope. Scope may be:

- **Whole-service** — the condition applies to the entire service
- **Component-specific** — the condition applies to a specific component
- **Dependency-specific** — the condition applies to a specific dependency
- **Endpoint-specific** — the condition applies to a specific endpoint or resource path
- **Regional / zonal** — the condition applies to a specific geographic region or availability zone

#### Relationship to Subject

Scope and Subject are related but distinct:

- **Subject** identifies *what* is being described
- **Scope** identifies *the boundary* of applicability

A single subject may have different conditions at different scopes. For example, a service may be "operational" globally but "degraded" in a specific region. These are two separate condition statements with the same subject but different scopes.

#### Default Scope

If no explicit scope is provided, the condition applies to the subject as a whole. Profiles may define more specific default scope rules.

---

## Concept Relationships

The following diagram shows how the core concepts relate within a condition statement:

```
┌─────────────────────────────────────────────────────────┐
│                     SNAPSHOT                            │
│                                                         │
│  ┌─────────────────────────────────────────────────┐    │
│  │            CONDITION STATEMENT                  │    │
│  │                                                 │    │
│  │  Subject ──── "what is being described"         │    │
│  │  Scope ────── "boundary of applicability"       │    │
│  │  Condition ── "current state"                   │    │
│  │  Timing ──── "when observed / reported"         │    │
│  │  Evidence ─── "why this condition"              │    │
│  │  Provenance ─ "who determined this"             │    │
│  └─────────────────────────────────────────────────┘    │
│                                                         │
│  ┌──────────────────────────────────────┐               │
│  │     STRUCTURAL RELATIONSHIPS        │               │
│  │                                      │               │
│  │  Subject ──→ Components (DAG)        │               │
│  │  Subject ──→ Dependencies (DAG)      │               │
│  └──────────────────────────────────────┘               │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Cardinality Summary

| Relationship | Cardinality |
|---|---|
| Snapshot → Condition Statements | 1..* |
| Condition Statement → Subject | exactly 1 |
| Condition Statement → Scope | exactly 1 (default: whole-service) |
| Condition Statement → Condition | exactly 1 |
| Condition Statement → Timing | 0..1 (recommended) |
| Condition Statement → Evidence | 0..* |
| Condition Statement → Provenance | exactly 1 |
| Subject → Components | 0..* |
| Subject → Dependencies | 0..* |

---

## Extensibility

The core model is designed for extension without breaking existing profiles or serializations.

### Extension Rules

1. **New concepts may be added** to the core model in future versions. Existing concepts must not be removed or have their semantics changed in incompatible ways.
2. **New provenance types** may be defined. Consumers must handle unrecognized provenance types gracefully.
3. **New scope types** may be introduced. The set of scope categories is not closed.
4. **Profiles extend, not replace** — profiles select and constrain core concepts but must not redefine their semantics. A "Condition" in a Health profile means the same thing as "Condition" in the core model, just with additional constraints.
5. **Serializations map, not invent** — serializations must not introduce new semantic concepts that have no counterpart in the core model. If a serialization needs a concept that doesn't exist in the core model, the core model should be extended first.

### Stability Tiers

Core concepts have varying stability expectations:

- **Stable** (Subject, Condition, Timing, Provenance) — unlikely to change in meaning. Field-level details may be refined.
- **Evolving** (Evidence, Scope, Components/Dependencies) — the concepts are stable, but their internal structure may gain additional capabilities in future versions.

---

## What the Core Model Does Not Define

The following are explicitly outside the core model's responsibility:

- **Wire-level field names or JSON structure** — that is Serializations (Layer 3)
- **Domain-specific field requirements** — that is Profiles (Layer 2)
- **How to find an endpoint** — that is Discovery (Layer 5)
- **How to translate a legacy format** — that is Adapters (Layer 4)
- **Specific condition values** — that is Profiles + Phase 3 vocabulary definitions
- **Trust or confidence levels** — consumer-level concerns, not core semantics
- **Transition or event semantics** — v1 is snapshot-only; future extensions may address this

---

## References

- [ARCHITECTURE.md](../ARCHITECTURE.md) — Layer 1 definition
- [Glossary](https://github.com/open-operational-state/governance/blob/main/GLOSSARY.md) — authoritative terminology
- [Design Note 006: Core Model Invariants](../design-notes/006-core-model-invariants.md) — formal invariant definitions
