# High-Level Design (HLD)

## Purpose

The HLD describes the overall architecture at a conceptual level — the major components, how they relate, and why the architecture is shaped this way. It does not include device-level configuration; that belongs in the [Low-Level Design](lld.md).

## Architecture summary

The network is organized into three sites connected by a routed WAN, with a hierarchical design at HQ scaled to its larger user population, and a simpler flat design at each branch scaled to their smaller footprint.

![Global Topology](diagrams/global-topology.png)

## Site design

### HQ (Copenhagen) — hierarchical 3-tier design

| Layer | Role |
|---|---|
| Core | Redundant Layer 3 switching pair, providing inter-VLAN routing and a resilient default gateway (via FHRP) for every VLAN |
| Distribution | Aggregates trunk links from the access layer up to the redundant core |
| Access | Connects end-user devices, enforcing VLAN membership at the edge |

This layered separation is used at HQ because its user count and VLAN count justify it, and because it allows the access and distribution layers to grow (more switches, more ports) without redesigning the core.

### Branch offices (Malmo, Stockholm) — flat design

Each branch uses a single router and switch pair. Given the smaller user population at each branch, a 3-tier design would add complexity without a corresponding benefit — this is a deliberate scope-matched design, not a simplification for its own sake.

## Routing architecture

Multi-area OSPF is used across the whole network:

- **Area 0 (backbone):** HQ's internal VLANs and all inter-site WAN links
- **Area 1:** Malmo's local network
- **Area 2:** Stockholm's local network

Each branch router functions as an Area Border Router (ABR), with one interface in Area 0 (the WAN link back to HQ) and one in its own local area.

## Redundancy architecture

HQ's core layer uses a First Hop Redundancy Protocol (HSRP) so that both core switches share a virtual gateway IP per VLAN. One switch is Active, the other Standby; if the Active switch fails, the Standby takes over automatically, and the original Active switch reclaims its role automatically once restored (preemption).

## Segmentation and security architecture

Five VLANs separate traffic by function at HQ (Users, Servers, Management, Guest, Voice). An extended ACL, applied consistently on both core switches, denies traffic originating from the Guest VLAN to every other internal VLAN, while permitting all other Guest traffic (e.g., outbound internet access).

## What is deliberately out of this design

See [Assumptions](../01-requirements/assumptions.md) and [Architecture Decision Records](architecture-decisions.md) for scoped-out elements (production VPN, dual-ISP redundancy, endpoint/identity security) and the reasoning behind each.
