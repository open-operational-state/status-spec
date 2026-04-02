# Non-Goals

This document explicitly defines what the Open Operational State v1 standard is **not** intended to be. Its purpose is to prevent scope creep and maintain focus.

---

## The Standard Is Not a Health-Check Format

The standard defines a full architecture for operational state — not just a JSON response shape for health checks. A health-check response format may be one serialization within the standard, but the standard is broader than any single format.

## The Standard Is Not a Full Observability Standard

Observability encompasses metrics, distributed tracing, logging, and event correlation. The operational-state standard is narrower: it covers what a service says about its condition, not the full telemetry stack.

## The Standard Is Not a Metrics or Tracing Standard

Metrics (Prometheus, OpenMetrics) and tracing (OpenTelemetry, Jaeger) are complementary but separate concerns. The operational-state standard may reference concepts from these domains but does not attempt to replace or replicate them.

## The Standard Is Not a Synthetic Testing Protocol

Synthetic transaction coordination — externally executed user-like workflows, form testing, controlled end-to-end verification — is a related but distinct concern. A future companion protocol may address synthetic testing, but it is **not part of v1**.

The standard may recognize provenance concepts (self-reported vs. externally observed) without merging synthetic coordination into the spec.

## The Standard Is Not Universal

v1 is locked to **machine-readable operational state of web services only**. It is not:

- A universal standard for all machine-readable state across all systems
- An attempt to standardize network device telemetry
- A protocol for local-only process health outside a web interface
- A generalized "everything that can emit state" standard

The architecture is extensible to non-web systems in future versions, but v1 does not target them.

## The Standard Does Not Mandate a Single Wire Format

The architecture explicitly supports multiple normative serializations. A single forced universal body format is a rejected approach. See [ARCHITECTURE.md](ARCHITECTURE.md) for the rationale.

## The Standard Does Not Own Discovery Exclusively

Discovery is a first-class layer, but the standard does not attempt to be the universal service discovery mechanism. It defines how monitors discover operational-state resources specifically, not how services discover each other in general.

## The Standard Does Not Define Authentication

The standard acknowledges authentication as a first-class concern and supports authenticated resources, but it does not define new authentication schemes. It works with existing, realistic auth approaches.
