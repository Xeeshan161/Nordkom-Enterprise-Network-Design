# Low-Level Design (LLD)

## Purpose

The LLD provides the concrete, device-level detail that implements the [High-Level Design](hld.md) — specific IP addressing, VLAN IDs, protocol parameters, and interface assignments. This is the level of detail an engineer would follow to actually build the network.

## Addressing scheme

Private addressing (`10.0.0.0/8`) is structured so the scheme is self-documenting:

- `10.10.x.x` — HQ (Copenhagen)
- `10.20.x.x` — Malmo branch
- `10.30.x.x` — Stockholm branch
- `10.10.100.0/24` — reserved for all router-to-router point-to-point links, subnetted into sequential `/30` blocks (2 usable hosts per link, no wasted address space)

Full addressing detail: see [`03-implementation/ip-addressing-table.md`](../03-implementation/ip-addressing-table.md)

## VLAN design (HQ)

| VLAN ID | Name | Subnet | Gateway (HSRP VIP) |
|---|---|---|---|
| 10 | Users | 10.10.10.0/24 | 10.10.10.1 |
| 20 | Servers | 10.10.20.0/24 | 10.10.20.1 |
| 30 | Management | 10.10.30.0/24 | 10.10.30.1 |
| 40 | Guest | 10.10.40.0/24 | 10.10.40.1 |
| 50 | Voice | 10.10.50.0/24 | 10.10.50.1 |

## HSRP parameters

| Parameter | Core-SW1 | Core-SW2 |
|---|---|---|
| Role | Primary (Active) | Secondary (Standby) |
| Priority | 110 | 100 (default) |
| Preempt | Enabled | Not required |
| HSRP group number | Matches VLAN ID (e.g., group 10 for VLAN 10) | Same |

## OSPF parameters

| Parameter | Value |
|---|---|
| Process ID | 1 (locally significant, consistent across all devices for clarity) |
| Area 0 | HQ VLANs (10.10.10.0/24 – 10.10.50.0/24) and all `/30` WAN links |
| Area 1 | Malmo LAN (10.20.1.0/24) |
| Area 2 | Stockholm LAN (10.30.1.0/24) |
| ABR devices | Malmo-Router, Stockholm-Router |

## Interconnect details

| Link | Subnet | Notes |
|---|---|---|
| Core-SW1 ↔ HQ-Router | 10.10.100.0/30 | Routed Layer 3 interface (not a VLAN trunk) — see [ADR-003](architecture-decisions.md) |
| HQ-Router ↔ Malmo-Router | 10.10.100.4/30 | OSPF Area 0 |
| HQ-Router ↔ Stockholm-Router | 10.10.100.8/30 | OSPF Area 0 |

## Trunking

All HQ inter-switch links (Core↔Core, Core↔Distribution, Distribution↔Access) are configured as 802.1Q trunks carrying all 5 VLANs. Access-layer ports are configured individually as access ports, one VLAN per port.

## Security policy (ACL)

A single named extended ACL (`GUEST-ISOLATION`) is applied inbound on the VLAN 40 (Guest) SVI on both core switches:

- Denies traffic from `10.10.40.0/24` to each of the other four internal VLAN subnets
- Implicit final rule: permit all other traffic (explicit `permit ip any any` to override the implicit deny-all)

Full configuration detail: see [`03-implementation/`](../03-implementation)
