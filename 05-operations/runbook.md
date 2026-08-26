# Operations Runbook

Operational notes for maintaining this environment — the kind of documentation that would sit alongside a design in a real operations handover, so the network can be supported by someone other than its original architect.

## Adding a new VLAN at HQ

1. Create the VLAN on Core-SW1, Core-SW2, Distribution-SW, and Access-SW (no VTP is configured, so each switch must be updated individually)
2. Create matching SVIs on Core-SW1 and Core-SW2, following the existing `.2` / `.3` addressing convention
3. Configure HSRP on the new SVI on both core switches, using an HSRP group number matching the VLAN ID, with Core-SW1 at priority 110 and preempt enabled
4. Add the new subnet to the OSPF `network` statement on both core switches
5. Assign access ports to the new VLAN on Access-SW as needed
6. If the new VLAN requires isolation (like Guest), extend the `GUEST-ISOLATION`-style ACL pattern and apply it identically on both core switches

## Adding a new branch site

1. Assign the next available site range following the existing convention (e.g., `10.40.x.x` for a fourth site)
2. Allocate the next sequential `/30` block in `10.10.100.0/24` for the new WAN link
3. Assign the next unused OSPF area number to the new site
4. Configure the new branch router with its WAN-facing interface in Area 0 and its LAN-facing interface in its new area, following the same pattern as Malmo/Stockholm
5. Verify OSPF adjacency and inter-area route learning before considering the site live

## Common troubleshooting checks

| Symptom | Likely check |
|---|---|
| A site can't reach HQ or another site | `show ip ospf neighbor` on the routers either side of the affected link — confirm the adjacency is up, not just that the link is physically connected |
| Devices in a VLAN can't reach their gateway | `show standby brief` on both core switches — confirm one is Active for that VLAN's group, and the virtual IP matches what end devices are configured with |
| A routed link between a switch and router won't form an OSPF adjacency | Confirm the switch port is a routed Layer 3 interface (`no switchport`), not still a VLAN trunk — see [ADR-003](../02-architecture/architecture-decisions.md#adr-003-convert-the-core-sw1hq-router-link-from-a-layer-2-trunk-to-a-routed-layer-3-interface) |
| Guest devices can reach something they shouldn't | Confirm the `GUEST-ISOLATION` ACL is present and applied on **both** core switches, not just the one currently Active |

## Planned enhancements (not yet implemented)

See [ADR-006](../02-architecture/architecture-decisions.md#adr-006-represent-wan-links-as-direct-routed-connections-rather-than-a-full-site-to-site-vpn) and [Assumptions](../01-requirements/assumptions.md) for the full list, summarized here:

- Site-to-site IPsec VPN over the WAN links, replacing the current direct routed connections
- Dual-ISP redundancy per site
- A dedicated datacenter/cloud segment distinct from the general Servers VLAN
