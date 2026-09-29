# Heimdall UI settings demo — graph representation

Companion diagram for [heimdall-ui-settings-demo.json](heimdall-ui-settings-demo.json): the tenant
settings (`heimdall:ui:tenant:settings;1`) and one user's settings (`heimdall:ui:user:settings;1`)
of the `nordstern-motors` tenant from [uns-demo.md](uns-demo.md). Both are Heimdall documents, not
twins on the node; they refer to agent groups and agents by twin id only.

## Graph

```mermaid
graph LR
  subgraph TS["tenant:settings;1"]
    direction TB
    T["⚙ Tenant Settings"]
    subgraph LV["AgentGroupLevels · outermost first"]
      direction LR
      L1["globe · blue<br/><b>1 country</b><br/>Country / Land"]
      L2["map-pin · cyan<br/><b>2 site</b><br/>Site / Standort"]
      L3["factory · green<br/><b>3 plant</b><br/>Plant / Werk"]
      L4["layer · orange<br/><b>4 area</b><br/>Area / Bereich"]
      L5["conveyor-belt · purple<br/><b>5 line</b><br/>Line / Linie"]
      L1 ~~~ L2 ~~~ L3 ~~~ L4 ~~~ L5
    end
    T --> LV
  end

  subgraph US["user:settings;1"]
    U["👤 User Settings"]
    NAV["AgentNavigation"]
    SP["StartPage"]
    S["StartAgentGroupId"]
    FAV["Favorites"]
    F1["AgentGroups[0]"]
    F2["AgentGroups[1]"]
    F3["AgentGroups[2]"]
    FA1["Agents[0]"]
    FA2["Agents[1]"]
    FA3["Agents[2]"]
    U --> NAV
    NAV --> SP --> S
    NAV --> FAV
    FAV --> F1 & F2 & F3
    FAV --> FA1 & FA2 & FA3
  end

  subgraph AG["agent groups and agents on the node (uns-demo)"]
    G1["austria"] --> G2["vienna"] --> G3["plant-north"] --> G4["body-shop"] --> G5["assembly-line-1"]
    H1["germany"] --> H2["munich"] --> H3["plant-south"] --> H4["body-shop"]
    K1["switzerland"] --> K2["zurich"]
    X1(["torque-station-03"])
    X2(["weld-robot-12"])
    X3(["site-gateway-03"])
  end

  S -.->|aaaa0103| G3
  F1 -.->|aaaa0105| G5
  F2 -.->|aaaa0204| H4
  F3 -.->|aaaa0302| K2
  FA1 -.->|dddd0105| X1
  FA2 -.->|dddd0204| X2
  FA3 -.->|dddd0301| X3

  classDef doc fill:#e8f0fe,stroke:#4d7cc7,color:#1a2b45
  classDef group fill:#f3f3f3,stroke:#888,color:#222
  classDef blue fill:#dbe8ff,stroke:#3b6fd4,color:#12264d
  classDef cyan fill:#d7f4f7,stroke:#1f9fb0,color:#0d3d44
  classDef green fill:#dcf3df,stroke:#34a04a,color:#133d1b
  classDef orange fill:#fde7d2,stroke:#d9822b,color:#4a2a0c
  classDef purple fill:#ece0fa,stroke:#8453c7,color:#2e1a4a
  class T,U,NAV,SP,S,FAV,F1,F2,F3,FA1,FA2,FA3 doc
  class X1,X2,X3 group
  class G1,H1,K1 blue
  class G2,H2,K2 cyan
  class G3,H3 green
  class G4,H4 orange
  class G5 purple
  class L1 blue
  class L2 cyan
  class L3 green
  class L4 orange
  class L5 purple
```

Solid edges are document structure, dashed edges are twin-id references into the node's topology.
An agent group takes its level's color and icon from its depth below the enrollment group, not
from an edge.

## What the demo shows

- **Levels style depth, not groups.** `AgentGroupLevels` holds five entries, one per depth 1..5;
  no level lists its agent groups.
- **`Name` is the identifier, labels are localized.** `plant` stays `plant`; `DisplayNames` carries
  `Plant` / `Werk`.
- **Favorites cross branches and depths.** The user stars a level-5 line in Austria, a level-4
  area in Germany and a level-2 site in Switzerland; the array order is the display order.
- **Agents are starred separately.** `Favorites.Agents` holds agents by twin id, next to
  `Favorites.AgentGroups`; each list takes at most 25 entries.
- **The start page is independent of favorites.** The agents section opens on `plant-north`,
  which is not starred.
- **References are plain twin ids.** Deleting an agent group or agent on the node leaves a
  dangling id in the user document; the client has to tolerate it.
