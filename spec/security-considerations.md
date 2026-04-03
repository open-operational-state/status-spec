# Security Considerations

**Status**: Draft specification
**Layer**: Cross-cutting (applies to all layers)

This document provides operational guidance for implementers on controlling the exposure of operational-state information. It operationalizes the security principles established in the architecture (see [ARCHITECTURE.md](../ARCHITECTURE.md)).

This document does **not** define authentication mechanisms, mandate specific access control implementations, or add new conformance requirements. It guides implementers on what to expose, what to restrict, and what the risks are. This guidance is intended to reduce accidental information exposure while preserving interoperability.

---

## 1. Public Exposure Is Optional

Exposing a public operational-state endpoint is **not required** for conformance. Implementations MAY expose no public operational-state resource at all.

The standard defines the *quality and structure* of what is exposed — not *whether* it is exposed publicly. Conformance levels (Basic, Standard, Extended) apply to the correctness and completeness of an implementation's output, regardless of audience.

An implementation that serves a fully conformant health resource exclusively to authenticated internal consumers is just as conformant as one that exposes the same resource publicly.

---

## 2. Exposure Tiers

The following tiers describe how much operational detail an endpoint exposes. They provide a shared vocabulary for discussing exposure decisions.

| Tier | What Is Exposed | Typical Audience |
|---|---|---|
| **None** | No operational-state endpoint exists | N/A |
| **Condition-only** | Top-level condition (`operational`, `degraded`, `down`) | Public / unauthenticated |
| **Condition + minimal metadata** | Condition, optional coarse timing, optional coarse service identity | Public or lightly authenticated |
| **Component-level** | Components, dependencies, per-component conditions | Authenticated monitors |
| **Full diagnostic** | All fields including evidence, dependency names, internal topology | Trusted internal consumers |

These are **guidance tiers**, not conformance levels. Implementations choose their own exposure level based on their security requirements, threat model, and operational context.

Exposure tiers are **independent of profiles**. Any profile (Liveness, Readiness, Health, Status) can be exposed at any tier. Exposure tier is a **deployment choice**, not a profile property. Implementations SHOULD NOT assume that a particular profile implies a particular exposure level.

---

## 3. Sensitive Implementation Details

Operational-state responses can inadvertently reveal information useful to attackers. Implementations SHOULD evaluate whether their responses expose any of the following:

### Component and Dependency Names

Component names (e.g., `redis-primary`, `postgres-billing`) and dependency names may reveal internal service architecture, third-party vendor relationships, or infrastructure topology. Implementations SHOULD restrict detailed component and dependency information to authenticated consumers.

### Error Messages

Error messages embedded in operational-state responses may leak stack traces, file paths, internal state, or configuration details. Evidence fields SHOULD contain structured, controlled values rather than raw error output when exposed to unauthenticated consumers.

### Measured Durations and Latency Data

Response time patterns, check durations, and latency measurements may reveal infrastructure characteristics, capacity constraints, or performance bottlenecks. These are more sensitive than simple timestamps and SHOULD be restricted to authenticated consumers where infrastructure characteristics are considered sensitive.

### Timestamps

Observed and reported timestamps are generally less sensitive than duration data, but may still be useful for timing attacks in some environments. Implementations MAY omit timing information from public responses if their threat model warrants it.

### Condition Transition Patterns

Rapid condition transitions (e.g., `operational` → `degraded` → `down`) observable through repeated polling may signal attack windows or reveal ongoing incidents. This is an inherent property of exposing any condition information and cannot be fully mitigated — but restricting detail depth limits the usefulness of such observations to attackers.

---

## 4. Discovery and Capabilities Leakage

Discovery and capabilities metadata can themselves be sensitive:

- A discovery document listing internal endpoint paths reveals attack surface
- Capabilities metadata advertising authentication modes may reveal what auth mechanisms are in use
- The existence of certain profiles (e.g., `status` with components) reveals architectural depth

### Guidance

- Public discovery SHOULD NOT enumerate authenticated or private resource URLs by default
- If public discovery signals that richer resources exist behind authentication, it SHOULD do so coarsely — indicating availability without exposing paths or internal structure
- The same exposure philosophy that applies to operational-state resources SHOULD apply to discovery documents and capabilities metadata

---

## 5. Recommended Pattern: Coarse Public / Rich Authenticated

The architecture is designed to support a layered exposure pattern:

1. **Public endpoint**: condition-only (Liveness or Readiness level detail)
2. **Authenticated endpoint**: full Health or Status with components, dependencies, timing, and evidence
3. **Discovery document**: public entry for the public endpoint; `"auth": "required"` entry for the richer endpoint

This pattern:

- Provides basic interoperability for public monitors (uptime services, status pages)
- Protects sensitive implementation detail behind authentication
- Uses discovery to connect the two without exposing internal structure

Implementations SHOULD consider this pattern as a safe default. It is already supported by the discovery and capabilities layers — this section makes the pattern explicit.

### Example: Discovery Document

```json
{
    "version": "1.0",
    "resources": [
        {
            "href": "/health",
            "profile": "health",
            "serialization": "application/health+json",
            "description": "Public health — condition only"
        },
        {
            "href": "/status",
            "profile": "status",
            "serialization": "application/status+json",
            "auth": "required",
            "description": "Detailed status — authenticated"
        }
    ]
}
```

### Example: Coarse Public Response

```json
{
    "condition": "operational"
}
```

This response is conformant, exposes no sensitive detail, and is sufficient for public monitoring and uptime checks.

---

## 6. Threat of Overexposure by Default

Implementations SHOULD treat any publicly exposed operational-state resource as potentially consumed by unintended parties — including automated scanners, competitors, and adversaries.

Specific guidance:

- **Examples and defaults** in frameworks and libraries implementing this standard SHOULD err toward coarse exposure
- **Richer detail** SHOULD require explicit opt-in rather than being the default for public endpoints
- **Documentation** for libraries and frameworks SHOULD call out the security implications of exposing component-level or full diagnostic detail publicly

---

## Normative Language Summary

This document uses RFC 2119 normative language as follows:

- **SHOULD** for security principles — implementations are expected to follow these unless they have specific reasons not to
- **MAY** for implementation choices — valid options that are neither recommended nor discouraged
- **MUST** is not used — exposure decisions are deployment-sensitive and this document provides guidance, not mandatory requirements

---

## Conformance

This document does not define new conformance requirements. It provides guidance that implementations SHOULD consider but are not tested against. The conformance levels defined in [CONFORMANCE.md](https://github.com/open-operational-state/status-conformance/blob/main/CONFORMANCE.md) remain the authoritative source for testable requirements.

The [status-conformance](https://github.com/open-operational-state/status-conformance) repository includes a coarse-public pattern fixture that demonstrates the recommended minimal public exposure pattern. This fixture validates structural correctness, not security policy.
