# Design Note 004: Discovery Mechanism Hierarchy

## Decision

Discovery mechanisms have a defined priority order:

1. **Link-based discovery** (primary)
2. **Well-known path** (baseline fallback)
3. **Inline references**
4. **DNS bootstrap** (optional)

## Context

Monitors need to find operational-state resources. The standard supports multiple discovery mechanisms because no single mechanism works in all scenarios. Without a defined priority order, implementations will diverge — some preferring well-known paths, others preferring link headers, creating inconsistent behavior.

## Rationale for Priority Order

### Link-based = Primary

Link-based discovery (HTTP `Link` headers per RFC 8288) is the most **authoritative** mechanism because:

- The target explicitly declares its operational-state resources
- It works from any existing endpoint (no special path required)
- It supports rich metadata (profile, serialization, auth requirements) via link parameters
- It follows established HTTP standards for resource relationship declaration

### Well-known path = Baseline Fallback

Well-known paths (per RFC 8615) serve as the **universal entry point** when no other discovery information is available:

- No prior knowledge of the target required — any client can attempt the well-known path
- Simple to implement and deploy
- Works as a bootstrap for discovering link-based and other discovery information

The well-known path is a **discovery document**, not the operational-state payload itself. This is an important distinction — the path serves as a pointer to operational-state resources, not as a health endpoint.

### Inline References = Supplementary

Resources linking to related resources provide navigational discovery:

- A liveness endpoint links to a richer health endpoint
- A public health endpoint links to an authenticated status endpoint
- Useful for progressive disclosure of operational-state depth

Inline references are ranked below well-known paths because they require already having found an initial resource.

### DNS = Optional Bootstrap

DNS TXT records provide domain-level discovery:

- Domain ownership verification
- Ecosystem-wide discovery for tools that scan many domains
- Pre-HTTP optimization (discover before connecting)

DNS is ranked last because:

- It introduces a dependency outside HTTP (the primary protocol surface)
- Not all targets control their DNS records
- DNS caching semantics differ from HTTP caching
- Any resource discoverable via DNS must also be discoverable through HTTP mechanisms

## Why Not Just Well-Known Paths?

Well-known paths alone are insufficient because:

- They assume a single operational-state entry point per origin
- They don't accommodate multi-service or multi-tenant origins
- They provide limited metadata about what's available
- They create a de facto "magic path" convention that may conflict with application routing

Link-based discovery solves all of these problems while well-known paths serve as the universal fallback.

## Trade-offs

**In favor of defined hierarchy:**
- Consistent behavior across implementations
- Clear guidance for implementers
- Predictable discovery experience for consumers

**Against:**
- Hierarchy adds conceptual complexity
- Some deployments may find well-known paths sufficient and skip link-based discovery
- Priority enforcement adds implementation burden

The trade-offs favor a defined hierarchy because inconsistent discovery behavior is the greater risk.

## References

- [discovery.md](../../spec/discovery.md)
- [RFC 8288](https://www.rfc-editor.org/rfc/rfc8288) — Web Linking
- [RFC 8615](https://www.rfc-editor.org/rfc/rfc8615) — Well-Known URIs
