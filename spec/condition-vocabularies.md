# Condition Vocabularies

> **Status:** Draft specification — Phase 3.

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## Purpose

This document defines the normative condition values for each v1 profile. Condition vocabularies are the concrete value sets that populate the [Condition](core-model.md#2-condition) concept in the core model.

## Vocabulary Structure

Each vocabulary:

- Is **profile-scoped** — each profile defines its own vocabulary appropriate to its use case
- Is **enumerable** — values come from a known, finite set
- Declares a **comparability model** — values are either orderable or categorical
- Is **extensible** — custom values are permitted via the `x-{vendor}-{value}` prefix convention

---

## Liveness Vocabulary

**Profile:** Liveness
**Comparability:** Orderable

| Value | Meaning | Order |
|---|---|---|
| `alive` | Service is reachable and accepting connections | 1 (best) |
| `unreachable` | Service is not responding to connection attempts | 2 (worst) |

**Usage constraints:**

- A Liveness response MUST use one of these two values
- Liveness condition MUST NOT reflect downstream dependency failures — a service that is running and accepting connections is `alive` even if a backend is unavailable
- The ordering MUST be: `alive` > `unreachable`

---

## Readiness Vocabulary

**Profile:** Readiness
**Comparability:** Orderable

| Value | Meaning | Order |
|---|---|---|
| `ready` | Service is ready to handle requests — all critical dependencies available, initialization complete | 1 (best) |
| `initializing` | Service is starting up and not yet ready to handle requests | 2 |
| `not-ready` | Service is live but cannot usefully process requests — critical dependency unavailable or configuration incomplete | 3 (worst) |

**Usage constraints:**

- A Readiness response MUST use one of these three values
- The ordering MUST be: `ready` > `initializing` > `not-ready`
- A service MAY be `alive` (Liveness) while simultaneously `not-ready` (Readiness) — this is expected during startup, dependency outages, or draining

---

## Health Vocabulary

**Profile:** Health
**Comparability:** Mixed (orderable core + categorical extensions)

### Orderable Values

| Value | Meaning | Order |
|---|---|---|
| `operational` | Fully operational — all systems functioning within normal parameters | 1 (best) |
| `degraded` | Operational with reduced capacity, elevated errors, or impaired performance | 2 |
| `partial-outage` | Some functionality is unavailable while other functionality continues to operate | 3 |
| `major-outage` | Most functionality is unavailable — widespread impact | 4 |
| `down` | Completely unavailable — no functionality accessible | 5 (worst) |

The ordering MUST be: `operational` > `degraded` > `partial-outage` > `major-outage` > `down`.

### Categorical Values

| Value | Meaning |
|---|---|
| `maintenance` | Planned maintenance is in progress — intentional, expected unavailability |
| `unknown` | Condition cannot be determined — insufficient information or check failure |

**Usage constraints:**

- A Health response MUST use a value from the orderable or categorical sets above, or a valid extension value
- Categorical values MUST NOT be compared using the orderable scale — `maintenance` is not "between" any two orderable values
- When aggregating component conditions, only orderable values participate in worst-of calculations — categorical values SHOULD be reported separately
- `unknown` indicates a gap in observability, not a condition — consumers SHOULD treat `unknown` as requiring investigation rather than as a severity level

---

## Status Vocabulary

**Profile:** Status
**Comparability:** Mixed (inherits Health vocabulary)

The Status profile uses the **Health vocabulary** in its entirety (both orderable and categorical values). Status adds no new core vocabulary values but permits the full use of extension values.

Status endpoints are expected to provide the richest condition information, including per-component and per-dependency conditions, each using Health vocabulary values.

---

## Extension Values

Custom condition values MAY be defined by implementations for domain-specific needs.

### Format

Extension values MUST use the prefix convention:

```
x-{vendor}-{value}
```

Where:

- `{vendor}` is a short, lowercase identifier for the organization defining the extension (e.g., `acme`, `cloudco`)
- `{value}` is the condition name, using lowercase with hyphens (e.g., `migrating`, `scaling-up`)

Examples:

- `x-acme-migrating`
- `x-cloudco-scaling-up`
- `x-internal-failover-active`

### Semantics

- Extension values MUST be treated as **categorical** by consumers that do not recognize them — no ordering assumed
- An implementation that defines extension values SHOULD document their meaning
- Extension values MUST NOT shadow or redefine any core vocabulary value
- Extension values MAY appear in any profile that uses the Health or Status vocabulary

### Future Registry

A formal extension registry is not defined in v1. Future versions MAY introduce a registration mechanism for promoting widely-used extension values to standard vocabulary values.

---

## Ecosystem Mapping

The following tables define how condition values from prior art and common implementations map to the vocabularies defined above. These mappings are normative for adapters (see [Adapters](adapters.md)) and informative for general reference.

### draft-inadarei-api-health-check Mapping

| Source Value | Liveness | Readiness | Health |
|---|---|---|---|
| `pass` | `alive` | `ready` | `operational` |
| `warn` | `alive` | `ready` | `degraded` |
| `fail` | `unreachable` | `not-ready` | `down` |

**Notes:**

- `pass` maps to the best state in each profile's vocabulary
- `warn` maps to `degraded` because the draft defines it as "healthy, with some concerns"
- For Liveness: `warn` maps to `alive` because a warning does not indicate unreachability
- `fail` at the Liveness level implies unreachability; at the Health level it may map to `down` or `major-outage` depending on context — adapters SHOULD default to `down`

### Spring Boot Actuator Mapping

| Source Value | Liveness | Readiness | Health |
|---|---|---|---|
| `UP` | `alive` | `ready` | `operational` |
| `DOWN` | `unreachable` | `not-ready` | `down` |
| `OUT_OF_SERVICE` | `alive` | `not-ready` | `maintenance` |
| `UNKNOWN` | `alive` | `not-ready` | `unknown` |

**Notes:**

- `OUT_OF_SERVICE` maps to `maintenance` (categorical) because Spring Boot uses it for intentional unavailability
- `UNKNOWN` at the Liveness level maps to `alive` because the service is responding (if it returned a status, it is reachable)

### Kubernetes Probe Mapping

| Source Signal | Liveness | Readiness |
|---|---|---|
| HTTP 2xx response | `alive` | `ready` |
| HTTP non-2xx response | `unreachable` | `not-ready` |
| Connection refused / timeout | `unreachable` | `not-ready` |

**Notes:**

- Kubernetes probes are profile-specific by design — liveness, readiness, and startup probes serve distinct roles that map naturally to the Liveness and Readiness profiles
- Kubernetes does not define Health or Status profile equivalent — probe responses typically cannot satisfy profiles beyond Readiness
- Startup probes map to `initializing` (Readiness) during the startup window

### HTTP Status Code Mapping

| HTTP Status Range | Liveness | Health |
|---|---|---|
| 2xx | `alive` | `operational` |
| 3xx | `alive` | `operational` |
| 4xx | `alive` | *(not mappable — client error, not operational state)* |
| 5xx | `alive` | `down` |
| Connection failure | `unreachable` | `down` |

**Notes:**

- A 4xx response indicates the service is reachable and processing requests (therefore `alive`) but does not convey operational state — it is a client-side error
- A 5xx response indicates the service is processing but encountering server-side failures — at the Liveness level this is still `alive` (the service responded); at the Health level this is `down`
- Connection failure (no response at all) is the only signal that maps to `unreachable`

---

## Vocabulary Versioning

Condition vocabularies are versioned independently of the core model:

- Adding new values to a vocabulary (orderable or categorical) is a **minor** change
- Changing the ordering of existing orderable values is a **breaking** change requiring a new major version
- Removing a value from a vocabulary is a **breaking** change
- Adding extension values (`x-{vendor}-{value}`) does not affect the vocabulary version

---

## References

- [core-model.md](core-model.md) — the Condition concept that vocabularies populate
- [profiles.md](profiles.md) — the profiles that scope each vocabulary
- [Design Note 003: Condition Vocabulary Design](../design-notes/003-condition-vocabulary-design.md) — Phase 2 design space analysis
- [Design Note 007: Condition Value Naming](../design-notes/007-condition-value-naming.md) — rationale for specific value names
