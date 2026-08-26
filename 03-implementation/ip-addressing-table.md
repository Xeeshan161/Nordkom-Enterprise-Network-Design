# IP Addressing Table

## Addressing scheme logic

The design uses the private range `10.0.0.0/8`, broken down so the structure is self-explanatory:

- **`10.10.x.x`** — HQ (Copenhagen) internal networks and HQ-side infrastructure
- **`10.20.x.x`** — Malmo branch
- **`10.30.x.x`** — Stockholm branch
- **`10.10.100.0/24`** — reserved for all router-to-router point-to-point links, subnetted into sequential `/30` blocks

Using **`/30` subnets** for all point-to-point router links is a deliberate choice: a `/30` provides exactly 2 usable host addresses, which is the minimum needed for a link between exactly two routers. This avoids wasting IP space that a `/24` would waste on a two-device link, which is standard real-world WAN addressing practice.

## HQ — VLANs

| VLAN ID | Name | Subnet | HSRP Virtual IP (Gateway) | Core-SW1 SVI | Core-SW2 SVI |
|---|---|---|---|---|---|
| 10 | Users | 10.10.10.0/24 | 10.10.10.1 | 10.10.10.2 | 10.10.10.3 |
| 20 | Servers | 10.10.20.0/24 | 10.10.20.1 | 10.10.20.2 | 10.10.20.3 |
| 30 | Management | 10.10.30.0/24 | 10.10.30.1 | 10.10.30.2 | 10.10.30.3 |
| 40 | Guest | 10.10.40.0/24 | 10.10.40.1 | 10.10.40.2 | 10.10.40.3 |
| 50 | Voice | 10.10.50.0/24 | 10.10.50.1 | 10.10.50.2 | 10.10.50.3 |

HSRP group number matches the VLAN ID for each group (e.g., VLAN 10 uses HSRP group 10) for clarity and easy troubleshooting.

## Router-to-router links (all /30, all OSPF Area 0)

| Link | Subnet | Side A | Side B |
|---|---|---|---|
| Core-SW1 ↔ HQ-Router | 10.10.100.0/30 | Core-SW1: 10.10.100.2 | HQ-Router (Gi0/0): 10.10.100.1 |
| HQ-Router ↔ Malmo-Router | 10.10.100.4/30 | HQ-Router (Gi0/1): 10.10.100.5 | Malmo-Router (Gi0/0): 10.10.100.6 |
| HQ-Router ↔ Stockholm-Router | 10.10.100.8/30 | HQ-Router (Gi0/2): 10.10.100.9 | Stockholm-Router (Gi0/0): 10.10.100.10 |

**Note:** the Core-SW1 ↔ HQ-Router link was originally a Layer 2 trunk port with no IP — it had to be converted to a routed Layer 3 interface (`no switchport`) before OSPF could form an adjacency between HQ's internal network and HQ-Router. See [design-decisions.md](design-decisions.md) for the full explanation.

## Branch office LANs

| Site | OSPF Area | Subnet | Router LAN Interface | Hosts |
|---|---|---|---|---|
| Malmo | Area 1 | 10.20.1.0/24 | 10.20.1.1 | Malmo-PC0: 10.20.1.10, Malmo-PC1: 10.20.1.11 |
| Stockholm | Area 2 | 10.30.1.0/24 | 10.30.1.1 | Stockholm-PC0: 10.30.1.10, Stockholm-PC1: 10.30.1.11 |

## Management addressing

| Device | Management IP |
|---|---|
| Malmo-Switch | 10.20.1.2 |
| Stockholm-Switch | 10.30.1.2 |
