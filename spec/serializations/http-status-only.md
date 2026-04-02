# HTTP Status Only Serialization

> **Status:** Draft specification — Phase 3.

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## Overview

The HTTP-status-only serialization represents the minimal viable operational-state signal: an HTTP response with no body (or an ignored body), where the condition is derived entirely from the HTTP status code and optional headers.

This is the most common "health check" pattern in the wild — a 200 means healthy, a 503 means unhealthy. Despite its simplicity, this serialization is essential because it is the **universal fallback** for targets that expose no structured operational-state endpoint.

**Content type:** None (no body required)

**Profile coverage:** Liveness only

---

## Condition Mapping

| HTTP Response | Condition |
|---|---|
| Connection successful + 2xx status code | `alive` |
| Connection successful + 3xx status code | `alive` |
| Connection successful + 4xx status code | `alive` |
| Connection successful + 5xx status code | `alive` |
| Connection failure (timeout, refused, DNS error) | `unreachable` |

**Key semantic:** The HTTP-status-only serialization assesses **reachability**, not operational quality. Any response — even a 500 Internal Server Error — proves the service is listening and responsive. Only a failure to establish a connection maps to `unreachable`.

---

## Optional Header Signals

The following HTTP headers MAY provide supplementary information when present:

| Header | Core Model Mapping |
|---|---|
| `Date` | Timing (report time) |
| `Retry-After` | Informational — suggests when to re-check |
| `Link` | Discovery — MAY point to richer operational-state resources |

---

## Core Model Mapping

| Core Model Concept | Representation | Coverage |
|---|---|---|
| Subject | Implicit: the resource being requested | Partial (no explicit identity) |
| Condition | Derived from connection success/failure | Liveness only |
| Timing | HTTP `Date` header if present | Partial |
| Evidence | Not representable | None |
| Components | Not representable | None |
| Dependencies | Not representable | None |
| Provenance | Implied: externally observed | Inferred only |
| Scope | Implicit: whole-service | Inferred only |

### Declared Limitations

This is the most lossy serialization. It can represent only:

- Subject (implicitly, via the URL being probed)
- Condition (binary: `alive` / `unreachable`)
- Limited timing (via HTTP `Date` header)

It MUST NOT be treated as a conformant serialization for profiles beyond Liveness. Consumers MUST NOT treat an HTTP-status-only response as equivalent to a richer serialization that returns the same condition.

---

## Conformance

A target providing only HTTP-status-code-based operational state satisfies **Basic conformance at the Liveness profile** and no higher.

This serialization exists primarily as an **adapter target** — it provides a standard way for monitors to interpret existing HTTP endpoints using core model semantics, even when those endpoints were not designed with this standard in mind.

---

## References

- [serializations.md](../serializations.md) — serialization overview and constraints
- [core-model.md](../core-model.md) — the model this serialization maps to (minimally)
- [condition-vocabularies.md](../condition-vocabularies.md#liveness-vocabulary) — Liveness vocabulary
