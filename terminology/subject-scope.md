# Terminology: Subject, Scope, Component, and Dependency

This document provides applied terminology guidance for the **Subject/Scope/Component/Dependency** family — four related but distinct concepts that are frequently confused.

For authoritative definitions, see the [Glossary](https://github.com/open-operational-state/governance/blob/main/GLOSSARY.md). For full conceptual definitions, see [core-model.md](../spec/core-model.md).

---

## Disambiguation

| Term | Question Answered | Example |
|---|---|---|
| **Subject** | "What is being described?" | "The payment API" |
| **Scope** | "What boundary does this apply to?" | "In the us-east-1 region" |
| **Component** | "What are its internal parts?" | "The order processing worker" |
| **Dependency** | "What external things does it rely on?" | "The Stripe payment gateway" |

These four concepts work together in condition statements but serve different semantic roles.

---

## Subject

The entity whose operational state is being described. Every condition statement has exactly one subject.

### Key Properties

- **Identity** — a subject must be uniquely identifiable within a snapshot and stably identifiable across snapshots
- **Contextual role** — the same entity can be a "service" (from its own perspective), a "dependency" (from a consumer's perspective), or a "component" (from a parent service's perspective)
- **Not tied to a URL** — a subject is a logical entity, not a specific endpoint. A single service (subject) may have multiple endpoints

### What Subject Is Not

- Subject is not a URL (a URL is a resource location, not an entity identity)
- Subject is not a hostname (a hostname is a network address, not a logical service identifier)
- Subject is not a health endpoint (an endpoint is where you get state information about a subject)

---

## Scope

What part of the system a condition statement applies to. Every condition statement has exactly one scope (defaulting to "whole service" if unspecified).

### Scope vs. Subject

The distinction is subtle but important:

- **Subject** = "what entity"
- **Scope** = "which aspect or boundary of that entity"

A single subject may have different conditions at different scopes:

| Subject | Scope | Condition |
|---|---|---|
| Payment API | global | operational |
| Payment API | us-east-1 | operational |
| Payment API | eu-west-1 | degraded |

These are three separate condition statements about the **same subject** at **different scopes**.

### Scope Types

- **Whole-service** — the condition applies to the entire subject (default)
- **Component-specific** — the condition applies to a specific internal component
- **Dependency-specific** — the condition applies to a specific external dependency
- **Endpoint-specific** — the condition applies to a specific API endpoint or resource path
- **Regional / zonal** — the condition applies within a geographic region or availability zone

---

## Component vs. Dependency

Both components and dependencies are subjects — they have their own conditions, timing, evidence, and provenance. The distinction is about **operational boundary**:

### Component (Internal)

A constituent part of the target that exists **within** its operational boundary.

- The target operates and controls the component
- Component health is the target's responsibility
- Examples: database shard, worker process pool, cache layer, microservice within an application

### Dependency (External)

An external system that the target relies on but **does not own or operate**.

- The target does not control the dependency
- Dependency availability is outside the target's direct operational responsibility
- Examples: third-party payment API, cloud provider DNS, external authentication service

### Contextual Relativity

The component/dependency distinction is **relative to the observer**:

```
Service A
├── Component: Worker Pool        (internal to A)
├── Component: Cache Layer         (internal to A)
└── Dependency: Service B          (external to A)
    ├── Component: Database        (internal to B)
    └── Dependency: Service C      (external to B)
```

Service B is a "dependency" from A's perspective, but a "service" (subject) from its own perspective, with its own components and dependencies.

### Why This Distinction Matters

- **Responsibility** — a degraded component is the target's problem to fix; a degraded dependency may require escalation to another team
- **Adapter mapping** — some source formats (e.g., Spring Boot Actuator) distinguish between components and external dependencies
- **Aggregation** — condition aggregation rules may treat component and dependency failures differently
- **Trust** — component conditions are typically self-reported; dependency conditions may be externally observed

---

## Structural Relationships

Components and dependencies form a **directed acyclic graph (DAG)**. Cycles are not permitted within a single snapshot.

```
Subject A
├──→ Component B       (A contains B)
├──→ Component C       (A contains C)
│    └──→ Dependency E  (C depends on E)
└──→ Dependency D      (A depends on D)
```

- Edges are directed: from parent to child (components) or from dependent to dependency
- No cycles: if A depends on B, B must not depend on A (directly or transitively) within the same snapshot
- No mandatory depth: a snapshot may report any level of structural detail

---

## References

- [Glossary](https://github.com/open-operational-state/governance/blob/main/GLOSSARY.md) — authoritative definitions for all four terms
- [Core Model: Subject](../spec/core-model.md#1-subject)
- [Core Model: Scope](../spec/core-model.md#7-scope)
- [Core Model: Components and Dependencies](../spec/core-model.md#5-components-and-dependencies)
