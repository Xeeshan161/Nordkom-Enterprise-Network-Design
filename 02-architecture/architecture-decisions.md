# Architecture Decision Records (ADRs)

Each entry records a significant design decision, the alternatives considered, and the reasoning — following the standard ADR format used in real architecture practice, so the *why* behind the design is preserved, not just the *what*.

---

## ADR-001: Use OSPF as the routing protocol

**Status:** Accepted

**Context:** The network needed a dynamic routing protocol connecting HQ to two branch offices, with room to add further sites in future.

**Options considered:**
- **EIGRP** — Cisco-proprietary (though partially opened since 2013), generally simpler to configure, composite metric can converge quickly in some topologies
- **OSPF** — open, RFC-defined standard, native hierarchical area structure

**Decision:** OSPF was selected.

**Rationale:** OSPF's vendor neutrality avoids locking the network into Cisco-only infrastructure — a real consideration for a growing company that may acquire or merge with organizations running different equipment. OSPF's native area hierarchy (backbone + numbered areas) also limits how far a topology change propagates, which matters for stability as more sites are added.

**Trade-off accepted:** EIGRP would have been simpler to configure and may converge marginally faster in some topologies. For a smaller, single-vendor, simplicity-first network, EIGRP would be a reasonable alternative — this was a genuine trade-off, not a one-sided choice.

---

## ADR-002: Use multi-area OSPF rather than a single flat area

**Status:** Accepted

**Context:** With three sites (and more anticipated), all routers could be placed in a single Area 0, or split into a formal hierarchy.

**Decision:** Each branch office was placed in its own OSPF area (Malmo = Area 1, Stockholm = Area 2), with HQ as the Area 0 backbone.

**Rationale:** This limits the scope of Shortest Path First (SPF) recalculation — a topology change within one branch's area does not force every router network-wide to recompute its routing table. This is standard practice once an OSPF network exceeds a handful of routers, and positions the design to scale cleanly as further sites are added (each simply becomes a new numbered area attached to the backbone).

**Trade-off accepted:** A single flat area is simpler to reason about and sufficient for very small networks. Given the explicit requirement that the design scale to more sites (BR-04), the added structure was judged worthwhile even at the current small scale.

---

## ADR-003: Convert the Core-SW1↔HQ-Router link from a Layer 2 trunk to a routed Layer 3 interface

**Status:** Accepted (implemented as a fix during build)

**Context:** During implementation, the physical link between Core-SW1 and HQ-Router was left configured as an 802.1Q trunk (carried over from earlier VLAN configuration on that port). This meant Core-SW1 had no IP-layer connectivity to HQ-Router, and OSPF could not form an adjacency between them, even though every device's OSPF configuration was individually correct.

**Diagnosis:** Comparing `show ip route` output between HQ-Router (which showed OSPF routes to both branches) and Core-SW1 (which showed only its own directly connected VLANs) revealed that Core-SW1 had no neighbor relationship with HQ-Router at all.

**Decision:** The port was converted from a switched trunk port to a routed interface (`no switchport`), assigned an IP in the `10.10.100.0/30` range, and added to Core-SW1's OSPF configuration.

**Rationale:** A point-to-point link between a Layer 3 switch and a router needs to be a routed interface, not a VLAN trunk — the two devices need a directly addressable IP relationship for OSPF to form an adjacency over that link.

**Outcome:** This is documented as an ADR rather than only a bug-fix note because it reflects a real, generalizable design principle: routed links between Layer 3 devices must be explicitly configured as Layer 3, not left as an artifact of earlier Layer 2 configuration.

---

## ADR-004: Use HSRP for HQ core gateway redundancy

**Status:** Accepted

**Context:** HQ's core layer needed a resilient default gateway per VLAN, so that a single core switch failure would not disconnect end users.

**Options considered:**
- **HSRP** (Cisco-proprietary First Hop Redundancy Protocol)
- **VRRP** (open-standard equivalent)

**Decision:** HSRP was used, with Core-SW1 set as the preferred Active router (priority 110) and preemption enabled.

**Rationale:** The environment is entirely Cisco-based, and HSRP is fully supported and straightforward to demonstrate and validate in this environment. Explicit priority and preemption were configured so failover and failback behavior is deterministic and testable, rather than left to default election behavior.

**Trade-off accepted:** VRRP would be the vendor-neutral choice, relevant if the network were multi-vendor. Given the current all-Cisco environment, this was not judged to be a limiting factor.

---

## ADR-005: Apply the Guest isolation ACL identically on both core switches

**Status:** Accepted

**Context:** The Guest VLAN's default gateway is HSRP-redundant — either Core-SW1 or Core-SW2 can be the Active router for that VLAN at any given time.

**Decision:** The `GUEST-ISOLATION` ACL was configured identically on both Core-SW1 and Core-SW2, not only on the switch that is normally Active.

**Rationale:** A security policy that exists on only one switch would silently disappear during an HSRP failover — Guest devices could gain unintended access to internal VLANs simply because the standby switch became active, with no configuration error triggering an alert. Applying the policy identically on both switches ensures the security posture holds regardless of which switch is currently handling traffic.

---

## ADR-006: Represent WAN links as direct routed connections rather than a full site-to-site VPN

**Status:** Accepted (scoped for this build; noted for future work)

**Context:** In production, HQ-to-branch connectivity would typically run over the internet via an encrypted site-to-site VPN (IPsec), not a private direct link.

**Decision:** For this build, WAN links are modeled as direct routed Ethernet connections between routers, with routing (OSPF) and design principles kept identical to what a production IPsec-secured link would carry.

**Rationale:** The focus of this design exercise is the routing, redundancy, and segmentation architecture. Adding full IPsec VPN configuration would demonstrate a different, additional skill set (VPN/security configuration) without changing the core routing/redundancy/segmentation design being showcased. This is documented explicitly as a scoping decision, not an oversight.

**Future work:** Implementing IPsec site-to-site VPN over the existing WAN links, with OSPF running over the VPN tunnel interfaces, would be a natural extension of this design.
