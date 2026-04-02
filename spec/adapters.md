# Adapters

> **Status:** Outline — not yet normative.

## Purpose

Adapters translate native or legacy target formats into the core model and profiles. They enable the ecosystem to interpret existing operational-state endpoints without requiring targets to change their implementations.

## Why Adapters Matter

The real world is full of existing health, status, and readiness endpoints. Broad interpretability requires the ability to ingest these existing formats, even imperfectly.

## What Adapters Define

- **Input format recognition** — how to identify what format is being adapted
- **Mapping rules** — how to translate fields and concepts to the core model
- **Fidelity indicators** — how complete or reliable the mapping is
- **Limitations** — what the adapter cannot infer from the source format

## Expected Adapter Targets

Adapters are expected to cover:

- Framework health endpoints (Spring Boot Actuator, ASP.NET, Rails, Express.js patterns)
- Vendor-specific health/status formats
- Kubernetes probe conventions
- Cloud provider health checks (AWS, GCP, Azure)
- Plain HTTP status code + body patterns
- Legacy text-based health responses

## Open Questions

- Adapter specification format (declarative vs. code-based)
- Adapter registry or discovery mechanism
- Confidence/fidelity scoring for adapter output
- How adapters interact with profiles (does an adapter target a specific profile?)

## References

- [ARCHITECTURE.md](../ARCHITECTURE.md) — Layer 4 definition
- [Conformance adapters](https://github.com/open-operational-state/status-conformance/blob/main/adapters/) — adapter validation in conformance repo
