# Profiles

> **Status:** Outline — not yet normative.

## Purpose

Profiles specialize the core model for concrete use cases within web-service scope. They define which core model concepts are required, optional, or constrained in a given context.

## Scope

Profiles narrow the general core model to specific operational scenarios without inventing new semantic categories.

## Expected v1 Profiles

The following profile distinctions are anticipated for v1. Exact definitions will be specified as the architecture stabilizes.

### Liveness

Is the service reachable and accepting connections? A minimal, shallow check.

### Readiness

Is the service ready to handle requests? May depend on downstream dependencies or initialization state.

### Health

What is the service's overall operational condition? Richer than liveness, may include component-level detail.

### Status

Rich, structured information about the service's state, components, dependencies, and operational metadata.

## What Profiles Define

- Required and optional core model concepts in context
- Condition vocabularies appropriate to the profile
- Constraints on evidence depth, provenance, and component reporting
- Valid interpretations and consumer expectations

## Open Questions

- Exact profile definitions and boundaries
- Whether profiles are composable or mutually exclusive
- How profiles relate to serializations (one profile may have multiple serializations)
- Profile versioning strategy

## References

- [ARCHITECTURE.md](../ARCHITECTURE.md) — Layer 2 definition
- [core-model.md](core-model.md) — the model that profiles specialize
