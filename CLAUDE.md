# data-asset-twin-v2

DTDL (Digital Twins Definition Language) model definitions for the Tributech Node / Tributech Edge Agent. This repo is pure data — JSON DTDL interface files, no build/test tooling.

## Repo layout

- `models/` — all DTDL interfaces, grouped by model family (see "Folder structure"). **One file per model version**; every version of a model lives together in that model's own subfolder.
- `vocabularies/release-N.json` — one **self-contained** manifest per release, listing the complete model set (latest version of every model) as of that release.
- `vocabulary.json` — the latest complete model set (same content as the newest `vocabularies/release-N.json`).
- `examples/` — example DTDL digital twin instances (`$metadata.$model` references) showing how models are used together, e.g. [ads-demo.json](examples/ads-demo.json), [opcua-demo.json](examples/opcua-demo.json).

## Versioning model (important — read before adding/editing any interface)

Every DTDL interface has a DTMI in the form:

```
dtmi:io:tributech:<category>:<name>;<version>
```

e.g. `dtmi:io:tributech:device:edge;4`, `dtmi:io:tributech:healthmessage:edge;2`, `dtmi:io:tributech:source:opcua;5`.

**The trailing `;N` is a per-model version counter, and it is always mirrored in the filename**: `models/edge/opcua/opcua-source/opcua-source-5.json` holds `dtmi:io:tributech:source:opcua;5`. There are no release folders — every model has its own subfolder (named after the model, matching the filename prefix) holding its complete version history (`ls models/edge/opcua/opcua-source/` shows every version of the OPC UA source model; `ls models/edge/opcua/` shows every OPC UA model).

Rules:

- **Model files are immutable.** Never edit an existing model file. Any change means a new version.
- A brand new model starts at `;1`: create `models/<family>/<name>/<name>-1.json`.
- A changed model bumps `;N` by exactly 1: copy the previous version's file to `<name>-<N+1>.json` in the **same model subfolder**, apply the change, and update the `@id`.
- References (`extends`/`schema`/`target`/`request`/`response`) in **other** models keep pointing at the old version until those models are themselves versioned up. Bump a reference only inside a file that is itself getting a new version in the same change.
- Unchanged models are never copied or touched.

## Folder structure

`models/` groups by family/protocol, not by release. Within a family (or protocol) folder, every model gets its own subfolder holding all of that model's versions:

```
models/
├── base/                  base-device/, base-source/, base-stream/,
│                          base-parameter/, base-options/, base-health-message/
├── options/               options-valuechange/
├── edge/                  edge/ (device), edge-stream/, edge-health-message/, …
│   ├── opcua/              opcua-source/, opcua-stream/, opcua-health-message/, opcua-parameter/
│   ├── mqtt/    rest/    simulated/    tcads/    syslog/    modbus/    (same pattern)
├── oem/                    oem/, oem-stream-base/, emb/
└── sdk/                    sdk/, sdk-source/, sdk-stream/, sdk-parameter/
```

Each model subfolder is named after the model (matching the filename prefix) and contains only that model's version files, e.g. `models/edge/opcua/opcua-source/opcua-source-1.json` … `opcua-source-5.json`.

Filename convention: `<model-name>-<version>.json` inside `models/<family>[/<protocol>]/<model-name>/`, where `<version>` equals the `;N` in the file's `@id`. Keep the model name aligned with the DTMI (e.g. `dtmi:io:tributech:stream:mqtt:jsonpath` → `models/edge/mqtt/mqtt-stream-jsonpath/mqtt-stream-jsonpath-<N>.json`).

## `@context` conventions

- Files created in the original V1 generation use DTDL spec v2: `"@context": "dtmi:dtdl:context;2"`. All newer files use DTDL spec v4: `"@context": "dtmi:dtdl:context;4"`. Check the file — never assume.
- New interfaces always use `dtmi:dtdl:context;4`.
- Any interface that uses `ValueAnnotation` (i.e. telemetry/property annotated with `"annotates": "..."`, used heavily by health-message interfaces) must add the annotation extension to `@context` as an array:
  ```json
  "@context": ["dtmi:dtdl:context;4", "dtmi:dtdl:extension:annotation;2"]
  ```
  See [models/edge/edge-health-message/edge-health-message-2.json](models/edge/edge-health-message/edge-health-message-2.json). Interfaces without `ValueAnnotation` content use the plain single-string context.

## Vocabulary files

Each `vocabularies/release-N.json` is **self-contained**: it lists the raw GitHub URL of the latest version of *every* model that is current as of release N. A consumer reads exactly one manifest — there is no overlay/merge across manifests. `vocabulary.json` at the repo root always equals the newest release manifest.

Raw URLs use the `main` branch: `https://raw.githubusercontent.com/tributech-solutions/data-asset-twin-v2/main/models/...`.

When releasing model changes:
1. Add the new model files under `models/` (see versioning rules above).
2. Create the next `vocabularies/release-N.json` by copying the previous one and replacing the superseded entries with the new versions' URLs (and adding entries for brand-new models).
3. Overwrite root `vocabulary.json` with the same content (name it "Tributech Core Vocabulary (Latest)").

## Naming/category conventions

DTMI categories in use: `device`, `source`, `stream`, `parameter`, `options`, `healthmessage`, `command:response`, `sdk`, `oem`. Folders under `models/` group by transport/protocol (`edge/mqtt`, `edge/opcua`, `edge/rest`, `edge/simulated`, `edge/tcads`, `edge/syslog`, `edge/modbus`, `oem/emb`, `sdk`), each typically providing `*-source-*.json`, `*-stream-*.json`, optionally `*-parameter-*.json` and `*-health-message-*.json`.

## Known dangling references (pre-existing)

These DTMIs are referenced by models in this repo but have no definition file here (defined externally or pending): `dtmi:io:tributech:annotations:base;1`, `dtmi:io:tributech:command:response:opcua;1`, `dtmi:io:tributech:command:response:simulated;2`, `dtmi:io:tributech:command:response:tcads;2`.
