# Technical Requirements

Derived from the business requirements in [`business-requirements.md`](business-requirements.md).

| ID | Technical Requirement | Traces to |
|---|---|---|
| TR-01 | HQ core switching must use a First Hop Redundancy Protocol (FHRP) to provide a redundant default gateway per VLAN | BR-01, BR-06 |
| TR-02 | Failover between redundant core devices must occur automatically, and the primary device must automatically reclaim its role once restored | BR-06 |
| TR-03 | Inter-site routing must be dynamic (not static), so new sites and topology changes are learned automatically | BR-02, BR-04 |
| TR-04 | Routing design must limit the propagation scope of topology changes, so a change at one site does not force route recalculation network-wide | BR-04 |
| TR-05 | Traffic types (users, servers, management, voice, guest) must be placed in separate broadcast domains (VLANs) | BR-05 |
| TR-06 | Guest VLAN traffic must be explicitly denied access to all other internal VLAN subnets at the routing layer | BR-03 |
| TR-07 | Guest isolation policy must be enforced consistently regardless of which redundant core device is currently active | BR-03, BR-01 |
| TR-08 | IP addressing must be structured and documented so that additional sites can be added following the same pattern | BR-04 |

## Constraints

- Environment built and tested in Cisco Packet Tracer (Cisco IOS syntax); design principles are vendor-general but the implementation is Cisco-specific
- All addressing uses RFC 1918 private space (10.0.0.0/8)
- WAN connectivity between sites is represented as direct routed links in the lab; see [Architecture Decision Records](../02-architecture/architecture-decisions.md) for the production equivalent (site-to-site VPN)
