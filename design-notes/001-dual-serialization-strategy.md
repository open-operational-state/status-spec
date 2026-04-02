# Design Note 001: Dual Serialization Strategy

## Decision

The v1 standard defines two primary JSON serializations rather than a single universal wire format.

## Context

Two prior-art Internet-Drafts inform the design:

- **`draft-inadarei-api-health-check-06`** — a focused, flat, component-oriented health-check format using `application/health+json` with `pass`/`fail`/`warn` status values
- **`draft-dallariva-web-service-status-json-00`** — a richer format with geographic scope, incident reporting, criticality classification, and structured component status

## Problem

These two drafts cannot be collapsed into a single strict universal wire format without structural conflicts:

- The health-check draft uses a flat `checks` object keyed by `{componentName}:{measurementName}` with arrays of observation objects
- The service-status draft uses a structured component list with criticality, geographic scope, and incident associations
- Their status vocabularies differ (`pass`/`fail`/`warn` vs. richer operational indicators)
- Their philosophies differ: one is observation-oriented, the other is status-page-oriented

Attempting to unify them into a single hybrid payload would create a format that:

- Is not cleanly compliant with either draft
- Forces unnecessary complexity on simple use cases
- Constrains extension in ways that satisfy neither lineage

## Decision Rationale

The correct architectural response is:

1. **One core model** — shared semantic foundation
2. **Multiple normative serializations** — each can faithfully represent the formats that practitioners already use
3. **Clear mappings** — between each serialization and the core model
4. **Internal normalization, external flexibility** — consumers normalize to the core model internally, but targets can publish in whichever serialization fits their use case

This approach:

- Honors existing adoption of both format families
- Enables adapters for existing health-check and status endpoints
- Avoids forcing a migration to a new, unfamiliar hybrid format
- Keeps each serialization simple and purpose-fit

## Trade-offs

**In favor of dual serialization:**
- Easier adoption — existing endpoints map naturally to one lineage
- Simpler individual serializations — each is internally consistent
- Better prior-art compatibility

**Against:**
- Two specs to maintain instead of one
- Consumers may need to handle both formats
- More complex capabilities/negotiation story

The trade-offs favor dual serialization because the alternative (a forced hybrid) fails on adoption and compatibility grounds.

## References

- [ARCHITECTURE.md: Layer 3](../../ARCHITECTURE.md#layer-3-serializations)
- [PRIOR-ART.md](../../PRIOR-ART.md)
- [serializations.md](../../spec/serializations.md)
