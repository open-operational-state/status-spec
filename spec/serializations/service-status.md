# Service Status Serialization

> **Status:** Draft specification — Phase 3.

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## Overview

The service-status serialization defines a rich JSON wire format for comprehensive operational-state data, informed by the `draft-dallariva-web-service-status-json-00` lineage. It provides multi-level component and dependency hierarchies, geographic scope, criticality classification, and structured incident reporting.

**Content type:** `application/status+json`

**Profile coverage:** All profiles (optimized for Status)

---

## Document Structure

A service-status document is a JSON object with the following structure:

```json
{
  "condition": "<condition-value>",
  "profiles": ["status"],
  "subject": {
    "id": "<subject-identity>",
    "description": "<human-readable-description>",
    "version": "<service-version>",
    "contact": "<contact-uri>"
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
  "scope": {
    "type": "whole-service" | "regional" | "endpoint",
    "identifier": "<scope-identifier>"
  },
  "components": [
    {
      "id": "<component-id>",
      "description": "<description>",
      "condition": "<condition-value>",
      "criticality": "critical" | "important" | "informational",
      "timing": { ... },
      "provenance": "<provenance-type>",
      "evidence": { ... },
      "components": [ ... ]
    }
  ],
  "dependencies": [
    {
      "id": "<dependency-id>",
      "description": "<description>",
      "condition": "<condition-value>",
      "criticality": "critical" | "important" | "informational",
      "timing": { ... },
      "provenance": "<provenance-type>",
      "evidence": { ... }
    }
  ],
  "incidents": [
    {
      "id": "<incident-id>",
      "title": "<human-readable-title>",
      "condition": "<condition-value>",
      "started": "<RFC-3339-timestamp>",
      "resolved": "<RFC-3339-timestamp>",
      "affectedComponents": ["<component-id>", ...],
      "updates": [
        {
          "timestamp": "<RFC-3339-timestamp>",
          "message": "<update-text>"
        }
      ]
    }
  ],
  "links": {
    "<relation>": "<uri>"
  }
}
```

---

## Field Definitions

### Top-Level Fields

#### `condition` (REQUIRED)

The overall operational condition. See [Condition Vocabularies](../condition-vocabularies.md).

#### `profiles` (REQUIRED)

Array of profile identifiers. MUST contain at least one value.

#### `subject` (REQUIRED)

Object identifying the service.

- `subject.id` (REQUIRED) — unique stable identifier
- `subject.description` (OPTIONAL) — human-readable name
- `subject.version` (OPTIONAL) — public version of the service
- `subject.contact` (OPTIONAL) — URI for operations contact (e.g., email, status page)

#### `timing` (REQUIRED for Status profile)

Same structure as defined in health-response serialization.

#### `provenance` (REQUIRED for Status profile)

Same values as defined in health-response serialization.

#### `evidence` (REQUIRED for Status profile)

Same structure as defined in health-response serialization.

#### `scope` (OPTIONAL)

Scope information for the condition statement.

- `scope.type` (REQUIRED when scope is present) — one of `"whole-service"`, `"regional"`, `"endpoint"`, or a custom scope type
- `scope.identifier` (OPTIONAL) — identifier for the specific scope (e.g., region name, endpoint path)

#### `components` (REQUIRED for Status profile)

An array of component objects representing internal constituents.

#### `dependencies` (REQUIRED for Status profile)

An array of dependency objects representing external systems.

#### `incidents` (OPTIONAL)

An array of incident objects providing structured incident reporting. This is an informational extension — `incidents` does not map to a core model concept but provides operationally valuable context.

#### `links` (OPTIONAL)

Same structure as defined in health-response serialization.

---

### Component and Dependency Object Fields

Both component and dependency objects share a common structure:

#### `id` (REQUIRED)

Unique identifier for this component or dependency. MUST be unique within the document.

#### `description` (OPTIONAL)

Human-readable description.

#### `condition` (REQUIRED)

Condition value from the Health vocabulary.

#### `criticality` (RECOMMENDED)

Classification of the component or dependency's importance to the parent service.

- `"critical"` — failure directly impacts service availability
- `"important"` — failure degrades service quality
- `"informational"` — status is reported for visibility; failure does not directly impact the service

#### `timing` (OPTIONAL)

Same structure as top-level timing.

#### `provenance` (OPTIONAL)

Same values as top-level provenance.

#### `evidence` (OPTIONAL)

Same structure as top-level evidence, with additional fields:

- `evidence.observedValue` (OPTIONAL) — measured value
- `evidence.observedUnit` (OPTIONAL) — unit of measurement

#### `components` (OPTIONAL, components only)

Nested component array. This enables **multi-level component hierarchies** — the key structural advantage over the health-response serialization. The nesting forms a DAG; circular references MUST NOT occur.

---

### Incident Object Fields

#### `id` (REQUIRED)

Unique identifier for the incident.

#### `title` (REQUIRED)

Human-readable incident title.

#### `condition` (REQUIRED)

The condition value associated with this incident.

#### `started` (REQUIRED)

RFC 3339 timestamp of incident start.

#### `resolved` (OPTIONAL)

RFC 3339 timestamp of incident resolution. Absent if ongoing.

#### `affectedComponents` (OPTIONAL)

Array of component IDs affected by this incident.

#### `updates` (OPTIONAL)

Array of update objects, each with `timestamp` and `message`.

---

## Core Model Mapping

| Core Model Concept | Field | Coverage |
|---|---|---|
| Subject | `subject.id`, `subject.description` | Full |
| Condition | `condition` (all levels) | Full |
| Timing | `timing.*` (all levels) | Full |
| Evidence | `evidence.*` (all levels) | Full |
| Components | `components` array | Full (multi-level) |
| Dependencies | `dependencies` array | Full |
| Provenance | `provenance` (all levels) | Full |
| Scope | `scope.type`, `scope.identifier` | Full |

### Declared Limitations

- **Incidents** are an informational extension not modeled in the core model. They provide operationally useful context but do not carry core model semantics.
- The `criticality` field is a serialization-level concept that informs consumption but is not a core model concept.

---

## Examples

### Full Status Response

```json
{
  "condition": "degraded",
  "profiles": ["status", "health", "readiness", "liveness"],
  "subject": {
    "id": "payment-platform",
    "description": "Payment Processing Platform",
    "version": "3.2.1",
    "contact": "mailto:ops@example.com"
  },
  "timing": {
    "observed": "2026-04-02T19:30:00Z",
    "reported": "2026-04-02T19:30:05Z",
    "stateChanged": "2026-04-02T18:45:00Z"
  },
  "provenance": "derived",
  "evidence": {
    "type": "aggregation",
    "detail": "Worst-of component conditions"
  },
  "components": [
    {
      "id": "payment-api",
      "description": "Payment API Gateway",
      "condition": "operational",
      "criticality": "critical",
      "timing": {
        "observed": "2026-04-02T19:30:00Z"
      },
      "provenance": "self-reported"
    },
    {
      "id": "fraud-engine",
      "description": "Fraud Detection Engine",
      "condition": "degraded",
      "criticality": "critical",
      "timing": {
        "observed": "2026-04-02T19:30:00Z",
        "stateChanged": "2026-04-02T18:45:00Z"
      },
      "provenance": "self-reported",
      "evidence": {
        "type": "response-time",
        "observedValue": 1250,
        "observedUnit": "ms",
        "detail": "P99 latency exceeds 1000ms threshold"
      },
      "components": [
        {
          "id": "ml-scoring",
          "description": "ML Scoring Service",
          "condition": "degraded",
          "criticality": "critical",
          "provenance": "self-reported",
          "evidence": {
            "type": "throughput",
            "observedValue": 150,
            "observedUnit": "rps",
            "detail": "Below expected 500 rps baseline"
          }
        }
      ]
    },
    {
      "id": "notification-service",
      "description": "Payment Notification Service",
      "condition": "operational",
      "criticality": "informational",
      "provenance": "self-reported"
    }
  ],
  "dependencies": [
    {
      "id": "card-network",
      "description": "Card Network Gateway",
      "condition": "operational",
      "criticality": "critical",
      "provenance": "externally-observed",
      "timing": {
        "observed": "2026-04-02T19:29:55Z"
      }
    },
    {
      "id": "bank-api",
      "description": "Partner Bank API",
      "condition": "operational",
      "criticality": "important",
      "provenance": "externally-observed"
    }
  ],
  "incidents": [
    {
      "id": "INC-2026-0402-001",
      "title": "Elevated latency in fraud detection",
      "condition": "degraded",
      "started": "2026-04-02T18:45:00Z",
      "affectedComponents": ["fraud-engine", "ml-scoring"],
      "updates": [
        {
          "timestamp": "2026-04-02T18:50:00Z",
          "message": "Investigating elevated P99 latency in ML scoring pipeline"
        },
        {
          "timestamp": "2026-04-02T19:15:00Z",
          "message": "Root cause identified: model cache invalidation storm. Mitigation in progress."
        }
      ]
    }
  ],
  "links": {
    "self": "https://status.example.com/api/v1/status",
    "operational-state": "https://status.example.com/api/v1/health"
  }
}
```

---

## HTTP Response Requirements

Same requirements as defined in the [health-response serialization](health-response.md#http-response-requirements).

---

## References

- [serializations.md](../serializations.md) — serialization overview and constraints
- [core-model.md](../core-model.md) — the model this serialization maps to
- [condition-vocabularies.md](../condition-vocabularies.md) — condition values
- [profiles.md](../profiles.md) — profile requirements
- [PRIOR-ART.md](../../PRIOR-ART.md) — draft-dallariva lineage
