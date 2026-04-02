# Terminology: Provenance

This document provides applied terminology guidance for **Provenance** — the concept that tracks how a condition statement was produced.

For the authoritative definition, see the [Glossary](https://github.com/open-operational-state/governance/blob/main/GLOSSARY.md). For the full conceptual definition, see [core-model.md](../spec/core-model.md).

---

## What Provenance Means

Provenance answers: _"Who says this is the condition?"_

It identifies **the source or method** by which a condition statement was generated. Provenance is per-condition-statement, not per-snapshot — a single snapshot may contain conditions from different sources.

## The Four Provenance Types

### Self-Reported

The target asserts its own condition. This is the most common provenance type for endpoints like `/health` or `/status`, where the service evaluates its own state and publishes the result.

**Example:** A web service's health endpoint runs internal checks (database connectivity, queue depth, error rate) and returns `{"status": "pass"}`.

### Externally Observed

A monitor or external system determined the condition based on its own observations. The target did not necessarily produce this condition statement.

**Example:** An uptime monitor probes a service's endpoint, receives no response within timeout, and reports the service as unreachable.

### Derived

The condition is computed from other data — aggregation, inference, or transformation of other condition statements or metrics.

**Example:** A dashboard computes a service's overall condition as "degraded" by applying worst-of aggregation across its component conditions.

### Manually Declared

A human operator has explicitly set the condition, overriding automated sources.

**Example:** An operator marks a service as "maintenance" during a planned deployment window.

---

## Provenance Does Not Imply Trust

A critical distinction: provenance describes **origin**, not **reliability**.

- A self-reported condition is not inherently more trustworthy than an externally observed one. A service may incorrectly report itself as healthy.
- An externally observed condition may miss internal state that only the service knows about.
- A derived condition is only as good as its inputs and aggregation logic.
- A manually declared condition may be stale or incorrect.

Trust, confidence, and authority are **consumer-level concerns** — they depend on the consumer's relationship with the source and their assessment of its reliability. The core model does not define trust semantics.

## Provenance vs. Evidence

| Concept | Question Answered |
|---|---|
| **Provenance** | "Who determined this condition?" |
| **Evidence** | "What data supported this determination?" |

These are complementary, not redundant:

- A **self-reported** condition may have rich evidence (the service checked its own database and measured response time)
- An **externally observed** condition may have different evidence (a monitor measured HTTP response time and received a 503)
- A **derived** condition's evidence might be the set of component conditions it was aggregated from

## Edge Cases

### Hybrid Provenance

Some scenarios involve multiple provenance types:

- A service runs its health checks (self-reported) but also incorporates the results of external probes it received (externally observed input). The resulting condition might be classified as **derived**, since it combines multiple sources.

- A monitor observes a service's health endpoint (externally observed access path) but the condition value itself was self-reported by the service. In this case, the provenance of the **condition** is self-reported — the monitor merely retrieved it.

**Guidance:** Provenance describes how the **condition value** was determined, not how it was **retrieved**. A monitor reading a self-reported health endpoint is relaying self-reported provenance, not creating externally-observed provenance.

### Provenance of Adapted Conditions

When an adapter translates a legacy format into the core model, the provenance of the resulting condition depends on the original source:

- If the adapter translates a service's own health endpoint → self-reported
- If the adapter translates an external monitor's observation → externally observed
- If the adapter infers condition from indirect signals → derived or inferred (the adapter's lossiness classification may also be relevant)

---

## References

- [Glossary: Provenance](https://github.com/open-operational-state/governance/blob/main/GLOSSARY.md#provenance)
- [Core Model: Provenance](../spec/core-model.md#6-provenance)
