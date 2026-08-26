# Business Requirements

## Organization

Nordkom Manufacturing is a fictional mid-size manufacturing company, used here as a realistic design brief. Headquartered in Copenhagen, with satellite offices in Malmo and Stockholm.

## Business context

- HQ hosts production-critical operations and the majority of staff (~150 users). Downtime at HQ's core network directly impacts production-related systems.
- Malmo and Stockholm are smaller satellite offices supporting local staff, with less stringent availability needs than HQ.
- The company anticipates adding further sites as it grows, so the design should not require rework to accommodate that.
- Guest and visitor network access is required at HQ, but must not expose internal systems.

## Business requirements

| ID | Requirement | Priority |
|---|---|---|
| BR-01 | HQ core network must have no single point of failure — a single switch failure must not cause an outage | Must have |
| BR-02 | All three sites must be able to communicate with each other over the corporate network | Must have |
| BR-03 | Guest/visitor devices must not be able to reach internal corporate resources | Must have |
| BR-04 | The design must scale to additional sites without requiring a redesign of the existing architecture | Should have |
| BR-05 | Different departments/traffic types (users, servers, management, voice, guest) must be logically separated | Must have |
| BR-06 | Recovery from a core device failure should be automatic, without manual intervention | Should have |

## Out of scope

- Physical site security, cabling, and power redundancy
- End-user application and server design (only the network transport layer is addressed)
- Formal disaster recovery / business continuity planning beyond core network redundancy
- Budget and procurement — this is a design and technical proof-of-concept exercise, not a costed proposal
