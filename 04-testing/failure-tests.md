# Failure Testing

Structured failure scenarios validated against this design, following a consistent format so each test's objective, expected behavior, and actual outcome are clear at a glance. This is deliberate architecture validation — proving the design survives failure, not just that it works under normal conditions.

---

## TEST-001 — HQ Core Switch Failure (HSRP Failover)

| Field | Detail |
|---|---|
| **Objective** | Confirm that a failure of the Active HSRP core switch does not cause a sustained outage for HQ VLANs |
| **Expected behavior** | The Standby core switch (Core-SW2) detects the failure and takes over as Active within the HSRP hold-down window; end devices experience at most a few seconds of interruption, with no reconfiguration required on any end device |
| **Failure introduced** | VLAN 10, 20, and 40 SVIs administratively shut down on Core-SW1 (the Active router for all VLANs at the time), simulating a core switch/interface failure |
| **Observed behavior** | Ping from an HQ PC across the affected gateway showed 2 timeouts, then successful replies. `show standby brief` on Core-SW2 confirmed it transitioned to Active for VLANs 10, 20, and 40 — the exact VLANs affected — while VLANs 30 and 50 (untouched) remained Active on Core-SW1 throughout |
| **Recovery time** | Sub-second to a few seconds — consistent with 2 lost ping packets before the connection re-established (default HSRP timers) |
| **Result** | **Pass.** Failover occurred automatically, with no manual intervention and no end-device reconfiguration required |

**Evidence:**
```
C:\>ping 10.10.20.10
Request timed out.
Request timed out.
Reply from 10.10.20.10: bytes=32 time=1ms TTL=127
Reply from 10.10.20.10: bytes=32 time<1ms TTL=127
Packets: Sent = 4, Received = 2, Lost = 2 (50% loss)

Core-SW2#show standby brief
Vl10  10  100  Active  local  unknown  10.10.10.1
Vl20  20  100  Active  local  unknown  10.10.20.1
Vl40  40  100  Active  local  unknown  10.10.40.1
```

---

## TEST-002 — HQ Core Switch Recovery (HSRP Failback / Preemption)

| Field | Detail |
|---|---|
| **Objective** | Confirm that the primary core switch automatically reclaims its Active role once restored, rather than leaving the network permanently dependent on the switch that took over during the failure |
| **Expected behavior** | Core-SW1, configured with `preempt` and a higher priority (110) than Core-SW2 (100), should reclaim Active status for all affected VLANs immediately upon restoration, without manual intervention |
| **Failure introduced** | N/A — this test follows directly from TEST-001; the previously shut-down interfaces on Core-SW1 were restored (`no shutdown`) |
| **Observed behavior** | `show standby brief` on Core-SW1 showed **Active** status restored on all 5 VLANs, with Core-SW2 correctly shown as the standby neighbor on each |
| **Recovery time** | Immediate upon interface restoration — no delay observed |
| **Result** | **Pass.** Preemption behaved exactly as configured — full automatic self-healing across the failure/recovery cycle, not just one-way failover |

**Evidence:**
```
Core-SW1#show standby brief
Vl10  10  110 P Active  local  10.10.10.3  10.10.10.1
Vl20  20  110 P Active  local  10.10.20.3  10.10.20.1
Vl30  30  110 P Active  local  10.10.30.3  10.10.30.1
Vl40  40  110 P Active  local  10.10.40.3  10.10.40.1
Vl50  50  110 P Active  local  10.10.50.3  10.10.50.1
```

---

## TEST-003 — Guest VLAN Attempting Access to Internal Resources

| Field | Detail |
|---|---|
| **Objective** | Confirm the Guest VLAN cannot reach internal corporate VLANs, regardless of which core switch is currently Active |
| **Expected behavior** | Traffic from the Guest subnet (10.10.40.0/24) to any internal VLAN subnet should be denied by the `GUEST-ISOLATION` ACL; traffic to the Guest VLAN's own gateway, and by extension outbound traffic, should remain unaffected |
| **Failure introduced** | N/A — this is a security policy validation test, not a failure scenario; included here because it follows the same "prove it, don't just configure it" principle |
| **Observed behavior** | Ping from the Guest PC to the Users VLAN (10.10.10.10) and Servers VLAN (10.10.20.10) both failed with "Destination host unreachable" from the ACL. A control ping to the Guest VLAN's own gateway (10.10.40.1) succeeded normally, confirming the ACL is selectively blocking only the intended destinations |
| **Recovery time** | N/A |
| **Result** | **Pass.** Isolation confirmed on both tested internal VLANs; normal Guest connectivity unaffected |

**Evidence:**
```
C:\>ping 10.10.10.10
Request timed out.
Reply from 10.10.40.2: Destination host unreachable.  (x3)

C:\>ping 10.10.20.10
Reply from 10.10.40.2: Destination host unreachable.  (x4)

C:\>ping 10.10.40.1
Reply from 10.10.40.1: bytes=32 time=10ms TTL=255  (x4)
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

---

## TEST-004 — Loss of OSPF Adjacency Between Core and HQ-Router *(diagnosed during build, not a planned test)*

| Field | Detail |
|---|---|
| **Objective** | Not originally planned as a test — this documents a real connectivity gap discovered and resolved during implementation, included here because it demonstrates the same systematic validation approach as the planned tests above |
| **Expected behavior** | Core-SW1 should have a full routing path to both branch offices via OSPF, learned through HQ-Router |
| **Failure introduced** | N/A — pre-existing misconfiguration: the link between Core-SW1 and HQ-Router was left as a Layer 2 trunk port rather than a routed Layer 3 interface, so no IP-layer adjacency could form |
| **Observed behavior** | `show ip route` on Core-SW1 showed only its own directly connected VLANs — no OSPF neighbor to HQ-Router, and no routes to either branch, despite correct OSPF configuration on every individual device |
| **Recovery time** | N/A — required a configuration fix (`no switchport` + IP addressing + OSPF network statement), not an automatic recovery mechanism |
| **Result** | **Fixed and re-verified.** Full inter-area (`O IA`) routes to both branches appeared on Core-SW1 immediately after the fix — see [ADR-003](../02-architecture/architecture-decisions.md#adr-003-convert-the-core-sw1hq-router-link-from-a-layer-2-trunk-to-a-routed-layer-3-interface) for the full design reasoning |

---

## Tests planned for future architecture iterations

The following failure scenarios are relevant to a fuller architecture (WAN edge, datacenter fabric, cloud connectivity) and are noted here as the design expands — not yet applicable to the current build, which does not yet include these components:

| Test ID | Scenario | Applicable when |
|---|---|---|
| TEST-005 | Primary WAN/internet link failure at a branch | Once dual-ISP/SD-WAN redundancy is implemented |
| TEST-006 | BGP neighbor failure at the WAN edge | Once BGP is introduced at the WAN edge |
| TEST-007 | Datacenter leaf switch failure | Once a VXLAN/EVPN datacenter fabric is added |
| TEST-008 | Site-to-site VPN tunnel failure | Once IPsec VPN replaces the current direct WAN links (see ADR-006) |
