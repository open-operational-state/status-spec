# Status Spec

The technical specification for [Open Operational State](https://github.com/open-operational-state) — a vendor-neutral standard for machine-readable operational state of web services.

## Overview

Today, every framework and platform exposes operational state differently — Spring Boot, Kubernetes, custom health endpoints — with no shared model for what the values mean. This repository defines how web services communicate their operational condition — health, readiness, liveness, status — in a machine-readable, interoperable way. It provides a unifying semantic model for health check endpoints, readiness and liveness probes, and service status APIs, built around a six-layer extensible architecture.

## Reading Guide

Start here:

1. **[ARCHITECTURE.md](ARCHITECTURE.md)** — the six-layer architecture that structures the entire standard
2. **[NON_GOALS.md](NON_GOALS.md)** — what this standard explicitly is not
3. **[PRIOR-ART.md](PRIOR-ART.md)** — influences and compatibility targets

Then explore the specification documents:

| Document | Layer | Description |
|---|---|---|
| [spec/core-model.md](spec/core-model.md) | Core Model | Stable, transport-agnostic semantics |
| [spec/condition-vocabularies.md](spec/condition-vocabularies.md) | Core Model | Condition values per profile with ecosystem mappings |
| [spec/profiles.md](spec/profiles.md) | Profiles | Domain-specific specializations (Liveness, Readiness, Health, Status) |
| [spec/serializations.md](spec/serializations.md) | Serializations | Wire-level representation overview |
| [spec/serializations/health-response.md](spec/serializations/health-response.md) | Serializations | `application/health+json` format |
| [spec/serializations/service-status.md](spec/serializations/service-status.md) | Serializations | `application/status+json` format |
| [spec/serializations/http-status-only.md](spec/serializations/http-status-only.md) | Serializations | Minimal HTTP-only serialization |
| [spec/adapters.md](spec/adapters.md) | Adapters | Adapter specification framework |
| [spec/adapters/plain-http.md](spec/adapters/plain-http.md) | Adapters | Plain HTTP status code adapter |
| [spec/adapters/health-check-draft.md](spec/adapters/health-check-draft.md) | Adapters | draft-inadarei health check adapter |
| [spec/discovery.md](spec/discovery.md) | Discovery | Locating operational-state resources |
| [spec/capabilities.md](spec/capabilities.md) | Capabilities | Negotiation and feature advertisement |

## Supporting Material

| Directory | Purpose |
|---|---|
| [terminology/](terminology/) | Applied spec-level usage and context (authoritative glossary is in [governance](https://github.com/open-operational-state/governance/blob/main/GLOSSARY.md)) |
| [design-notes/](design-notes/) | Design rationale and working notes |
| [examples/](examples/) | Example payloads and scenarios |

## Relationship to Existing Work

Open Operational State builds on prior work in machine-readable service health and status, including:

- **[Health Check Response Format for HTTP APIs](https://datatracker.ietf.org/doc/draft-inadarei-api-health-check/)** (`draft-inadarei-api-health-check`) — a widely adopted format for HTTP API health checks
- **[Service Status Resource Format for Web Services](https://datatracker.ietf.org/doc/draft-dallariva-web-service-status-json/)** (`draft-dallariva-web-service-status-json`) — a proposal for richer, structured service status resources

These approaches represent different models of expressing operational state. Concepts from both are commonly seen in production systems, such as health endpoints exposed by frameworks like [Spring Boot Actuator](https://docs.spring.io/spring-boot/reference/actuator/endpoints.html) and [readiness/liveness probes in Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/). This project provides a unifying architecture that allows these formats to be interpreted through a common model. See [PRIOR-ART.md](PRIOR-ART.md) for detailed analysis.

## Related Repositories

| Repository | Purpose |
|---|---|
| [governance](https://github.com/open-operational-state/governance) | Charter, governance model, authoritative glossary |
| [status-conformance](https://github.com/open-operational-state/status-conformance) | Conformance definitions and test taxonomy |
| [status-tooling](https://github.com/open-operational-state/status-tooling) | Vendor-neutral reference tooling |

## Project Rules

See [PROJECT_RULES.md](PROJECT_RULES.md) for repo-specific constraints. This repository is **markdown only** — no code.

## License

This repository is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). See [LICENSE](LICENSE).
