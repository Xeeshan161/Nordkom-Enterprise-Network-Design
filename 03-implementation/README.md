# Implementation

This folder contains the actual device configuration that implements the [Low-Level Design](../02-architecture/lld.md), organized by architectural layer rather than by individual device — so the configuration reads in the same order as the design.

| File | Covers |
|---|---|
| [`ip-addressing-table.md`](ip-addressing-table.md) | Full IP addressing reference for every device and link |
| [`01-vlans-and-trunking.md`](01-vlans-and-trunking.md) | VLAN creation and trunk configuration across all HQ switches |
| [`02-inter-vlan-routing-and-hsrp.md`](02-inter-vlan-routing-and-hsrp.md) | SVI and HSRP configuration on the HQ core |
| [`03-ospf-routing.md`](03-ospf-routing.md) | Multi-area OSPF configuration across all sites |
| [`04-security-acl.md`](04-security-acl.md) | Guest VLAN isolation ACL |

Each file includes the configuration itself, plus a short explanation of what each block does — not just a raw config dump.
