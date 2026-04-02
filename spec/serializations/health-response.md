# Health Response Serialization

> **Status:** Draft specification — Phase 3.

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## Overview

The health-response serialization defines a JSON wire format for operational-state data, informed by the `draft-inadarei-api-health-check-06` lineage. It is a flat, single-resource-oriented structure optimized for the Liveness, Readiness, and Health profiles.

**Content type:** `application/health+json`

**Profile coverage:** Liveness, Readiness, Health (partial Status support)

---

## Document Structure

A health-response document is a JSON object with the following structure:

```json
{
  "condition": "<condition-value>",
  "profiles": ["<profile-id>", ...],
  "subject": {
    "id": "<subject-identity>",
    "description": "<human-readable-description>"
  },
  "timing": {
    "observed": "<RFC-3339-timestamp>",
    "reported": "<RFC-3339-timestamp>",
    "stateChanged": "<RFC-3339-timestamp>"
  },
  "provenance": "<provenance-type>",
  "evidence": {
    "type": "<evidence-type>",
    "detail": "<supplementary-information>"
  },
  "checks": {
    "<component-or-dependency-key>": {
      "condition": "<condition-value>",
      "role": "component" | "dependency",
      "timing": { ... },
      "provenance": "<provenance-type>",
      "evidence": {
        "type": "<evidence-type>",
        "observedValue": <any>,
        "observedUnit": "<unit>",
        "detail": "<supplementary-information>"
      }
    }
  },
  "links": {
    "<relation>": "<uri>"
  }
}
```

---

## Field Definitions

### Top-Level Fields

#### `condition` (REQUIRED)

The operational condition of the subject. MUST be a value from the condition vocabulary appropriate to the declared profile(s). See [Condition Vocabularies](../condition-vocabularies.md).

- Type: string
- Example: `"operational"`, `"degraded"`, `"alive"`

#### `profiles` (REQUIRED)

An array of profile identifiers that this response declares conformance to. MUST contain at least one profile identifier.

- Type: array of strings
- Valid values: `"liveness"`, `"readiness"`, `"health"`, `"status"`
- Example: `["health"]`, `["liveness", "readiness"]`

#### `subject` (REQUIRED)

An object identifying the entity whose operational state is described.

- `subject.id` (REQUIRED) — unique, stable identifier for the subject. Type: string.
- `subject.description` (OPTIONAL) — human-readable description. Type: string.

#### `timing` (CONDITIONAL)

Temporal metadata. MUST be present when the declared profiles include Health or Status. MAY be present for Liveness and Readiness.

- `timing.observed` (RECOMMENDED) — RFC 3339 timestamp of when the condition was observed
- `timing.reported` (OPTIONAL) — RFC 3339 timestamp of when the condition was communicated
- `timing.stateChanged` (OPTIONAL) — RFC 3339 timestamp of the last condition transition

#### `provenance` (CONDITIONAL)

The provenance type for the top-level condition. SHOULD be present when the declared profiles include Health. MUST be present when Status is declared.

- Type: string
- Valid values: `"self-reported"`, `"externally-observed"`, `"derived"`, `"manually-declared"`

#### `evidence` (CONDITIONAL)

Evidence supporting the top-level condition. SHOULD be present for Health when condition is not `operational`. MUST be present for Status.

- `evidence.type` (REQUIRED when evidence is present) — the kind of check or observation. Type: string.
- `evidence.detail` (OPTIONAL) — supplementary information. Type: string.

#### `checks` (CONDITIONAL)

An object containing component and dependency conditions. MAY be present for Health. MUST be present for Status.

Each key in the `checks` object is a unique identifier for a component or dependency. The value is a check object (see below).

#### `links` (OPTIONAL)

An object containing link relations and URIs for related resources.

- Keys SHOULD be registered link relation types or URIs per [RFC 8288](https://www.rfc-editor.org/rfc/rfc8288)
- Values MUST be URIs
- A `self` link MAY be included

---

### Check Object Fields

Each entry in the `checks` object MUST contain:

#### `condition` (REQUIRED)

Condition value for this component or dependency.

#### `role` (RECOMMENDED)

Whether this entry represents a `"component"` (internal) or `"dependency"` (external). SHOULD be present to enable the component/dependency distinction defined in the core model.

- Type: string
- Valid values: `"component"`, `"dependency"`
- Default (if absent): `"component"`

#### Additional Check Fields

The following fields follow the same definitions as their top-level counterparts:

- `timing` (OPTIONAL)
- `provenance` (OPTIONAL)
- `evidence` (OPTIONAL) — MAY additionally include:
  - `evidence.observedValue` (OPTIONAL) — the measured value. Type: any valid JSON value.
  - `evidence.observedUnit` (OPTIONAL) — the unit of measurement. Type: string.

---

## Core Model Mapping

| Core Model Concept | Field | Coverage |
|---|---|---|
| Subject | `subject.id`, `subject.description` | Full |
| Condition | `condition` (top-level and per-check) | Full |
| Timing | `timing.observed`, `timing.reported`, `timing.stateChanged` | Full |
| Evidence | `evidence.type`, `evidence.detail`, `evidence.observedValue`, `evidence.observedUnit` | Full |
| Components | `checks` entries with `role: "component"` | Full |
| Dependencies | `checks` entries with `role: "dependency"` | Full |
| Provenance | `provenance` | Full |
| Scope | Not directly represented | Limitation |

### Declared Limitations

- **Scope** is not directly representable. The health-response format assumes whole-service scope at the top level and component/dependency scope within checks. Regional or endpoint-specific scope cannot be expressed.
- **Nested component trees** beyond one level are not structurally supported. A check entry cannot contain its own `checks`. Deeply nested structures require the service-status serialization.

---

## Examples

### Liveness Response

```json
{
  "condition": "alive",
  "profiles": ["liveness"],
  "subject": {
    "id": "api-gateway"
  }
}
```

### Readiness Response

```json
{
  "condition": "ready",
  "profiles": ["readiness"],
  "subject": {
    "id": "api-gateway",
    "description": "API Gateway Service"
  },
  "timing": {
    "observed": "2026-04-02T19:30:00Z"
  }
}
```

### Health Response

```json
{
  "condition": "degraded",
  "profiles": ["health"],
  "subject": {
    "id": "authz-service",
    "description": "Authorization Service"
  },
  "timing": {
    "observed": "2026-04-02T19:30:00Z",
    "stateChanged": "2026-04-02T19:15:00Z"
  },
  "provenance": "self-reported",
  "evidence": {
    "type": "error-rate",
    "detail": "Error rate exceeds 5% threshold"
  },
  "checks": {
    "primary-database": {
      "condition": "operational",
      "role": "dependency",
      "evidence": {
        "type": "response-time",
        "observedValue": 42,
        "observedUnit": "ms"
      },
      "timing": {
        "observed": "2026-04-02T19:30:00Z"
      }
    },
    "cache-cluster": {
      "condition": "down",
      "role": "component",
      "evidence": {
        "type": "connectivity",
        "detail": "Connection refused on all nodes"
      },
      "timing": {
        "observed": "2026-04-02T19:30:00Z"
      }
    }
  },
  "links": {
    "self": "https://api.example.com/health",
    "operational-state": "https://api.example.com/status"
  }
}
```

### Composite Profile Response

```json
{
  "condition": "operational",
  "profiles": ["liveness", "readiness", "health"],
  "subject": {
    "id": "payment-service"
  },
  "timing": {
    "observed": "2026-04-02T19:30:00Z"
  },
  "provenance": "self-reported"
}
```

---

## HTTP Response Requirements

- HTTP responses carrying this serialization MUST use the content type `application/health+json`
- For `operational` / `alive` / `ready` conditions, HTTP response status SHOULD be in the 2xx range
- For `down` / `unreachable` / `not-ready` conditions, HTTP response status SHOULD be 503 (Service Unavailable)
- For `degraded` / `partial-outage` / `major-outage` conditions, HTTP response status SHOULD be 200 (the service is responding; machine-readable condition conveys the nuance)
- For `maintenance` conditions, HTTP response status SHOULD be 503 with a `Retry-After` header if the maintenance window end is known
- Responses SHOULD include `Cache-Control` headers indicating appropriate caching behavior

---

## References

- [serializations.md](../serializations.md) — serialization overview and constraints
- [core-model.md](../core-model.md) — the model this serialization maps to
- [condition-vocabularies.md](../condition-vocabularies.md) — condition values
- [profiles.md](../profiles.md) — profile requirements
- [PRIOR-ART.md](../../PRIOR-ART.md) — draft-inadarei lineage
