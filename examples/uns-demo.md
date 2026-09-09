# Unified namespace demo — graph representation

Companion diagram for [uns-demo.json](uns-demo.json). One enrollment group of a car manufacturer,
nested by geolocation from a continent down to an assembly line — the maximum legal agent group
level of 5, with its agents one hop below at level 6.

Levels are counted **from the enrollment group, which is level 0**: agent groups occupy levels 1..5
and agents sit at level 6 at the deepest.

## Graph

```mermaid
graph TD
  N["🖧 tributech-node-eu<br/><i>node;1</i> · level -1"]
  EG["🏢 nordstern-motors<br/><i>enrollmentgroup;1</i> · level 0"]

  G1["🌍 europe<br/>level 1"]
  G2["🏙 vienna<br/>level 2"]
  G3["🏭 plant-north<br/>level 3"]
  G4["🔧 body-shop<br/>level 4"]
  G5A["🚗 assembly-line-1<br/>level 5"]
  G5B["🚗 assembly-line-2<br/>level 5"]

  U["📥 unassigned<br/><i>unassignedagentgroup;1</i> · level 1"]

  A1(["site-gateway-01"])
  A2(["energy-meter-main"])
  A3(["press-line-2"])
  A4(["weld-robot-07"])
  A5(["torque-station-03"])
  A6(["agent-af31c9"])

  N -->|EnrollmentGroups| EG
  EG -->|AgentGroups| G1
  EG -->|UnassignedAgentGroup| U

  G1 -->|ChildAgentGroups| G2
  G2 -->|ChildAgentGroups| G3
  G3 -->|ChildAgentGroups| G4
  G4 -->|ChildAgentGroups| G5A
  G4 -->|ChildAgentGroups| G5B

  G2 -->|Agents| A1
  G3 -->|Agents| A2
  G4 -->|Agents| A3
  G5A -->|Agents| A4
  G5A -->|Agents| A5
  U -->|Agents| A6

  classDef agent fill:#e8f0fe,stroke:#4d7cc7,color:#1a2b45
  classDef reserved fill:#fdf0e3,stroke:#c78b3d,color:#452f1a,stroke-dasharray:4 3
  class A1,A2,A3,A4,A5,A6 agent
  class U reserved
```

## Namespace paths

The `Name` of each agent group is a path segment; the path is the chain of `ChildAgentGroups` names
below the enrollment group.

| Agent | Namespace path |
|---|---|
| `weld-robot-07`, `torque-station-03` | nordstern-motors › europe › vienna › plant-north › body-shop › assembly-line-1 |
| `press-line-2` | nordstern-motors › europe › vienna › plant-north › body-shop |
| `energy-meter-main` | nordstern-motors › europe › vienna › plant-north |
| `site-gateway-01` | nordstern-motors › europe › vienna |
| `agent-af31c9` | nordstern-motors › unassigned |

## What the demo shows

- **Level 5 is the deepest legal agent group.** `assembly-line-1` and `assembly-line-2` sit there;
  creating a child below either is rejected.
- **Agents consume no level of their own.** `weld-robot-07` hangs off a level-5 group and lands at
  level 6 — the deepest anything gets, and the maximum of the `depth` parameter.
- **Agents may sit on any level, not only on leaves.** Three agents are filed on the intermediate
  groups `vienna`, `plant-north` and `body-shop` — a site gateway, a plant energy meter and a shop
  press line each belong to their level rather than to a line.
- **An agent group may be empty.** `assembly-line-2` exists with no agents.
- **Uncategorized agents are a real node, not a computed set.** `agent-af31c9` was activated without
  an agent group, so it holds an ordinary `Agents` edge from the reserved `unassigned` group. The
  enrollment group itself never holds `Agents` edges.
- **Labels are localized, paths are not.** Every group carries `en`/`de` `DisplayNames`; the `Name`
  that forms the path stays untranslated, so `vienna` resolves identically in either locale.
- **The node twin is level -1.** It is reachable only with `startOffset=-1`, and then carries only
  the one `EnrollmentGroups` edge that leads to the caller's own enrollment group.

## Size

| Element | Count |
|---|---|
| Node twins | 1 |
| Enrollment groups | 1 |
| Agent groups (incl. the reserved `unassigned`) | 7 |
| Agents | 6 |
| **Twins total** | **15** |
| Relationships | 14 |
| **Graph elements total** | **29** |
