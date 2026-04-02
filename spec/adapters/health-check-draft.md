# Adapter: Health Check Draft (draft-inadarei-api-health-check)

> **Status:** Draft specification — Phase 3.

## Overview

This adapter translates JSON responses conforming to the `draft-inadarei-api-health-check-06` format (`application/health+json`) into core model semantics. It serves as a **meta-adapter**: demonstrating that the prior-art format can be mapped through the core model, validating the dual-lineage architectural decision.

---

## Input Pattern Definition

### Content Type

`application/health+json` (primary) or `application/json` with matching structural markers.

### Structural Markers

The adapter activates when:

- Content type is `application/health+json`, OR
- The response body is a JSON object containing a `status` field with a value of `"pass"`, `"fail"`, or `"warn"` (case-insensitive)

### Source Context

- Endpoints implementing the `draft-inadarei-api-health-check` format
- Node.js health check libraries (Terminus pattern)
- APIs using `application/health+json` media type

### Disambiguation

- Distinguished from Spring Boot Actuator by status values: the draft uses `pass`/`fail`/`warn` while Spring Boot uses `UP`/`DOWN`/`OUT_OF_SERVICE`/`UNKNOWN`
- Distinguished from the service-status draft by structure: this format uses a flat `checks` object keyed by component name, while the service-status format uses structured arrays

---

## Mapping Table

| Source Field | Core Model Concept | Mapping Notes |
|---|---|---|
| `status` (top-level) | Condition | `"pass"` → `operational`, `"warn"` → `degraded`, `"fail"` → `down`. Case-insensitive. Aliases: `"ok"` → `operational`, `"error"` → `down`, `"up"` → `operational`, `"down"` → `down`. |
| `serviceId` | Subject (identity) | Direct mapping. If absent, subject identity is inferred from the request URL. |
| `description` | Subject (description) | Direct mapping. |
| `checks` | Components / Dependencies | Each key in the `checks` object becomes a component or dependency. See below. |
| `checks.{key}.time` | Timing (observation time) | Per-component observation time. |
| `checks.{key}.status` | Condition (per-component) | Same value mapping as top-level `status`. |
| `checks.{key}.observedValue` | Evidence (observed value) | Direct mapping to `evidence.observedValue`. |
| `checks.{key}.observedUnit` | Evidence (observed unit) | Direct mapping to `evidence.observedUnit`. |
| `checks.{key}.output` | Evidence (detail) | Maps to `evidence.detail`. |
| `checks.{key}.componentId` | Subject identity (per-component) | Per-component unique identifier. |
| `checks.{key}.componentType` | Role inference | `"system"` or `"datastore"` → `"dependency"`; `"component"` → `"component"`. Heuristic. |
| `checks.{key}.affectedEndpoints` | Scope (endpoint-specific) | Partial mapping — indicates affected endpoints but scope model is richer. |
| `links` | Links | Direct mapping to `links` object. |
| `version` | *(not mapped)* | Informational metadata, no core model counterpart. |
| `releaseId` | *(not mapped)* | Informational metadata, no core model counterpart. |
| `notes` | *(not mapped)* | Informational metadata, no core model counterpart. |
| `output` (top-level) | Evidence (detail) | Maps to top-level `evidence.detail`. |
| *(not present)* | Provenance | Inferred as `self-reported` — the target produced this response. |
| *(not present)* | Timing (report time) | Inferred from HTTP `Date` header if present. |

### Checks Key Parsing

The draft defines check keys as `"{componentName}:{measurementName}"`. The adapter SHOULD:

- Use `componentName` as the check entry key (component identity)
- Use `measurementName` as `evidence.type`
- Group multiple check entries with the same `componentName` under the same component, preserving each as a separate evidence entry

---

## Lossiness Classification

| Core Model Concept | Classification |
|---|---|
| Subject | **Lossless** when `serviceId` is present; **Inferred** when absent |
| Condition | **Lossless** — `pass`/`warn`/`fail` map cleanly to the Health vocabulary |
| Timing | **Partially lossy** — only observation time per-component; no top-level observation time, no state-change time |
| Evidence | **Lossless** within the draft's evidence model (`observedValue`, `observedUnit`, `output`) |
| Components | **Heuristic** — components are identified from `checks` keys; the component/dependency distinction relies on `componentType` inference |
| Dependencies | **Heuristic** — inferred from `componentType` values |
| Provenance | **Inferred** — always assumed `self-reported` |
| Scope | **Partially lossy** — `affectedEndpoints` provides partial scope information but does not map cleanly to the scope model |

---

## Profile Coverage

| Profile | Satisfiable? | Notes |
|---|---|---|
| Liveness | **Yes** | `status` field provides binary pass/fail signal |
| Readiness | **Yes** | `status` with dependency checks can indicate readiness |
| Health | **Yes** | Full condition, component-level checks, evidence |
| Status | **Partial** | Can provide condition, components, and evidence; lacks explicit provenance, scope, and multi-level nesting |

---

## Limitations

- **No explicit provenance** — the draft assumes self-reported; the adapter cannot determine if the response was generated by the target or an intermediary
- **No component/dependency semantic distinction** — the draft's `componentType` field provides hints but not a definitive mapping; `"system"` could be internal or external
- **No multi-level nesting** — `checks` is flat; deeply nested component trees cannot be represented
- **No scope model** — `affectedEndpoints` is the closest equivalent but is limited to endpoint-level scope
- **Alias handling** — the draft permits aliases (`"ok"`, `"error"`, `"up"`, `"down"`) which the adapter MUST normalize to canonical vocabulary values
- **Multiple check entries per component** — the draft allows multiple array entries for the same check key (e.g., per-node); the adapter SHOULD represent each as a separate evidence entry under the same component

---

## Examples

### Basic Health Response

**Input:**

```json
{
  "status": "pass",
  "serviceId": "authz-service",
  "description": "Authorization Service"
}
```

**Core model output:**

```json
{
  "condition": "operational",
  "profiles": ["liveness", "readiness", "health"],
  "subject": {
    "id": "authz-service",
    "description": "Authorization Service"
  },
  "provenance": "self-reported"
}
```

### Rich Health Response with Components

**Input:**

```json
{
  "status": "warn",
  "serviceId": "authz-service",
  "description": "Authorization Service",
  "checks": {
    "cassandra:responseTime": [
      {
        "componentId": "dfd6cf2b",
        "componentType": "datastore",
        "observedValue": 250,
        "observedUnit": "ms",
        "status": "pass",
        "time": "2026-04-02T19:30:00Z"
      }
    ],
    "cpu:utilization": [
      {
        "componentType": "system",
        "observedValue": 85,
        "observedUnit": "percent",
        "status": "warn",
        "time": "2026-04-02T19:30:00Z",
        "output": "CPU utilization above 80% threshold"
      }
    ]
  }
}
```

**Core model output:**

```json
{
  "condition": "degraded",
  "profiles": ["health"],
  "subject": {
    "id": "authz-service",
    "description": "Authorization Service"
  },
  "provenance": "self-reported",
  "checks": {
    "cassandra": {
      "condition": "operational",
      "role": "dependency",
      "timing": {
        "observed": "2026-04-02T19:30:00Z"
      },
      "evidence": {
        "type": "responseTime",
        "observedValue": 250,
        "observedUnit": "ms"
      }
    },
    "cpu": {
      "condition": "degraded",
      "role": "component",
      "timing": {
        "observed": "2026-04-02T19:30:00Z"
      },
      "evidence": {
        "type": "utilization",
        "observedValue": 85,
        "observedUnit": "percent",
        "detail": "CPU utilization above 80% threshold"
      }
    }
  }
}
```

---

## References

- [adapters.md](../adapters.md) — adapter specification framework
- [core-model.md](../core-model.md) — the model this adapter targets
- [condition-vocabularies.md](../condition-vocabularies.md) — condition values and ecosystem mapping
- [PRIOR-ART.md](../../PRIOR-ART.md) — draft-inadarei lineage
- [serializations/health-response.md](../serializations/health-response.md) — the health-response serialization (related but distinct)
