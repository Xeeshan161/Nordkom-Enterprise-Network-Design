# VLANs and Trunking

Implements: [LLD — VLAN design and trunking](../02-architecture/lld.md#vlan-design-hq)

## VLAN creation

Created identically on Distribution-SW, Access-SW, Core-SW1, and Core-SW2 (no VTP configured, so each switch needs to know about every VLAN independently):

```
vlan 10
name Users
exit
vlan 20
name Servers
exit
vlan 30
name Management
exit
vlan 40
name Guest
exit
vlan 50
name Voice
exit
```

## Trunk configuration

Applied to every inter-switch link (Core-SW1↔Core-SW2, Core-SW1↔Distribution-SW, Core-SW2↔Distribution-SW, Distribution-SW↔Access-SW):

```
interface GigabitEthernet0/1
switchport trunk encapsulation dot1q
switchport mode trunk
```

**Note:** the 2960 (Access-SW) only supports 802.1Q trunking, so it does not accept the `switchport trunk encapsulation` command at all — `switchport mode trunk` alone is sufficient on that model.

## Access port configuration

Each end-device-facing port on Access-SW is assigned to exactly one VLAN:

```
interface FastEthernet0/1
switchport mode access
switchport access vlan 10
```

| Port | Device | VLAN |
|---|---|---|
| Fa0/1 | PC0 | 10 (Users) |
| Fa0/2 | PC1 | 10 (Users) |
| Fa0/3 | PC2 | 20 (Servers) |
| Fa0/4 | PC3 | 40 (Guest) |
