# Specification Documents

This directory contains the normative specification documents for each layer of the Open Operational State architecture. All documents are **draft specifications** using RFC 2119 normative language.

## Reading Order

These documents should be read in layer order, starting from the foundational core model:

| Document | Layer | Status |
|---|---|---|
| [core-model.md](core-model.md) | 1. Core Model | Draft specification |
| [condition-vocabularies.md](condition-vocabularies.md) | 1. Core Model | Draft specification |
| [profiles.md](profiles.md) | 2. Profiles | Draft specification |
| [serializations.md](serializations.md) | 3. Serializations | Draft specification |
| [serializations/health-response.md](serializations/health-response.md) | 3. Serializations | Draft specification |
| [serializations/service-status.md](serializations/service-status.md) | 3. Serializations | Draft specification |
| [serializations/http-status-only.md](serializations/http-status-only.md) | 3. Serializations | Draft specification |
| [adapters.md](adapters.md) | 4. Adapters | Draft specification |
| [adapters/plain-http.md](adapters/plain-http.md) | 4. Adapters | Draft specification |
| [adapters/health-check-draft.md](adapters/health-check-draft.md) | 4. Adapters | Draft specification |
| [discovery.md](discovery.md) | 5. Discovery | Draft specification |
| [capabilities.md](capabilities.md) | 6. Capabilities / Negotiation | Draft specification |

## Supporting Material

- [ARCHITECTURE.md](../ARCHITECTURE.md) — the six-layer architecture that structures the entire standard
- [Glossary](https://github.com/open-operational-state/governance/blob/main/GLOSSARY.md) — controlled terminology
- [Terminology](../terminology/) — spec-level applied usage and context
- [Design Notes](../design-notes/) — rationale for key architectural and naming decisions

## Specification Status

All documents in this directory are **Phase 3 draft specifications**. They contain normative requirements (MUST/SHOULD/MAY per RFC 2119), locked vocabulary values, concrete JSON shapes, and testable conformance criteria. See the [Roadmap](https://github.com/open-operational-state/governance/blob/main/ROADMAP.md) for phase definitions.
