# Status Spec

The technical specification for [Open Operational State](https://github.com/open-operational-state) — a vendor-neutral standard for machine-readable operational state of web services.

## Overview

This repository defines how web services communicate their operational condition — health, readiness, liveness, status — in a machine-readable, interoperable way. The standard is built around a six-layer extensible architecture.

## Reading Guide

Start here:

1. **[ARCHITECTURE.md](ARCHITECTURE.md)** — the six-layer architecture that structures the entire standard
2. **[NON_GOALS.md](NON_GOALS.md)** — what this standard explicitly is not
3. **[PRIOR-ART.md](PRIOR-ART.md)** — influences and compatibility targets

Then explore the specification documents:

| Document | Layer | Description |
|---|---|---|
| [spec/core-model.md](spec/core-model.md) | Core Model | Stable, transport-agnostic semantics |
| [spec/profiles.md](spec/profiles.md) | Profiles | Domain-specific specializations |
| [spec/serializations.md](spec/serializations.md) | Serializations | Wire-level representations |
| [spec/adapters.md](spec/adapters.md) | Adapters | Bridges for existing formats |
| [spec/discovery.md](spec/discovery.md) | Discovery | Locating operational-state resources |
| [spec/capabilities.md](spec/capabilities.md) | Capabilities | Negotiation and feature advertisement |

## Supporting Material

| Directory | Purpose |
|---|---|
| [terminology/](terminology/) | Applied spec-level usage and context (authoritative glossary is in [governance](https://github.com/open-operational-state/governance/blob/main/GLOSSARY.md)) |
| [design-notes/](design-notes/) | Design rationale and working notes |
| [examples/](examples/) | Example payloads and scenarios |

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
