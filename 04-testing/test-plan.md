# Test Plan

Defines what was tested and why, before presenting the results. A design isn't proven until each requirement is deliberately verified — this maps each test back to the requirement it validates.

| Test | Validates requirement | Method |
|---|---|---|
| Inter-VLAN connectivity at HQ | TR-05 (VLAN segmentation doesn't break legitimate routing) | Ping between devices in different VLANs |
| HSRP failover | TR-01, TR-02 (redundant gateway, automatic failover) | Administratively shut down the Active switch's SVIs, observe traffic during and after |
| HSRP failback (preemption) | TR-02 (primary automatically reclaims role) | Restore the failed switch, confirm it reclaims Active status without manual intervention |
| Multi-area OSPF routing | TR-03, TR-04 (dynamic, hierarchical routing) | Inspect route tables for correct inter-area (`O IA`) route learning on multiple devices |
| End-to-end site connectivity | BR-02 (all sites can communicate) | Ping from HQ to each branch office |
| Guest VLAN isolation | TR-06, TR-07 (Guest blocked from internal VLANs, consistently) | Ping from Guest VLAN to multiple internal VLANs (expect failure) and to its own gateway (expect success) |

Full results for every test above: [`test-evidence.md`](test-evidence.md)
