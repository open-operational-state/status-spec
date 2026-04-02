# Design Note 006: Core Model Invariants

## Purpose

This document formally defines the invariants that the core model must preserve. These invariants are constraints that every valid snapshot, serialization, profile, and adapter must respect. They are the foundation of interoperability.

---

## Invariant 1: Snapshot Immutability

**Once a snapshot is produced, its content does not change.**

Subsequent observations produce new snapshots. There is no mechanism for modifying a snapshot after creation. This ensures that any reference to a snapshot (by timestamp, identifier, or other means) is stable.

**Implication:** Serializations must not define mutable response semantics. An operational-state response is a snapshot, not a live-updating stream.

---

## Invariant 2: Snapshot Internal Consistency

**A single snapshot must not contain contradictory condition statements for the same subject at the same scope.**

If a snapshot contains multiple condition statements, each must be for a different subject or a different scope. Two condition statements asserting different conditions for the same subject at the same scope within a single snapshot is a violation.

**Implication:** Producers must resolve conflicts before publishing a snapshot. If multiple sources disagree about a subject's condition, the producer must choose one or scope them differently (e.g., by provenance).

---

## Invariant 3: Subject Identity Uniqueness

**Within a snapshot, each subject has a unique identity.**

No two subjects in a snapshot may share the same identity. Subject identity must be sufficient to distinguish one subject from another within the same snapshot.

**Across snapshots, subject identity must be stable.** The same entity in different snapshots must be identifiable as the same subject. This enables consumers to correlate conditions over time, compute trends, and detect changes.

**Implication:** Serializations must define an identity field or convention for subjects. Profiles may constrain what forms of identity are valid. Adapters must map source-format identifiers to stable subject identities.

---

## Invariant 4: Condition Uniqueness

**At most one condition per subject per scope per snapshot.**

This is a corollary of Invariant 2 but stated explicitly because it is the most operationally important constraint. A consumer reading a snapshot must be able to unambiguously determine "what is the condition of subject X at scope Y?"

Multiple sources or provenances are handled by the provenance concept — they do not create duplicate conditions. The snapshot producer is responsible for resolving multi-source input into a single authoritative condition per subject/scope.

**Implication:** A snapshot that contains `{subject: "api", scope: "global", condition: "operational"}` and `{subject: "api", scope: "global", condition: "degraded"}` is invalid.

---

## Invariant 5: Dependency Graph Acyclicity

**The component/dependency graph within a snapshot forms a directed acyclic graph (DAG). Cyclic dependencies are not permitted.**

If subject A lists subject B as a component or dependency, subject B must not list subject A (directly or transitively) as a component or dependency within the same snapshot.

**Rationale:** Cycles in operational-state graphs create ambiguity in:

- Condition aggregation (worst-of a cycle is undefined)
- Dependency impact analysis (circular impact is meaningless)
- Traversal and rendering (cycles cause infinite loops)

**Implication:** Producers must ensure acyclicity when constructing snapshot graphs. Validators must check for cycles. Adapters translating from formats that allow circular references must break cycles explicitly.

**Note:** This constraint applies within a single snapshot. The same entities may have different relationships in different snapshots (e.g., a dependency restructuring). Cross-snapshot consistency is not a core model invariant.

---

## Invariant 6: Serialization Semantic Preservation

**A serialization must not introduce new semantics that have no counterpart in the core model.**

If a wire format needs a concept that doesn't exist in the core model, the core model should be extended first. Serializations map core model concepts to wire-level fields — they do not invent meaning.

**Corollary:** A serialization must not silently drop or distort core model concepts within its declared coverage. If a serialization cannot represent a concept, it must declare that limitation.

**Implication:** Serialization authors must demonstrate a mapping table from their wire format to the core model. Fields that have no core model counterpart are either informational extensions (which must be documented as non-semantic) or evidence of a core model gap.

---

## Invariant 7: Profile Semantic Compatibility

**Profiles must not redefine core model semantics.**

A profile selects, constrains, and contextualizes core model concepts. It does not change what those concepts mean. "Condition" in a Liveness profile has the same fundamental meaning as "Condition" in a Status profile — the profile changes what values are valid and what evidence is expected, not the concept itself.

**Implication:** A consumer that understands the core model can interpret any profile, even an unfamiliar one, at a basic level. Profile-specific knowledge adds depth but is not required for basic interoperability.

---

## Summary Table

| # | Invariant | Scope |
|---|---|---|
| 1 | Snapshot immutability | Producers, serializations |
| 2 | Snapshot internal consistency | Producers |
| 3 | Subject identity uniqueness | Producers, serializations, adapters |
| 4 | Condition uniqueness (per subject/scope) | Producers |
| 5 | Dependency graph acyclicity | Producers, validators, adapters |
| 6 | Serialization semantic preservation | Serializations |
| 7 | Profile semantic compatibility | Profiles |

---

## References

- [core-model.md](../../spec/core-model.md) — normative specification where these invariants are encoded as MUST/MUST NOT requirements (Phase 3)
- [ARCHITECTURE.md](../../ARCHITECTURE.md) — the six-layer model these invariants protect
