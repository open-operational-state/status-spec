# Core Model

> **Status:** Outline — not yet normative.

## Purpose

The core model is the stable, transport-agnostic semantic center of the Open Operational State standard. It defines the canonical concepts that all profiles and serializations reference.

## Scope

The core model defines **what operational state means**, independent of how it is represented on the wire or discovered by monitors.

## Concepts to Define

The following concepts are expected to be part of the core model. Exact definitions, constraints, and relationships will be specified as the architecture stabilizes.

### Subject

The entity whose operational state is being described — a service, endpoint, component, or dependency.

### Condition

The normalized operational state of a subject at a point in time. Condition values should be drawn from a controlled vocabulary appropriate to the profile.

### Timing

When the condition was observed, reported, or last changed. Timing semantics must distinguish between observation time, report time, and state-change time where relevant.

### Evidence

The basis on which the condition is asserted — what checks, observations, or inputs informed the condition statement.

### Scope

What part of the system the condition applies to — the whole service, a specific component, a dependency, or a subset.

### Dependencies and Components

Hierarchical or compositional relationships between subjects. A service may report condition for its own components and external dependencies.

### Provenance

How the condition was produced:

- **Self-reported** — the target asserts its own condition
- **Externally observed** — a monitor or external system determined the condition
- **Derived** — computed from other data sources
- **Manually declared** — set by a human operator

## Open Questions

- Exact condition vocabulary and normalization rules
- Cardinality rules for components and dependencies
- Whether the core model defines severity or impact concepts
- Relationship between provenance and trust/confidence

## References

- [ARCHITECTURE.md](../ARCHITECTURE.md) — Layer 1 definition
- [Glossary](https://github.com/open-operational-state/governance/blob/main/GLOSSARY.md) — authoritative terminology
