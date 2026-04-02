# Design Note 002: Profile Boundaries

## Decision

v1 defines four profiles: **Liveness**, **Readiness**, **Health**, and **Status**. Profiles are composable — a single resource may declare multiple profiles.

## Context

Operational-state resources serve different consumers with different needs:

- Infrastructure (load balancers, reverse proxies) needs fast binary liveness signals
- Orchestrators (Kubernetes, deployment pipelines) need readiness gates
- Monitoring systems need ongoing health insights with component detail
- Operators need comprehensive status with full dependency trees

## Why Four Profiles

The four-profile model maps to real-world usage patterns observed across platforms:

| Profile | Real-World Analog | Typical Consumer |
|---|---|---|
| Liveness | Kubernetes liveness probe, load balancer health check | Infrastructure |
| Readiness | Kubernetes readiness probe, deployment gate | Orchestrators |
| Health | Spring Boot Actuator `/health`, `application/health+json` | Monitoring systems |
| Status | Status pages, comprehensive dashboards | Operators, incident response |

### Why Not Fewer?

Collapsing to fewer profiles loses important pragmatic distinctions:

- **Liveness ≠ Readiness** — a service can be live (process running, port open) but not ready (still initializing, missing config). Kubernetes explicitly separates these, and that distinction is operationally meaningful.
- **Health ≠ Status** — health provides ongoing operational condition suitable for automated monitoring. Status provides comprehensive detail suitable for human investigation. Collapsing them either overburdens simple monitors or under-serves operators.

### Why Not More?

More profiles increase complexity without proportional benefit for v1:

- The four profiles cover the dominant real-world use cases for web services
- Additional profiles (e.g., "startup," "diagnostic," "performance") can be added in future versions
- The composability model means each profile can be well-scoped without worrying about gaps

## Composability Decision

**Profiles are composable. A single resource may declare conformance to multiple profiles simultaneously.**

Rationale:

- Real systems do not split endpoints cleanly — a Spring Boot `/actuator/health` endpoint provides information relevant to liveness, readiness, and health simultaneously
- Forcing exclusivity creates artificial fragmentation, requiring targets to maintain more endpoints than necessary
- Users naturally expect that a "Health" endpoint also provides a liveness signal

### Composability Rules

1. A resource declaring multiple profiles must satisfy **all** declared profiles' requirements
2. Profile declarations are additive — each adds constraints, never removes them
3. The profile hierarchy (Status ⊃ Health ⊃ Readiness ⊃ Liveness) means higher profiles automatically subsume lower ones

## Trade-offs

**In favor of four composable profiles:**
- Maps to existing real-world patterns
- Each profile is tractable in isolation
- Composability reduces endpoint proliferation

**Against:**
- Four profiles is more complex than one or two
- Composability rules add conceptual overhead
- Profile-specific condition vocabularies create vocabulary management burden

## References

- [profiles.md](../../spec/profiles.md)
- [ARCHITECTURE.md: Layer 2](../../ARCHITECTURE.md#layer-2-profiles)
