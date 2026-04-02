# Serializations

> **Status:** Outline — not yet normative.

## Purpose

Serializations define wire-level representations of operational-state data, decoupled from semantic meaning. They specify how core model concepts are expressed in a concrete format.

## Key Principle

Meaning and wire shape are distinct concerns. The architecture supports multiple normative serializations rather than one forced universal body format.

## Expected Serializations

At minimum, v1 is expected to define serializations informed by the two prior-art lineages:

- A serialization compatible with the **health-response lineage** (`draft-inadarei-api-health-check`)
- A serialization compatible with the **service-status lineage** (`draft-dallariva-web-service-status-json`)

Additional serializations may be defined as needed.

## What Serializations Define

- Field names and structure for a given wire format
- Required vs. optional fields in that format
- Mapping from serialization fields to core model concepts
- Content types and media type conventions
- Serialization-specific constraints

## Open Questions

- Exact field names and JSON structure for each serialization
- Whether non-JSON serializations are in scope for v1
- Content-type registration strategy
- How minimal serializations (HTTP status code only) are handled

## References

- [ARCHITECTURE.md](../ARCHITECTURE.md) — Layer 3 definition
- [PRIOR-ART.md](../PRIOR-ART.md) — the drafts that inform serialization design
