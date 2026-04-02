# Design Note 007: Condition Value Naming

## Decision

The v1 condition vocabularies use the following canonical value names:

- **Liveness:** `alive` / `unreachable`
- **Readiness:** `ready` / `initializing` / `not-ready`
- **Health (orderable):** `operational` / `degraded` / `partial-outage` / `major-outage` / `down`
- **Health (categorical):** `maintenance` / `unknown`

## Context

Condition values are the most visible part of the standard. They appear in every response, drive alerting logic, populate dashboards, and are the first thing implementers see. Naming choices here have outsized impact on adoption and ergonomics.

The design space is wide. Existing ecosystems use at least four distinct naming families:

| Family | Values | Used By |
|---|---|---|
| pass/warn/fail | `pass`, `warn`, `fail` | draft-inadarei-api-health-check |
| up/down | `UP`, `DOWN`, `OUT_OF_SERVICE`, `UNKNOWN` | Spring Boot Actuator |
| ok/error | `ok`, `error` | Node.js Terminus |
| HTTP-derived | 200, 503 | Kubernetes probes, load balancers |

## Rationale

### Why "operational" over alternatives

| Candidate | Rejected Because |
|---|---|
| `pass` | Test-oriented framing. Implies a check was run, not that a state exists. Tied to draft-inadarei specifically. |
| `healthy` | Implies internal health. The standard models *externally visible operational state*, not internal system health. |
| `ok` | Too informal for a normative specification. Overloaded (HTTP 200, generic confirmation). |
| `up` | Overloaded across ecosystems (Kubernetes, Spring Boot, informal usage). Too binary — doesn't express the nuance between "running" and "fully functional." |
| `operational` | **Chosen.** Precise, professional, and directly aligned with the standard's name ("operational state"). Matches the vocabulary used by operations teams, status pages, and incident response. |

### Why "alive" / "unreachable" for Liveness

- `alive` is the standard term in distributed systems for "process is running and accepting connections" (cf. liveness probes)
- `unreachable` describes the observable outcome (cannot be reached) rather than asserting internal state (`dead` implies the process has terminated, which may not be true)
- `unreachable` is slightly network-biased but is more precise than `dead` and more descriptive than `down` (which is reserved for the Health vocabulary)

### Why "partial-outage" / "major-outage"

These terms match how real-world systems communicate incidents publicly. Status pages (Atlassian Statuspage, Instatus, etc.) universally use "partial outage" and "major outage" as severity levels. Aligning with this convention reduces cognitive load for operations teams.

### Why "maintenance" is categorical

Maintenance is not a severity level — it is an intentional state. A service in maintenance may be fully unavailable, but this is expected and planned. Treating maintenance as orderable (e.g., "worse than degraded but better than down") creates false alerting scenarios.

### Why "unknown" is categorical

`unknown` represents an observability gap, not a condition. It should trigger investigation, not severity-based alerting. Placing it in the orderable scale would force consumers to assign it a relative severity, which is semantically incorrect.

## Trade-offs

- **Longer names** (`operational`, `partial-outage`) vs. shorter (`pass`, `up`) — chosen longer for precision over brevity. Wire format efficiency is not a design goal for v1.
- **Hyphenated compound values** (`partial-outage`, `major-outage`, `not-ready`) — necessary for multi-word concepts. Consistent with HTTP header conventions and URI segment conventions.
- **Not reusing prior art names** — intentional. The standard defines a canonical semantic layer with adapters to prior ecosystems. Reusing names from one prior art lineage would create false equivalence with that lineage and confusion with others.

## References

- [condition-vocabularies.md](../../spec/condition-vocabularies.md) — the normative vocabulary definitions
- [Design Note 003: Condition Vocabulary Design](003-condition-vocabulary-design.md) — Phase 2 structural analysis
- [terminology/condition.md](../../terminology/condition.md) — applied terminology context
