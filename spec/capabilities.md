# Capabilities / Negotiation

> **Status:** Outline — not yet normative.

## Purpose

Capabilities describe what a target supports. Negotiation covers how a monitor and target determine the best interaction mode.

## Why This Layer Exists

As the ecosystem supports multiple profiles, serializations, and authentication modes, monitors need a way to understand what is available before attempting to consume a resource. Without this layer, extensibility becomes opaque or fragile.

## What Capabilities Describe

- Which **profile(s)** the target supports
- Which **serialization(s)** are available
- What **authentication modes** apply
- What **evidence depth** is available (e.g., self-reported only vs. component-level detail)
- What **condition vocabularies** the target uses

## Negotiation Mechanisms

Negotiation may happen through:

- **Capabilities documents** — dedicated endpoints describing what is available
- **HTTP headers** — standard or extension headers advertising capabilities
- **Discovery metadata** — capabilities information embedded in discovery responses
- **Content negotiation** — standard HTTP content negotiation where supported

## Design Constraints

- Capabilities should be **discoverable** without trial and error
- Monitors should be able to determine available interaction modes **before** consuming resources
- Capabilities should not require complex multi-step negotiation for basic use cases

## Open Questions

- Exact capabilities document format
- Whether capabilities are a separate resource or embedded in discovery
- How capabilities interact with caching and freshness
- Version negotiation strategy

## References

- [ARCHITECTURE.md](../ARCHITECTURE.md) — Layer 6 definition
- [discovery.md](discovery.md) — the layer that locates resources (capabilities describes them)
