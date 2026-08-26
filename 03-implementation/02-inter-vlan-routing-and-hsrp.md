# Inter-VLAN Routing and HSRP

Implements: [LLD — HSRP parameters](../02-architecture/lld.md#hsrp-parameters)

## Enabling Layer 3 routing

Both core switches require routing enabled explicitly, since they are multilayer switches, not dedicated routers:

```
ip routing
```

## SVIs (Switch Virtual Interfaces)

Each VLAN gets a routable Layer 3 interface on each core switch. Example for VLAN 10:

**Core-SW1:**
```
interface vlan 10
ip address 10.10.10.2 255.255.255.0
no shutdown
```

**Core-SW2:**
```
interface vlan 10
ip address 10.10.10.3 255.255.255.0
no shutdown
```

This pattern is repeated for VLANs 20, 30, 40, and 50 on both switches (Core-SW1 uses `.2`, Core-SW2 uses `.3` in each subnet).

## HSRP configuration

Each VLAN gets an HSRP group providing a shared virtual gateway IP.

**Core-SW1 (Active):**
```
interface vlan 10
standby 10 ip 10.10.10.1
standby 10 priority 110
standby 10 preempt
```

**Core-SW2 (Standby):**
```
interface vlan 10
standby 10 ip 10.10.10.1
```

| Command | Purpose |
|---|---|
| `standby 10 ip 10.10.10.1` | Creates HSRP group 10, virtual IP `.1` — this is the address end devices use as their gateway |
| `standby 10 priority 110` | Makes Core-SW1 win the election over Core-SW2's default priority of 100 |
| `standby 10 preempt` | Core-SW1 automatically reclaims Active status once restored after a failure |

Repeated identically for VLANs 20, 30, 40, and 50, using HSRP groups 20, 30, 40, and 50 respectively (group number matches VLAN ID by convention, for clarity).

Verified with `show standby brief` — see [test evidence](../04-testing/test-evidence.md#2-hsrp-failover-test) for full failover/failback proof.
