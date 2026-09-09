# Unified namespace demo — graph representation

Companion diagram for [uns-demo.json](uns-demo.json). One enrollment group of a car manufacturer,
nested by geolocation from a country down to an assembly line — the maximum legal agent group
level of 5, with its agents one hop below at level 6. Three balanced country branches show that the
same path segment (`body-shop`, `assembly-line-1`) is only required to be unique among its siblings.

Levels are counted **from the enrollment group, which is level 0**: agent groups occupy levels 1..5
and agents sit at level 6 at the deepest.

## Graph

```mermaid
graph TD
  N["🖧 tributech-node-eu<br/><i>node;1</i> · level -1"]
  EG["🏢 nordstern-motors<br/><i>enrollmentgroup;1</i> · level 0"]
  U["📥 unassigned<br/><i>unassignedagentgroup;1</i> · level 1"]

  N -->|EnrollmentGroups| EG
  EG -->|UnassignedAgentGroup| U
  U -->|Agents| UA(["agent-af31c9"])

  subgraph AT["austria"]
    C1["🌍 austria<br/>level 1"]
    C1G["🏙 vienna<br/>level 2"]
    C1P["🏭 plant-north<br/>level 3"]
    C1S["🔧 body-shop<br/>level 4"]
    C1L1["🚗 assembly-line-1<br/>level 5"]
    C1L2["🚗 assembly-line-2<br/>level 5"]
    C1 --> C1G --> C1P --> C1S
    C1S --> C1L1
    C1S --> C1L2
    C1G -->|Agents| A11(["site-gateway-01"])
    C1P -->|Agents| A12(["energy-meter-main"])
    C1S -->|Agents| A13(["press-line-2"])
    C1L1 -->|Agents| A14(["weld-robot-07"])
    C1L1 -->|Agents| A15(["torque-station-03"])
  end

  subgraph DE["germany"]
    C2["🌍 germany<br/>level 1"]
    C2G["🏙 munich<br/>level 2"]
    C2P["🏭 plant-south<br/>level 3"]
    C2S["🔧 body-shop<br/>level 4"]
    C2L1["🚗 assembly-line-1<br/>level 5"]
    C2L2["🚗 assembly-line-2<br/>level 5"]
    C2 --> C2G --> C2P --> C2S
    C2S --> C2L1
    C2S --> C2L2
    C2G -->|Agents| A21(["site-gateway-02"])
    C2P -->|Agents| A22(["energy-meter-hall-2"])
    C2S -->|Agents| A23(["press-line-5"])
    C2L1 -->|Agents| A24(["weld-robot-12"])
    C2L1 -->|Agents| A25(["torque-station-08"])
  end

  subgraph CH["switzerland"]
    C3["🌍 switzerland<br/>level 1"]
    C3G["🏙 zurich<br/>level 2"]
    C3P["🏭 plant-west<br/>level 3"]
    C3S["🔧 body-shop<br/>level 4"]
    C3L1["🚗 assembly-line-1<br/>level 5"]
    C3L2["🚗 assembly-line-2<br/>level 5"]
    C3 --> C3G --> C3P --> C3S
    C3S --> C3L1
    C3S --> C3L2
    C3G -->|Agents| A31(["site-gateway-03"])
    C3P -->|Agents| A32(["energy-meter-hall-4"])
    C3S -->|Agents| A33(["press-line-9"])
    C3L1 -->|Agents| A34(["weld-robot-21"])
    C3L1 -->|Agents| A35(["torque-station-14"])
  end

  EG -->|AgentGroups| C1
  EG -->|AgentGroups| C2
  EG -->|AgentGroups| C3

  classDef agent fill:#e8f0fe,stroke:#4d7cc7,color:#1a2b45
  classDef reserved fill:#fdf0e3,stroke:#c78b3d,color:#452f1a,stroke-dasharray:4 3
  class A11,A12,A13,A14,A15,A21,A22,A23,A24,A25,A31,A32,A33,A34,A35,UA agent
  class U reserved
```

Unlabelled edges inside a country branch are `ChildAgentGroups`.

## Namespace paths

The `Name` of each agent group is a path segment; the path is the chain of `ChildAgentGroups` names
below the enrollment group. All paths below are prefixed with `nordstern-motors`.

| Agent | Namespace path |
|---|---|
| `weld-robot-07`, `torque-station-03` | austria › vienna › plant-north › body-shop › assembly-line-1 |
| `press-line-2` | austria › vienna › plant-north › body-shop |
| `energy-meter-main` | austria › vienna › plant-north |
| `site-gateway-01` | austria › vienna |
| `weld-robot-12`, `torque-station-08` | germany › munich › plant-south › body-shop › assembly-line-1 |
| `press-line-5` | germany › munich › plant-south › body-shop |
| `energy-meter-hall-2` | germany › munich › plant-south |
| `site-gateway-02` | germany › munich |
| `weld-robot-21`, `torque-station-14` | switzerland › zurich › plant-west › body-shop › assembly-line-1 |
| `press-line-9` | switzerland › zurich › plant-west › body-shop |
| `energy-meter-hall-4` | switzerland › zurich › plant-west |
| `site-gateway-03` | switzerland › zurich |
| `agent-af31c9` | unassigned |

## What the demo shows

- **Level 5 is the deepest legal agent group.** The `assembly-line-*` groups sit there; creating a
  child below any of them is rejected.
- **Agents consume no level of their own.** `weld-robot-07` hangs off a level-5 group and lands at
  level 6 — the deepest anything gets, and the maximum of the `depth` parameter.
- **Path segments are unique among siblings, not globally.** All three countries carry a
  `body-shop` and an `assembly-line-1`; only the full path disambiguates them.
- **Agents may sit on any level, not only on leaves.** In every country three agents are filed on
  the intermediate city, plant and shop groups — a site gateway, a plant energy meter and a shop
  press line each belong to their level rather than to a line.
- **An agent group may be empty.** `assembly-line-2` exists in each country with no agents.
- **Uncategorized agents are a real node, not a computed set.** `agent-af31c9` was activated without
  an agent group, so it holds an ordinary `Agents` edge from the reserved `unassigned` group. The
  enrollment group itself never holds `Agents` edges.
- **Labels are localized, paths are not.** Every group carries `en`/`de` `DisplayNames`; the `Name`
  that forms the path stays untranslated, so `zurich` resolves identically in either locale.
- **The node twin is level -1.** It is reachable only with `startOffset=-1`, and then carries only
  the one `EnrollmentGroups` edge that leads to the caller's own enrollment group.

## Size

| Element | Count |
|---|---|
| Node twins | 1 |
| Enrollment groups | 1 |
| Agent groups (incl. the reserved `unassigned`) | 19 |
| Agents | 16 |
| **Twins total** | **37** |
| Relationships | 36 |
| **Graph elements total** | **73** |
