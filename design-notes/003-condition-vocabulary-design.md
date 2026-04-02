# Design Note 003: Condition Vocabulary Design

## Decision

Phase 2 defines the **structure** of condition vocabularies. Phase 3 defines actual vocabulary values.

## Context

Condition is the most important concept in the core model. The values that a condition can take — and how those values relate to each other — have profound downstream effects on alerting, aggregation, UI rendering, adapter mapping, and interoperability.

## Design Space

Three axes of design choice:

### 1. Value Format

| Option | Example | Pros | Cons |
|---|---|---|---|
| **Enum** | `"operational"`, `"degraded"`, `"down"` | Simple, easy to compare, standardized | Rigid, hard to extend |
| **URI** | `"urn:oos:condition:operational"` | Extensible, namespaced, avoids collisions | Verbose, unfamiliar for simple cases |
| **Extensible string** | `"pass"`, `"x-vendor-migrating"` | Simple base, extensible via prefix convention | Less formal, collision risk |

**Phase 2 direction:** Condition values are drawn from defined, enumerable vocabularies. The exact format (enum vs. URI vs. extensible string) is a Phase 3 decision that will be informed by serialization design.

### 2. Comparability

**This must be decided in Phase 2** because it affects the core model's ability to support aggregation and alerting.

| Option | Description | Implications |
|---|---|---|
| **Orderable** | Values have a defined severity ordering | Enables worst-of aggregation, threshold alerting, severity comparison |
| **Categorical** | Values represent distinct states without ordering | Simpler model, but aggregation and comparison are undefined |
| **Mixed** | Some values are orderable, some are categorical | Most flexible, but most complex |

**Phase 2 decision:** A condition vocabulary must declare its comparability model. Vocabularies may contain both orderable and categorical values, provided the orderable subset is clearly identified.

This means:

- Orderable values can be compared: `operational > degraded > down`
- Categorical values cannot be ordered: `maintenance`, `migrating`, `provisioning` are distinct states, not severity levels
- Consumers can rely on ordering for the orderable subset
- Aggregation semantics (worst-of, majority, etc.) apply only to orderable values

### 3. Vocabulary Scope

| Option | Description |
|---|---|
| **Universal** | One vocabulary for all profiles |
| **Profile-scoped** | Each profile defines its own vocabulary |
| **Layered** | A core vocabulary shared across profiles, extended per-profile |

**Phase 2 direction:** Vocabularies are profile-scoped. Each profile defines the condition values appropriate to its use case. This allows Liveness to use a simple binary vocabulary while Status uses a richer multi-valued vocabulary.

## Why Not Define Values Now

Defining actual condition values in Phase 2 risks:

- **Bikeshed** — value naming is subjective and contentious (pass/fail? up/down? operational/degraded?)
- **Premature optimization** — values chosen now may not fit all serialization designs
- **Ecosystem lock-in** — early values may favor one ecosystem's conventions over another

Phase 3 vocabulary definition will be informed by:

- Serialization design (which shapes wire-level naming)
- Adapter analysis (which reveals the value spaces that existing formats use)
- Profile refinement (which clarifies what each profile needs to express)

## References

- [core-model.md: Condition](../../spec/core-model.md#2-condition)
- [terminology/condition.md](../../terminology/condition.md)
