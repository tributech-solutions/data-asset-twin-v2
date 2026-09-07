# Unified namespace demo — graph representation

Companion diagram for [uns-demo.json](uns-demo.json). One enrollment group of a car manufacturer,
nested by geolocation from a continent down to an assembly line — the maximum agent group depth of 6.

## Graph

```mermaid
graph TD
  N["🖧 tributech-node-eu<br/><i>node;1</i>"]
  EG["🏢 nordstern-motors<br/><i>enrollmentgroup;1</i> · depth 0"]

  G1["🌍 europe<br/>depth 1"]
  G2["🇦🇹 austria<br/>depth 2"]
  G3["🏙 vienna<br/>depth 3"]
  G4["🏭 plant-north<br/>depth 4"]
  G5["🔧 body-shop<br/>depth 5"]
  G6A["🚗 assembly-line-1<br/>depth 6"]
  G6B["🚗 assembly-line-2<br/>depth 6"]

  U["📥 unassigned<br/><i>unassignedagentgroup;1</i> · depth 1"]

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
  G4 -->|ChildAgentGroups| G5
  G5 -->|ChildAgentGroups| G6A
  G5 -->|ChildAgentGroups| G6B

  G3 -->|Agents| A1
  G4 -->|Agents| A2
  G5 -->|Agents| A3
  G6A -->|Agents| A4
  G6A -->|Agents| A5
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
| `weld-robot-07`, `torque-station-03` | nordstern-motors › europe › austria › vienna › plant-north › body-shop › assembly-line-1 |
| `press-line-2` | nordstern-motors › europe › austria › vienna › plant-north › body-shop |
| `energy-meter-main` | nordstern-motors › europe › austria › vienna › plant-north |
| `site-gateway-01` | nordstern-motors › europe › austria › vienna |
| `agent-af31c9` | nordstern-motors › unassigned |

## What the demo shows

- **Depth 6 is the deepest legal agent group.** `assembly-line-1` and `assembly-line-2` sit there;
  creating a child below either is rejected.
- **Agents consume no depth level.** `weld-robot-07` hangs off a depth-6 group and is still valid.
- **Agents may sit on any level, not only on leaves.** Three agents are filed on the intermediate
  groups `vienna`, `plant-north` and `body-shop` — a site gateway, a plant energy meter and a shop
  press line each belong to their level rather than to a line.
- **An agent group may be empty.** `assembly-line-2` exists with no agents.
- **Uncategorized agents are a real node, not a computed set.** `agent-af31c9` was activated without
  an agent group, so it holds an ordinary `Agents` edge from the reserved `unassigned` group. The
  enrollment group itself never holds `Agents` edges.
- **Labels are localized, paths are not.** Every group carries `en`/`de` `DisplayNames`; the `Name`
  that forms the path stays untranslated, so `vienna` resolves identically in either locale.

## Size

| Element | Count |
|---|---|
| Node twins | 1 |
| Enrollment groups | 1 |
| Agent groups (incl. the reserved `unassigned`) | 8 |
| Agents | 6 |
| **Twins total** | **16** |
| Relationships | 15 |
| **Graph elements total** | **31** |
