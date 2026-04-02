# Terminology: Condition

This document provides applied terminology guidance for **Condition** — the most semantically loaded concept in the Open Operational State standard.

For the authoritative definition, see the [Glossary](https://github.com/open-operational-state/governance/blob/main/GLOSSARY.md). For the full conceptual definition, see [core-model.md](../spec/core-model.md).

---

## What Condition Means

Condition is the answer to: _"What is the state of this thing right now?"_

Condition is a **semantic concept** — it represents normalized operational state. It is neither a raw metric value (that is evidence) nor an HTTP status code (that is a wire-level representation).

## Condition vs. HTTP Status Code

| Concept | Role | Example |
|---|---|---|
| **Condition** | Semantic state of a subject | "degraded", "operational", "down" |
| **HTTP status code** | Wire-level transport indicator | 200, 503, 429 |

These are related but not equivalent:

- A 200 HTTP response may carry a condition of "degraded" (the service is reachable but not fully operational)
- A 503 HTTP response may carry a condition of "maintenance" (planned downtime, not a failure)
- An adapter may infer condition from an HTTP status code, but the inference is heuristic

Condition lives in the **core model**. HTTP status codes live in the **serialization** layer. Conflating them is a design error this architecture explicitly avoids.

## Condition vs. Evidence

| Concept | Question Answered |
|---|---|
| **Condition** | "What is the state?" |
| **Evidence** | "Why is the state what it is?" |

A condition of "degraded" might be supported by evidence that database response time is 500ms (exceeding a 200ms threshold). The condition is the summary; the evidence is the supporting data.

## Condition Values and Vocabularies

Condition values must come from a defined **condition vocabulary**. Free-form strings are not valid condition values.

Key structural properties of condition vocabularies:

- **Profile-scoped** — different profiles define different vocabularies appropriate to their use case
- **Enumerable** — values come from a known, finite set
- **Extensible** — the vocabulary can be expanded without breaking existing consumers
- **Comparability-declared** — each vocabulary declares whether its values are orderable (have severity ordering, supporting aggregation and threshold comparisons) or categorical (distinct states without inherent ordering)

Exact condition values are **not defined in Phase 2**. They are a Phase 3 deliverable.

## Condition Uniqueness

Within a single snapshot, there must be **at most one condition per subject per scope**. This invariant prevents ambiguity:

- If a service has different conditions in different regions, those are different scopes
- If a service has self-reported and externally-observed conditions, those are different provenances — but the condition value is determined by whoever produced the snapshot

## Common Misuses to Avoid

- Do not use "condition" as a synonym for "HTTP response" — condition is semantic, HTTP is transport
- Do not conflate condition values with metrics — "response time: 500ms" is evidence, not a condition
- Do not invent ad-hoc condition values outside a vocabulary — this breaks interoperability
- Do not assume condition values are universally ordered — some vocabularies are categorical

---

## References

- [Glossary: Condition](https://github.com/open-operational-state/governance/blob/main/GLOSSARY.md#condition)
- [Core Model: Condition](../spec/core-model.md#2-condition)
- [Design Note 003: Condition Vocabulary Design](../design-notes/003-condition-vocabulary-design.md)
