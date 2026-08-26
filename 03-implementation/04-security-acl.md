# Security — Guest VLAN Isolation ACL

Implements: [LLD — Security policy](../02-architecture/lld.md#security-policy-acl) · Visualized in [security-zones.png](../02-architecture/diagrams/security-zones.png)

## The ACL

Applied identically on **both** Core-SW1 and Core-SW2 (see [ADR-005](../02-architecture/architecture-decisions.md#adr-005-apply-the-guest-isolation-acl-identically-on-both-core-switches) for why both, not just the Active switch):

```
ip access-list extended GUEST-ISOLATION
deny ip 10.10.40.0 0.0.0.255 10.10.10.0 0.0.0.255
deny ip 10.10.40.0 0.0.0.255 10.10.20.0 0.0.0.255
deny ip 10.10.40.0 0.0.0.255 10.10.30.0 0.0.0.255
deny ip 10.10.40.0 0.0.0.255 10.10.50.0 0.0.0.255
permit ip any any
```

Applied inbound on the Guest VLAN's SVI:

```
interface vlan 40
ip access-group GUEST-ISOLATION in
```

## Why each line matters

| Line | Purpose |
|---|---|
| `deny ip 10.10.40.0 ... 10.10.10.0 ...` (and the following three) | Explicitly blocks Guest-to-internal-VLAN traffic, one line per protected VLAN |
| `permit ip any any` | Without this, an implicit deny-all at the end of every ACL would block all other Guest traffic too — this line explicitly allows everything else (e.g., outbound internet access) once the specific denials have been checked |
| `ip access-group ... in` | Applies the ACL to traffic **entering** the switch from the Guest VLAN — the correct direction to filter traffic originating from Guest devices |

## Verification

Tested directly, not just configured — a Guest device was confirmed unable to reach the Users and Servers VLANs, while retaining connectivity to its own gateway (proving the ACL is selective, not a blanket block). Full ping output: [test evidence](../04-testing/test-evidence.md#6-acl--guest-vlan-isolation).
