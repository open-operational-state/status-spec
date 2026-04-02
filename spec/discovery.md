# Discovery

> **Status:** Draft specification — Phase 3.

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## Purpose

Discovery defines how monitors locate available operational-state resources and determine what they represent. It is a first-class architectural layer, not an afterthought bolted onto path conventions.

## Key Principle

**Monitors SHOULD NOT have to guess where operational-state resources live or what they represent.** Hard-coded path conventions are a useful baseline, but discovery MUST support richer, more reliable mechanisms.

---

## Discovery Priority Order

The standard defines a **priority hierarchy** for discovery mechanisms. More specific, explicitly provided discovery information takes precedence over fallback mechanisms.

| Priority | Mechanism | Role |
|---|---|---|
| **1 (primary)** | Link-based discovery | Explicit, authoritative advertisement by the target |
| **2 (baseline fallback)** | Well-known path | Predictable, convention-based bootstrap |
| **3** | Inline references | Resources linking to related resources |
| **4 (optional)** | DNS bootstrap | Domain-level discovery for ecosystem-aware clients |

Monitors MUST respect this hierarchy:

- If link-based discovery provides resource locations, those are authoritative
- If no explicit links are available, monitors MAY fall back to well-known paths
- DNS MUST NOT be the sole discovery mechanism

---

## Discovery Mechanisms

### 1. Link-Based Discovery (Primary)

HTTP Link headers and link relations that advertise operational-state resources from existing endpoints.

**How it works:**

- A target SHOULD include `Link` headers (per [RFC 8288](https://www.rfc-editor.org/rfc/rfc8288)) in HTTP responses from its primary endpoints
- The link relation type `operational-state` identifies operational-state resources
- Consumers parse Link headers to discover available resources

**Link relation type:** `operational-state`

**Format:**

```http
Link: <https://api.example.com/health>; rel="operational-state"
```

With optional profile parameter:

```http
Link: <https://api.example.com/health>; rel="operational-state"; profile="health"
Link: <https://api.example.com/status>; rel="operational-state"; profile="status"
```

**Requirements:**

- Targets SHOULD include at least one `Link` header with `rel="operational-state"` in responses from primary API endpoints
- The `profile` parameter is OPTIONAL — when present, it indicates which profile the linked resource implements
- Multiple `Link` headers with `rel="operational-state"` MAY be present, pointing to different resources (e.g., different profiles, different serializations)

### 2. Well-Known Path (Baseline Fallback)

A predictable URI path serving as a bootstrap discovery point.

**Well-known path:** `/.well-known/operational-state`

This path MUST serve a **discovery document** (see [Discovery Document Format](#discovery-document-format)), not necessarily the operational-state payload itself.

**Requirements:**

- Targets SHOULD serve the well-known path if they expose any operational-state resources
- The well-known resource MUST be safe to probe (no side effects, lightweight)
- The well-known resource SHOULD be publicly accessible without authentication
- The well-known resource MUST return `application/json` content type

### 3. Inline References

Operational-state resources linking to related resources within their own responses.

**How it works:**

- A shallow endpoint (e.g., liveness) includes links to deeper endpoints (e.g., health, status) via the `links` field in the response body
- Resources advertise related resources at different depth levels
- Consumers can navigate from a simple entry point to richer operational-state information

**Requirements:**

- Inline references SHOULD use the `operational-state` link relation
- The `links` object in serialization responses MAY include references to related operational-state resources

### 4. DNS Bootstrap (Optional)

DNS TXT records for domain-level discovery.

**How it works:**

- A domain MAY publish a TXT record at `_operational-state.<domain>` indicating the location of the well-known discovery document
- Ecosystem-aware clients MAY check DNS before making HTTP requests

**Format:**

```
_operational-state.example.com. IN TXT "v=oos1; url=https://api.example.com/.well-known/operational-state"
```

**Constraints (normative):**

- Discovery MUST NOT depend solely on DNS
- Any resource discoverable via DNS MUST also be discoverable through HTTP-based mechanisms
- DNS is an optimization and an ownership signal, not a required dependency
- Exact TXT record format beyond the above example is informational in v1

---

## Discovery Document Format

The well-known path and standalone discovery endpoints MUST serve a discovery document with the following JSON structure:

```json
{
  "version": "1.0",
  "subject": {
    "id": "<subject-identity>",
    "description": "<human-readable-description>"
  },
  "resources": [
    {
      "href": "<url>",
      "profiles": ["<profile-id>", ...],
      "serialization": "<content-type>",
      "auth": "none" | "required",
      "description": "<human-readable-description>"
    }
  ]
}
```

### Discovery Document Fields

#### `version` (REQUIRED)

Discovery document format version. MUST be `"1.0"` for v1.

#### `subject` (REQUIRED)

Object identifying the service this discovery document describes.

- `subject.id` (REQUIRED) — same identity used in operational-state responses
- `subject.description` (OPTIONAL) — human-readable name

#### `resources` (REQUIRED)

Array of available operational-state resources.

Each resource entry:

- `href` (REQUIRED) — absolute URL of the operational-state endpoint
- `profiles` (REQUIRED) — array of profile identifiers this resource implements
- `serialization` (REQUIRED) — content type served by this resource
- `auth` (RECOMMENDED) — `"none"` if publicly accessible, `"required"` if authentication is needed
- `description` (OPTIONAL) — human-readable description of the resource

### Discovery Document Example

```json
{
  "version": "1.0",
  "subject": {
    "id": "payment-platform",
    "description": "Payment Processing Platform"
  },
  "resources": [
    {
      "href": "https://api.example.com/health",
      "profiles": ["liveness", "readiness", "health"],
      "serialization": "application/health+json",
      "auth": "none",
      "description": "Public health endpoint"
    },
    {
      "href": "https://api.example.com/status",
      "profiles": ["status"],
      "serialization": "application/status+json",
      "auth": "required",
      "description": "Authenticated full status endpoint"
    }
  ]
}
```

---

## Multi-Service / Multi-Tenant Discovery

When a single origin hosts multiple services or tenants:

- **Path-scoped discovery** — different services at different path prefixes MAY have separate well-known paths (e.g., `/.well-known/operational-state` serves a discovery document enumerating all services)
- **Host-based separation** — virtual hosts or subdomains SHOULD each have their own discovery document
- **Aggregation** — a single discovery document MAY enumerate multiple services by including multiple subjects with separate resource entries

---

## Public / Private Discovery Split

Discovery MUST support the distinction between public and authenticated operational-state resources:

- **Public resources SHOULD exist where possible** — they lower adoption friction and enable basic interoperability
- **Authenticated resources SHOULD extend, not replace, public ones** — deeper detail behind auth, not hidden basics
- Discovery documents SHOULD indicate the existence of authenticated endpoints (via the `auth` field) without exposing their content
- A monitor SHOULD be able to discover that richer information is available behind authentication, then decide whether to authenticate

---

## References

- [ARCHITECTURE.md](../ARCHITECTURE.md) — Layer 5 definition
- [capabilities.md](capabilities.md) — the layer that describes discovered resources
- [RFC 8288](https://www.rfc-editor.org/rfc/rfc8288) — Web Linking
- [RFC 8615](https://www.rfc-editor.org/rfc/rfc8615) — Well-Known URIs
- [Design Note 004: Discovery Mechanism Hierarchy](../design-notes/004-discovery-mechanism-hierarchy.md) — rationale for priority order
