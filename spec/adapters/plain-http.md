# Adapter: Plain HTTP Status Code

> **Status:** Draft specification — Phase 3.

## Overview

This adapter translates a plain HTTP response (status code only, no structured body) into core model semantics. It is the most common operational-state pattern in the wild and serves as the **universal fallback adapter**.

---

## Input Pattern Definition

### Content Type

Any content type or no content type. The body is ignored.

### Structural Markers

No body structure is required. This adapter activates when:

- An HTTP response is received from a probed endpoint
- The response does not match any other adapter's structural markers (i.e., this adapter is the fallback)

### Source Context

- Generic HTTP health check endpoints (`/health`, `/ping`, `/healthz`, etc.)
- Any HTTP endpoint probed for availability
- Load balancer health check targets
- Kubernetes liveness/readiness probes without structured bodies

### Disambiguation

This adapter MUST be used only when no structured body format is recognized. If the response body matches `application/health+json` or another known format, the corresponding specific adapter takes precedence.

---

## Mapping Table

| Source Signal | Core Model Concept | Mapping Notes |
|---|---|---|
| Connection success + any HTTP status code | Subject | Inferred: the URL being probed. Subject identity is the request URL. |
| Connection success | Condition | `alive` (Liveness vocabulary) |
| Connection failure (timeout, refused, DNS) | Condition | `unreachable` (Liveness vocabulary) |
| HTTP `Date` header | Timing (report time) | Mapped if present |
| *(no source)* | Timing (observation time) | Inferred: the time the probe was initiated |
| *(no source)* | Evidence | Not mapped |
| *(no source)* | Components | Not mapped |
| *(no source)* | Dependencies | Not mapped |
| *(no source)* | Provenance | Inferred: `externally-observed` (the monitor performed the probe) |
| *(no source)* | Scope | Inferred: `whole-service` |

---

## Lossiness Classification

| Core Model Concept | Classification |
|---|---|
| Subject | **Inferred** — derived from the probe URL, not explicitly declared by the target |
| Condition | **Heuristic** — binary alive/unreachable derived from connection success/failure |
| Timing | **Partially lossy** — only report time available via `Date` header; observation time inferred |
| Evidence | **Not mapped** — no evidence available |
| Components | **Not mapped** |
| Dependencies | **Not mapped** |
| Provenance | **Inferred** — always `externally-observed` since the adapter processes probe results |
| Scope | **Inferred** — always whole-service |

---

## Profile Coverage

| Profile | Satisfiable? | Notes |
|---|---|---|
| Liveness | **Yes** | Binary alive/unreachable maps directly |
| Readiness | **No** | Cannot distinguish ready from alive |
| Health | **No** | Cannot determine degradation, components, or evidence |
| Status | **No** | Cannot provide any Status-level information |

---

## Limitations

- Cannot determine **why** a service is unreachable — connection refused, DNS failure, and timeout are all mapped to `unreachable`
- A 500 Internal Server Error is mapped to `alive` — the service responded, therefore it is reachable. This is semantically correct for Liveness but may surprise users expecting health-level semantics
- Cannot distinguish between a health endpoint and any other URL — the adapter treats all URLs identically
- Subject identity is the probe URL, which may not be the canonical service identity

---

## Examples

### Successful Probe

**Input:**

```http
HTTP/1.1 200 OK
Date: Wed, 02 Apr 2026 19:30:00 GMT
Content-Type: text/plain

OK
```

**Core model output:**

```json
{
  "condition": "alive",
  "profiles": ["liveness"],
  "subject": {
    "id": "https://api.example.com/health"
  },
  "timing": {
    "observed": "2026-04-02T19:30:00Z",
    "reported": "2026-04-02T19:30:00Z"
  },
  "provenance": "externally-observed"
}
```

### Failed Probe (Connection Refused)

**Input:** Connection refused — no HTTP response received.

**Core model output:**

```json
{
  "condition": "unreachable",
  "profiles": ["liveness"],
  "subject": {
    "id": "https://api.example.com/health"
  },
  "timing": {
    "observed": "2026-04-02T19:30:00Z"
  },
  "provenance": "externally-observed"
}
```

### Server Error (Still Alive)

**Input:**

```http
HTTP/1.1 500 Internal Server Error
Date: Wed, 02 Apr 2026 19:30:00 GMT
```

**Core model output:**

```json
{
  "condition": "alive",
  "profiles": ["liveness"],
  "subject": {
    "id": "https://api.example.com/health"
  },
  "timing": {
    "observed": "2026-04-02T19:30:00Z",
    "reported": "2026-04-02T19:30:00Z"
  },
  "provenance": "externally-observed"
}
```

---

## References

- [adapters.md](../adapters.md) — adapter specification framework
- [core-model.md](../core-model.md) — the model this adapter targets
- [condition-vocabularies.md](../condition-vocabularies.md#liveness-vocabulary) — Liveness vocabulary
- [serializations/http-status-only.md](../serializations/http-status-only.md) — the related serialization
