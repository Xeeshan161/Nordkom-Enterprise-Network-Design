# Test Evidence

This document captures the actual command output and test results gathered while building this network, as proof that each design element genuinely works — not just that it was configured. Each section maps to a test defined in [`test-plan.md`](test-plan.md).

## 1. Inter-VLAN routing at HQ

Ping from PC0 (VLAN 10, Users) to PC2 (VLAN 20, Servers) and PC3 (VLAN 40, Guest, prior to ACL being applied):

```
C:\>ping 10.10.20.10
Reply from 10.10.20.10: bytes=32 time<1ms TTL=127
Reply from 10.10.20.10: bytes=32 time<1ms TTL=127
Reply from 10.10.20.10: bytes=32 time<1ms TTL=127
Reply from 10.10.20.10: bytes=32 time=16ms TTL=127
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)

C:\>ping 10.10.40.10
Reply from 10.10.40.10: bytes=32 time<1ms TTL=127
Reply from 10.10.40.10: bytes=32 time=1ms TTL=127
Reply from 10.10.40.10: bytes=32 time<1ms TTL=127
Reply from 10.10.40.10: bytes=32 time<1ms TTL=127
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

**Result:** confirms SVIs, trunking, and inter-VLAN routing via the core switches are all functioning correctly.

## 2. HSRP failover test

**Setup:** Core-SW1 was Active (priority 110) for all 5 VLANs, Core-SW2 Standby (priority 100).

**Action:** VLAN 10, 20, and 40 SVIs were administratively shut down on Core-SW1 to simulate a failure.

**Ping during failure (PC0 → PC2, crossing the gateway):**
```
C:\>ping 10.10.20.10
Request timed out.
Request timed out.
Reply from 10.10.20.10: bytes=32 time=1ms TTL=127
Reply from 10.10.20.10: bytes=32 time<1ms TTL=127
Packets: Sent = 4, Received = 2, Lost = 2 (50% loss)
```

**Standby status on Core-SW2 during the failure:**
```
Core-SW2#show standby brief
Interface Grp Pri P State   Active  Standby  Virtual IP
Vl10      10  100   Active  local   unknown  10.10.10.1
Vl20      20  100   Active  local   unknown  10.10.20.1
Vl30      30  100   Standby 10.10.30.2 local 10.10.30.1
Vl40      40  100   Active  local   unknown  10.10.40.1
Vl50      50  100   Standby 10.10.50.2 local 10.10.50.1
```

**Result:** Core-SW2 correctly detected Core-SW1's failure and took over as Active for the affected VLANs (10, 20, 40). Only VLANs that were actually shut down on Core-SW1 failed over — VLANs 30 and 50 remained on Core-SW1 throughout, exactly as expected since they weren't part of the simulated failure. The two lost pings represent the brief, expected HSRP hold-down detection window before takeover.

## 3. HSRP failback (preemption) test

**Action:** Core-SW1's VLAN 10, 20, and 40 interfaces were restored (`no shutdown`).

**Standby status on Core-SW1 after restoration:**
```
Core-SW1#show standby brief
Interface Grp Pri P State   Active  Standby     Virtual IP
Vl10      10  110 P Active  local   10.10.10.3  10.10.10.1
Vl20      20  110 P Active  local   10.10.20.3  10.10.20.1
Vl30      30  110 P Active  local   10.10.30.3  10.10.30.1
Vl40      40  110 P Active  local   10.10.40.3  10.10.40.1
Vl50      50  110 P Active  local   10.10.50.3  10.10.50.1
```

**Result:** Core-SW1 automatically reclaimed Active status on all 5 VLANs the moment it came back online, confirming `preempt` worked as designed — full, automatic self-healing behavior, not just one-way failover.

## 4. Multi-area OSPF routing

**HQ-Router route table**, showing inter-area (IA) routes learned from both branches:
```
HQ-Router#show ip route
O IA  10.20.1.0/24 [110/2] via 10.10.100.6, GigabitEthernet0/1
O IA  10.30.1.0/24 [110/2] via 10.10.100.10, GigabitEthernet0/2
```

**Core-SW1 route table**, after fixing the routed-link issue (see design-decisions.md), showing it too correctly learned both branch networks as inter-area routes:
```
Core-SW1#show ip route
O IA  10.20.1.0/24 [110/3] via 10.10.100.1, FastEthernet0/1
O IA  10.30.1.0/24 [110/3] via 10.10.100.1, FastEthernet0/1
```

**Result:** the "IA" designation confirms OSPF correctly recognizes these as inter-area routes, proving the Area 0 / Area 1 / Area 2 hierarchy is functioning as designed, not just flooding routes as if it were a single flat area.

## 5. End-to-end connectivity, HQ to both branches

```
C:\>ping 10.20.1.10          (HQ PC0 -> Malmo-PC)
Reply from 10.20.1.10: bytes=32 time<1ms TTL=125   x4
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)

C:\>ping 10.30.1.10          (HQ PC0 -> Stockholm-PC)
Request timed out.
Reply from 10.30.1.10: bytes=32 time=27ms TTL=125
Reply from 10.30.1.10: bytes=32 time=60ms TTL=125
Reply from 10.30.1.10: bytes=32 time<1ms TTL=125
Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)
```

**Result:** full connectivity confirmed from HQ to both branch offices. The single timeout on the first Stockholm ping is expected ARP-resolution delay on the first packet to a new destination, not a routing fault — subsequent packets succeeded normally.

## 6. ACL — Guest VLAN isolation

**Test: Guest PC (PC3, VLAN 40) attempting to reach the Users VLAN (VLAN 10):**
```
C:\>ping 10.10.10.10
Request timed out.
Reply from 10.10.40.2: Destination host unreachable.  x3
Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

**Test: Guest PC attempting to reach the Servers VLAN (VLAN 20):**
```
C:\>ping 10.10.20.10
Reply from 10.10.40.2: Destination host unreachable.  x4
Packets: Sent = 4, Received = 0, Lost = 4 (100% loss)
```

**Control test: Guest PC reaching its own gateway (should still succeed):**
```
C:\>ping 10.10.40.1
Reply from 10.10.40.1: bytes=32 time=10ms TTL=255
Reply from 10.10.40.1: bytes=32 time=20ms TTL=255
Reply from 10.10.40.1: bytes=32 time=20ms TTL=255
Reply from 10.10.40.1: bytes=32 time=1ms TTL=255
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

**Result:** Guest VLAN traffic is consistently and correctly blocked from every internal VLAN it was designed to be isolated from, while normal Guest connectivity to its own gateway is unaffected — confirming the ACL is selectively enforcing policy, not broadly breaking connectivity.
