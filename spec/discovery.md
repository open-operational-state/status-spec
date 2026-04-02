# Discovery

> **Status:** Outline — not yet normative.

## Purpose

Discovery defines how monitors locate available operational-state resources and determine what they represent. It is a first-class architectural layer, not an afterthought.

## Key Principle

Monitors should not have to guess where operational-state resources live or what they represent.

## Discovery Mechanisms

The standard should support multiple discovery approaches:

### Link-Based Discovery

HTTP Link headers and link relations that advertise operational-state resources from existing endpoints.

### Predictable Resources

Well-known paths (e.g., via RFC 8615) as a baseline discovery mechanism.

### DNS Bootstrap

Optional DNS TXT records for domain-level discovery. DNS is a secondary mechanism — HTTP is the primary protocol surface.

### Inline References

Operational-state resources linking to related resources (e.g., a shallow liveness endpoint linking to a richer health resource).

## What Discovery Communicates

- **Resource location** — where operational-state endpoints exist
- **Resource relations** — how endpoints relate to each other
- **Profile awareness** — which profile(s) a resource implements
- **Access requirements** — public vs. authenticated endpoints
- **Depth indication** — which resources offer richer vs. shallower information

## Open Questions

- Specific link relation types to register or use
- Well-known path conventions
- DNS TXT record format for bootstrap discovery
- How discovery interacts with capabilities (Layer 6)
- Discovery for multi-tenant or multi-service environments

## References

- [ARCHITECTURE.md](../ARCHITECTURE.md) — Layer 5 definition
- [RFC 8288](https://www.rfc-editor.org/rfc/rfc8288) — Web Linking
- [RFC 8615](https://www.rfc-editor.org/rfc/rfc8615) — Well-Known URIs
