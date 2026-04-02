# Core Model

> **Status:** Draft specification — Phase 3.

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## Purpose

The core model is the stable, transport-agnostic semantic center of the Open Operational State standard. It defines the canonical concepts that all profiles and serializations reference.

The core model is where **long-term semantic durability** lives. Wire formats and serializations may evolve, but the meaning of operational-state concepts defined here MUST remain durable across versions and representations.

## Scope

The core model defines **what operational state means**, independent of:

- how it is represented on the wire (that is Serializations)
- what domain-specific constraints apply (that is Profiles)
- how endpoints are located (that is Discovery)
- how legacy formats are translated (that is Adapters)

The core model is the **normative semantic foundation** of the standard; all other layers MUST map to it.

---

## Snapshot Semantics

The core model represents a **snapshot of operational condition at a point in time**. Each observation produces an independent, immutable snapshot.

### Invariants

1. **Immutability** — once a snapshot is produced, its content MUST NOT change. Subsequent observations produce new snapshots.
2. **Internal consistency** — a single snapshot MUST NOT contain contradictory condition statements for the same subject at the same scope.
3. **Point-in-time** — a snapshot describes the state of the world as understood at a specific moment. It MUST NOT represent a time range or a transition.
4. **Independence** — snapshots are independent of each other. The core model does not define ordering, diffing, or transition semantics between snapshots. Future extensions MAY model condition transitions or event streams, but v1 treats each snapshot as standalone.

### What a Snapshot Contains

A snapshot MUST contain one or more **condition statements** — each binding a subject, at a scope, to a condition, with associated timing, evidence, and provenance. A snapshot MAY also contain component and dependency relationships between subjects.

---

## Core Concepts

The core model defines seven canonical concepts. Each concept has a precise semantic role that MUST be preserved across all serializations and profiles.

### 1. Subject

**Definition:** The entity whose operational state is being described.

A subject is the "thing" about which a condition statement is made. In v1 scope, subjects are web services, APIs, websites, web applications, or their constituent parts.

#### Identity

A subject MUST be **uniquely identifiable** within a snapshot and **stably identifiable** across snapshots.

- **Within a snapshot:** no two condition statements MAY refer to the same subject at the same scope without being contradictory. Subject identity MUST be sufficient to distinguish one subject from another.
- **Across snapshots:** a consumer MUST be able to determine whether two subjects in different snapshots refer to the same entity. This enables comparisons, trending, and change detection over time. Subject identity MUST be comparable across snapshots to enable longitudinal analysis. The standard does not define how identity is maintained over time (e.g., via registry, convention, or stable naming), but the identity value itself MUST be stable enough that two snapshots referring to the same entity use the same identity.

The core model defines that subjects MUST have an **identity** but does not mandate a specific identifier format (URI, UUID, string, etc.). Serializations MUST define how identity is expressed on the wire. Profiles MAY constrain what types of identifiers are valid.

#### Subject Types

The core model recognizes conceptual distinctions among subjects, but does not enforce a rigid taxonomy:

- **Service** — a top-level web service or API
- **Component** — a constituent part of a service (see [Components and Dependencies](#5-components-and-dependencies))
- **Dependency** — an external system that a service relies on (see [Components and Dependencies](#5-components-and-dependencies))

These distinctions are contextual. An entity that is a "dependency" from one service's perspective MAY be a "service" from its own.

---

### 2. Condition

**Definition:** The normalized operational state of a subject at a point in time.

Condition is the semantic core of any operational-state statement. It answers the question: _"What is the state of this thing right now?"_

#### Condition Values and Vocabularies

Condition values MUST be drawn from a **condition vocabulary** — a defined set of allowed values. The core model defines the **structure** of condition vocabularies; specific values are defined in [Condition Vocabularies](condition-vocabularies.md).

Structural requirements for condition vocabularies:

- **Profile-scoped** — each profile defines its own condition vocabulary appropriate to its use case. The core model does not mandate a single universal vocabulary.
- **Extensible** — vocabularies MUST support extension without breaking existing consumers. Custom extension values MUST use the `x-{vendor}-{value}` prefix convention.
- **Enumerable** — condition values MUST come from a known, finite set within a given vocabulary. Free-form strings MUST NOT be used as condition values.

#### Comparability Model

Condition values within a vocabulary MUST be declared as **orderable** or **categorical**:

- **Orderable** — values have a defined severity or health ordering (e.g., `operational` > `degraded` > `down`). This supports alerting thresholds, aggregation, and worst-of calculations.
- **Categorical** — values represent distinct states without inherent ordering (e.g., `maintenance`, `unknown`).

A condition vocabulary MUST declare which comparability model it uses. A single vocabulary MAY contain both orderable and categorical values, provided the orderable subset is clearly identified.

Consumers MUST NOT assume ordering for categorical values. Consumers SHOULD use the declared ordering for orderable values when performing aggregation or comparison.

The comparability model has downstream implications for:

- **Alerting** — can a monitor compare current condition to a threshold?
- **Aggregation** — can a service's overall condition be derived from component conditions?
- **UI** — can conditions be sorted or ranked?
- **Adapters** — can a foreign format's values be mapped to an ordered scale?

#### Condition Uniqueness

**For a given Subject and Scope within a snapshot, exactly one Condition MUST be defined.** If a subject has different conditions at different scopes, those are separate condition statements. If a subject has multiple condition sources (e.g., self-reported and externally observed), those are distinguished by provenance, not by duplicating the condition. This constraint prevents conflicting adapter outputs and ambiguous normalization.

---

### 3. Timing

**Definition:** Temporal metadata associated with a condition statement.

Timing records **when** something happened. The core model distinguishes three temporal concepts:

- **Observation time** — when the condition was observed or measured. This is the moment the check was performed.
- **Report time** — when the condition was communicated or made available. This MAY differ from observation time if there is caching or delayed publication.
- **State-change time** — when the condition last transitioned to its current value. This enables duration calculations ("has been degraded for 45 minutes").

#### Requirements

- Observation time SHOULD be present when known.
- Report time and state-change time are supplementary. Not all systems can provide them.
- All timestamps MUST use [RFC 3339](https://www.rfc-editor.org/rfc/rfc3339) format. Serializations MUST define the exact wire representation.
- Profiles MAY require or make optional specific timing fields based on the use case.

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

Evidence is OPTIONAL at the core model level. Profiles MAY require evidence for certain condition values (e.g., a Health profile SHOULD include evidence for degraded or failed conditions).

#### Relationship to Provenance

Evidence describes the basis for a condition; provenance describes how that evidence was obtained and by whom. A self-reported condition MAY have rich evidence (the service checked its own database). An externally observed condition MAY have different evidence (a monitor measured response time). These are complementary, not redundant. Implementations MUST maintain this distinction in serializations and adapters.

---

### 5. Components and Dependencies

**Definition:** Structural relationships between subjects within a snapshot.

A service MAY report condition not only for itself, but for its constituent parts (components) and the external systems it relies on (dependencies).

#### Distinction

- **Component** — an internal constituent part of the subject. The component exists within the subject's operational boundary. Examples: a microservice within an application, a database shard, a worker pool.
- **Dependency** — an external system or service that the subject relies on but does not own or operate. Examples: a third-party API, a cloud provider service, a DNS provider.

This distinction is **contextual and relative** — a dependency from one subject's perspective MAY be a service with its own components from another perspective.

#### Graph Structure

The component/dependency model MUST form a **directed acyclic graph (DAG)**.

**Invariants:**

1. **Acyclicity** — cyclic dependencies MUST NOT be permitted within a single snapshot. If A depends on B, B MUST NOT depend on A (directly or transitively) within the same snapshot.
2. **Direction** — edges MUST point from the subject to its components or dependencies.
3. **No mandatory depth** — the core model does not mandate a minimum or maximum depth. A snapshot MAY report only a top-level service with no components, or it MAY report arbitrarily deep component trees.

#### Component and Dependency Conditions

Each component or dependency is itself a **subject** with its own condition, timing, evidence, and provenance. This enables recursive condition modeling — a service can be "degraded" because one of its components is "down."

Profiles MAY define rules for how component and dependency conditions relate to the parent subject's condition (e.g., worst-of aggregation, majority quorum, or explicit override).

---

### 6. Provenance

**Definition:** How a condition was produced — the origin or method by which the condition statement was generated.

Provenance answers the question: _"Who says this is the condition?"_

#### Provenance Types

The core model defines four provenance types:

| Type | Description | Example |
|---|---|---|
| **Self-reported** | The target asserts its own condition. The entity being described produced the condition statement. | A web service's `/health` endpoint returns `{"condition": "operational"}`. |
| **Externally observed** | A monitor or external system determined the condition based on its own observations. | An uptime monitor reports the service as unreachable after probe failures. |
| **Derived** | The condition is computed from other data — aggregation, inference, or transformation of other condition statements or metrics. | A dashboard computes "degraded" from component-level conditions using worst-of aggregation. |
| **Manually declared** | A human operator has explicitly set the condition, overriding automated sources. | An operator marks a service as "maintenance" during a planned deployment. |

#### Semantics

- Provenance is **per-condition-statement**, not per-snapshot. A single snapshot MAY contain conditions with different provenances (e.g., a self-reported service condition alongside externally observed dependency conditions).
- Each condition statement MUST have exactly one provenance type. The provenance type MUST be one of the four defined types or a registered extension type.
- Multiple provenances MAY exist for the same subject/scope if they come from different sources. Conflict resolution rules are not defined by the core model but MAY be addressed by profiles or consumer implementations.
- Provenance does not imply **trust**. A self-reported condition is not inherently more or less trustworthy than an externally observed one. Trust and confidence are consumer-level concerns, not core model semantics.

---

### 7. Scope

**Definition:** What part of the system a condition statement applies to.

Scope delineates the **boundary** of a condition's applicability.

#### Usage

A condition statement always applies at a scope. Scope MAY be:

- **Whole-service** — the condition applies to the entire service
- **Component-specific** — the condition applies to a specific component
- **Dependency-specific** — the condition applies to a specific dependency
- **Endpoint-specific** — the condition applies to a specific endpoint or resource path
- **Regional / zonal** — the condition applies to a specific geographic region or availability zone

#### Relationship to Subject

Scope and Subject are related but distinct:

- **Subject** identifies *what* is being described
- **Scope** identifies *the boundary* of applicability

A single subject MAY have different conditions at different scopes. For example, a service MAY be "operational" globally but "degraded" in a specific region. These are two separate condition statements with the same subject but different scopes.

#### Default Scope

If no explicit scope is provided, the condition MUST be interpreted as applying to the subject as a whole. Profiles MAY define more specific default scope rules.

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
| Condition Statement → Timing | 0..1 (RECOMMENDED) |
| Condition Statement → Evidence | 0..* |
| Condition Statement → Provenance | exactly 1 |
| Subject → Components | 0..* |
| Subject → Dependencies | 0..* |

---

## Extensibility

The core model is designed for extension without breaking existing profiles or serializations.

### Extension Rules

1. **New concepts MAY be added** to the core model in future versions. Existing concepts MUST NOT be removed or have their semantics changed in incompatible ways.
2. **New provenance types MAY be defined.** Consumers MUST handle unrecognized provenance types gracefully (treat as opaque, do not reject the snapshot).
3. **New scope types MAY be introduced.** The set of scope categories is not closed.
4. **Profiles extend, not replace** — profiles select and constrain core concepts but MUST NOT redefine their semantics. A "Condition" in a Health profile means the same thing as "Condition" in the core model, just with additional constraints.
5. **Serializations map, not invent** — serializations MUST NOT introduce new semantic concepts that have no counterpart in the core model. If a serialization needs a concept that doesn't exist in the core model, the core model SHOULD be extended first.
6. **Custom condition values** MUST use the `x-{vendor}-{value}` prefix convention. Consumers MUST treat unrecognized condition values as categorical (no ordering assumed).

### Stability Tiers

Core concepts have varying stability expectations:

- **Stable** (Subject, Condition, Timing, Provenance) — unlikely to change in meaning. Field-level details MAY be refined.
- **Evolving** (Evidence, Scope, Components/Dependencies) — the concepts are stable, but their internal structure MAY gain additional capabilities in future versions.

---

## What the Core Model Does Not Define

The following are explicitly outside the core model's responsibility:

- **Wire-level field names or JSON structure** — that is Serializations (Layer 3)
- **Domain-specific field requirements** — that is Profiles (Layer 2)
- **How to find an endpoint** — that is Discovery (Layer 5)
- **How to translate a legacy format** — that is Adapters (Layer 4)
- **Specific condition values** — that is [Condition Vocabularies](condition-vocabularies.md)
- **Trust or confidence levels** — consumer-level concerns, not core semantics
- **Transition or event semantics** — v1 is snapshot-only; future extensions MAY address this

---

## References

- [ARCHITECTURE.md](../ARCHITECTURE.md) — Layer 1 definition
- [Condition Vocabularies](condition-vocabularies.md) — normative condition values per profile
- [Glossary](https://github.com/open-operational-state/governance/blob/main/GLOSSARY.md) — authoritative terminology
- [Design Note 006: Core Model Invariants](../design-notes/006-core-model-invariants.md) — formal invariant definitions
