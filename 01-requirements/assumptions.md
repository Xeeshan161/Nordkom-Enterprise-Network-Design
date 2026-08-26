# Assumptions

Documented explicitly so the design's boundaries are clear — an architecture document should state what it assumes, not just what it decides.

1. **Company and requirements are fictional.** Nordkom Manufacturing does not exist; the scenario was constructed to be realistic enough to justify genuine architectural decisions, not to reflect a real organization.

2. **Lab environment substitutes for physical infrastructure.** Cisco Packet Tracer is used to build and test the design. Device models (2911 routers, 3560/2960 switches) were chosen for their availability in Packet Tracer and their reasonable real-world equivalents, not as a specific product recommendation.

3. **WAN links are simplified.** In the lab, HQ-to-branch connectivity is modeled as direct routed Ethernet links. In production, these would be site-to-site VPN tunnels over internet-provided WAN circuits — see [Architecture Decision Records](../02-architecture/architecture-decisions.md) for how this would be implemented and why it was scoped out of the lab build.

4. **Single ISP per site assumed.** Dual-ISP/dual-WAN redundancy was not implemented; each site currently has a single upstream path. Noted as a future enhancement.

5. **User and device counts are illustrative, not load-tested.** VLAN and IP scheme sizing (e.g., /24 subnets) comfortably exceeds the stated ~150-user HQ population and would not need to change at that scale; no capacity/load testing was performed since this is a design and connectivity proof-of-concept, not a performance benchmark.

6. **Security scope is limited to network-layer segmentation.** Endpoint security, identity/access management, and application-layer controls are out of scope for this design; only network segmentation (VLANs + ACLs) is addressed.
