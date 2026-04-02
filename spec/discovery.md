# Discovery

> **Status:** Conceptual definition — Phase 2 (not yet normative).

## Purpose

Discovery defines how monitors locate available operational-state resources and determine what they represent. It is a first-class architectural layer, not an afterthought bolted onto path conventions.

## Key Principle

**Monitors should not have to guess where operational-state resources live or what they represent.** Hard-coded path conventions are a useful baseline, but discovery must support richer, more reliable mechanisms.

---

## Discovery Priority Order

The standard defines a **priority hierarchy** for discovery mechanisms. More specific, explicitly provided discovery information takes precedence over fallback mechanisms.

| Priority | Mechanism | Role |
|---|---|---|
| **1 (primary)** | Link-based discovery | Explicit, authoritative advertisement by the target |
| **2 (baseline fallback)** | Well-known path | Predictable, convention-based bootstrap |
| **3** | Inline references | Resources linking to related resources |
| **4 (optional)** | DNS bootstrap | Domain-level discovery for ecosystem-aware clients |

This hierarchy means:

- If link-based discovery provides resource locations, those are authoritative
- If no explicit links are available, monitors may fall back to well-known paths
- DNS is never required and never the sole discovery mechanism

---

## Discovery Mechanisms

### 1. Link-Based Discovery (Primary)

HTTP Link headers and link relations that advertise operational-state resources from existing endpoints.

**How it works:**

- A target includes `Link` headers (per [RFC 8288](https://www.rfc-editor.org/rfc/rfc8288)) in HTTP responses from its primary endpoints
- Link relations identify the type and role of the operational-state resource
- Consumers parse Link headers to discover available resources

**Advantages:**

- Authoritative — the target explicitly declares its operational-state resources
- Discoverable from any endpoint — no need to guess paths
- Supports profile and serialization indication via link parameters

**Link relation strategy:**

- Existing registered link relations should be used where applicable
- New link relations may need to be registered for operational-state-specific roles
- Exact link relation types are a Phase 3 deliverable

### 2. Well-Known Path (Baseline Fallback)

A predictable URI path under `/.well-known/` (per [RFC 8615](https://www.rfc-editor.org/rfc/rfc8615)) that serves as a bootstrap discovery point.

**Role clarification:** the well-known path is a **discovery document**, not necessarily the health payload itself. It may:

- Point to operational-state resources at other paths
- Describe available profiles and serializations
- Provide links to specific operational-state endpoints

The exact well-known path name is deferred to Phase 3.

**Advantages:**

- No prior knowledge required — any client can attempt the well-known path
- Simple to implement
- Works as a starting point when no other discovery information is available

**Constraints:**

- The well-known path should not be the only discovery mechanism relied upon
- Targets may serve the well-known path even if they also provide link-based discovery
- The well-known resource should be safe to probe (no side effects, lightweight)

### 3. Inline References

Operational-state resources linking to related resources within their own responses.

**How it works:**

- A shallow endpoint (e.g., liveness) includes links to deeper endpoints (e.g., health, status)
- Resources advertise related resources at different depth levels
- Consumers can navigate from a simple entry point to richer operational-state information

**Use cases:**

- Liveness endpoint linking to full health endpoint
- Health endpoint linking to authenticated status endpoint
- Component-level resources linking to parent service status

### 4. DNS Bootstrap (Optional)

DNS TXT records for domain-level discovery of operational-state resources.

**How it works:**

- A domain publishes TXT records indicating the availability and location of operational-state resources
- Ecosystem-aware clients check DNS before making HTTP requests
- DNS provides a domain-ownership-verified entry point

**Constraints (locked):**

- Discovery **must not depend solely on DNS**
- Any resource discoverable via DNS must also be discoverable through HTTP-based mechanisms
- DNS is an optimization and an ownership signal, not a required dependency

**Exact TXT record format is deferred to Phase 3.**

---

## What Discovery Communicates

Regardless of mechanism, discovery must be able to communicate:

- **Resource location** — where operational-state endpoints exist (URLs)
- **Resource relations** — how endpoints relate to each other (shallow vs. rich, summary vs. detail)
- **Profile awareness** — which profile(s) a resource implements
- **Access requirements** — whether a resource is public or requires authentication
- **Depth indication** — which resources offer richer vs. shallower information
- **Serialization availability** — what wire formats are available at each resource

---

## Multi-Service / Multi-Tenant Discovery

When a single origin hosts multiple services or tenants, discovery must accommodate:

- **Path-scoped discovery** — different services at different path prefixes may have different operational-state resources
- **Host-based separation** — virtual hosts or subdomains may each have their own operational-state endpoints
- **Aggregation points** — a single discovery document may enumerate multiple services

The conceptual approach is that the discovery mechanism does not assume one-service-per-origin. Exact patterns for multi-service discovery are a Phase 3 deliverable.

---

## Public / Private Discovery Split

Discovery must support the distinction between public and authenticated operational-state resources:

- **Public resources should exist where possible** — they lower adoption friction and enable basic interoperability
- **Authenticated resources should extend, not replace, public ones** — deeper detail behind auth, not hidden basics
- Discovery should indicate the existence of authenticated endpoints without exposing their content
- A monitor should be able to discover that richer information is available behind authentication, then decide whether to authenticate

---

## Boundary with Capabilities

Discovery and Capabilities are complementary but distinct:

- **Discovery identifies resources** — it answers "where can I find operational-state information?"
- **Capabilities describes resources** — it answers "what does this resource support and how can I interact with it?"

A discovery mechanism locates an endpoint; capabilities metadata describes what profiles, serializations, and auth modes that endpoint supports.

---

## References

- [ARCHITECTURE.md](../ARCHITECTURE.md) — Layer 5 definition
- [capabilities.md](capabilities.md) — the layer that describes discovered resources
- [RFC 8288](https://www.rfc-editor.org/rfc/rfc8288) — Web Linking
- [RFC 8615](https://www.rfc-editor.org/rfc/rfc8615) — Well-Known URIs
- [Design Note 004: Discovery Mechanism Hierarchy](../design-notes/004-discovery-mechanism-hierarchy.md) — rationale for priority order
