# Prior Art

This document catalogs prior art that has influenced the design of the Open Operational State standard. These are treated as **influences and compatibility targets**, not as normative foundations.

---

## Internet-Drafts

### draft-inadarei-api-health-check

**Title:** Health Check Response Format for HTTP APIs  
**Author:** Irakli Nadareishvili  
**Latest version reviewed:** draft-inadarei-api-health-check-06  
**Status:** Expired individual Internet-Draft  
**Reference:** https://datatracker.ietf.org/doc/html/draft-inadarei-api-health-check-06

This draft defines a JSON response format for health-check endpoints in HTTP APIs. It represents the "health-response lineage" — a focused, single-resource approach to communicating service health.

**Relevance to this standard:**

- Demonstrates the demand for machine-readable health information
- Defines a useful baseline vocabulary (status values, component checks)
- Informs the design of at least one serialization profile
- Its limitations (single format, limited extensibility) motivate the multi-serialization architecture

### draft-dallariva-web-service-status-json

**Title:** Service Status Resource Format for Web Services  
**Author:** Arthur Dallariva  
**Latest version reviewed:** draft-dallariva-web-service-status-json-00  
**Status:** Internet-Draft  
**Reference:** https://datatracker.ietf.org/doc/draft-dallariva-web-service-status-json/00/

This draft defines a status resource format for web services with richer structure than the health-check draft. It represents the "service-status lineage" — a more comprehensive approach to service condition.

**Relevance to this standard:**

- Demonstrates the need for richer operational-state representations
- Introduces structural concepts that inform profile and serialization design
- Its structural differences from the health-check draft validate the multi-serialization approach

## Key Structural Insight

These two drafts cannot be collapsed into a single strict universal wire format without structural conflicts. This insight directly motivated the architectural decision to use:

- One core model
- Multiple normative serializations / profiles
- Clear mappings between them

Rather than a single hybrid payload. See [ARCHITECTURE.md](ARCHITECTURE.md) for details.

---

## Industry Patterns

The following industry patterns and implementations have also informed the design:

### Framework Health Endpoints

- **Spring Boot Actuator** (`/actuator/health`) — structured health with component-level detail
- **ASP.NET Health Checks** — middleware-based health reporting with configurable checks
- **Rails Health Check** (`/up`) — minimal liveness-style endpoint
- **Express.js patterns** — various community conventions for health routes

### Platform Probes

- **Kubernetes liveness, readiness, and startup probes** — the most widely deployed operational-state convention, demonstrating the importance of profile distinctions (liveness vs. readiness vs. startup)
- **Cloud provider health checks** (AWS ALB/NLB, GCP health checks, Azure health probes) — platform-level consumption of basic health signals

### Monitoring Conventions

- **Uptime monitoring services** — external observation of HTTP status codes and response content
- **Status page conventions** (Atlassian Statuspage, Instatus, etc.) — human-readable status communication with machine-readable feeds

### Standards-Adjacent Work

- **RFC 7234** — HTTP caching, relevant to freshness and timing
- **RFC 8288** — Web Linking, relevant to link-based discovery
- **RFC 8615** — Well-Known URIs, relevant to predictable resource location
- **OpenTelemetry** — complementary observability standard (not a target for unification, but context)

---

## Relationship to This Standard

All prior art listed here is treated as:

- **Influence** — informing design and vocabulary choices
- **Compatibility target** — the standard should be able to map to/from these where practical
- **Not normative** — no prior art document is a required foundation for the v1 standard

Schemas and field names are not copied from prior drafts. The architecture provides its own model and uses adapters and serializations to bridge to existing formats.
