# Multi-Area OSPF Routing

Implements: [LLD — OSPF parameters](../02-architecture/lld.md#ospf-parameters)

## HQ-Router

Advertises its three WAN-facing `/30` links into Area 0:

```
router ospf 1
network 10.10.100.0 0.0.0.3 area 0
network 10.10.100.4 0.0.0.3 area 0
network 10.10.100.8 0.0.0.3 area 0
```

## Core-SW1 and Core-SW2

Advertise all five HQ VLAN subnets into Area 0. Core-SW1 additionally advertises the routed link to HQ-Router:

```
router ospf 1
network 10.10.10.0 0.0.0.255 area 0
network 10.10.20.0 0.0.0.255 area 0
network 10.10.30.0 0.0.0.255 area 0
network 10.10.40.0 0.0.0.255 area 0
network 10.10.50.0 0.0.0.255 area 0
network 10.10.100.0 0.0.0.3 area 0
```

**Important:** the link between Core-SW1 and HQ-Router must be a routed Layer 3 interface, not a VLAN trunk, for this OSPF adjacency to form. See [ADR-003](../02-architecture/architecture-decisions.md#adr-003-convert-the-core-sw1hq-router-link-from-a-layer-2-trunk-to-a-routed-layer-3-interface) for the full story of this issue and its fix.

## Malmo-Router (Area Border Router)

```
router ospf 1
network 10.10.100.4 0.0.0.3 area 0
network 10.20.1.0 0.0.0.255 area 1
```

## Stockholm-Router (Area Border Router)

```
router ospf 1
network 10.10.100.8 0.0.0.3 area 0
network 10.30.1.0 0.0.0.255 area 2
```

## Verification

```
show ip ospf neighbor
show ip route
```

Routes learned from another area appear with the `O IA` (inter-area) designation — confirming the area hierarchy is functioning as designed. Full output captured in [test evidence](../04-testing/test-evidence.md#4-multi-area-ospf-routing).
