# data-asset-twin-v2

DTDL (Digital Twins Definition Language) model definitions for the Tributech Node / Tributech Edge Agent. This repo is pure data — JSON DTDL interface files, no build/test tooling.

## Repo layout

- `DTDL/V1`, `DTDL/V2`, `DTDL/V3`, `DTDL/V4` — versioned model folders, described below.
- `vocabulary.json`, `vocabulary-v2.json`, `vocabulary-v3.json`, `vocabulary-v4.json` — one vocabulary manifest per `DTDL/Vn` folder (see "Vocabulary files").
- `examples/` — example DTDL digital twin instances (`$metadata.$model` references) showing how models are used together, e.g. [ads-demo.json](examples/ads-demo.json), [opcua-demo.json](examples/opcua-demo.json).

## Versioning model (important — read before adding/editing any interface)

Every DTDL interface has a DTMI in the form:

```
dtmi:io:tributech:<category>:<name>;<version>
```

e.g. `dtmi:io:tributech:device:edge;4`, `dtmi:io:tributech:healthmessage:edge;2`, `dtmi:io:tributech:source:opcua;4`.

**The trailing `;N` is a per-model version counter, not the folder number.** Rules:

- A brand new model starts at `;1` in whichever `DTDL/Vx` folder it's first introduced in.
- When an existing model is changed, its version increments by exactly 1 and the new file is placed in the current/next `DTDL/Vx` folder.
- If a model is *not* changed for a given release, it is **not duplicated** into the new `Vx` folder — the old file (and its old `;N`) simply keeps being referenced from its original folder via `extends`/`schema`/`target`.

Example: `dtmi:io:tributech:device:base` was introduced as `;1` in `DTDL/V1/base-device.json`, bumped to `;2` in `DTDL/V2/base-device.json`, and has not changed since — there is no `base-device.json` in `DTDL/V3` or `DTDL/V4`; `edge.json` in those folders still does `"extends": "dtmi:io:tributech:device:base;2"`.

Another example: `dtmi:io:tributech:healthmessage:edge` was introduced as `;1` in [DTDL/V3/edge/edge-health-message.json](DTDL/V3/edge/edge-health-message.json), then changed and bumped to `;2` in [DTDL/V4/edge/edge-health-message.json](DTDL/V4/edge/edge-health-message.json) (Snapshot/Windowed telemetry restructure).

So: **the folder a file lives in tells you where it was last touched, not what its version number is.** Always check the actual `@id` in the file (or grep for the DTMI) rather than assuming `;N` == folder number `N`. In practice many "core" models (`device:edge`, `source:opcua`, ...) happen to have changed in every release so far, so their `;N` does line up with the folder number — but that's a coincidence of history, not a rule, and `device:base` above already breaks it.

When adding a new version of a model:
1. Copy the file into the current `DTDL/Vx` folder (create the folder if starting a new version line).
2. Bump `@id`'s `;N` by 1.
3. Bump `;N` in every other file's `extends`/`schema`/`target`/`request`/`response` reference that points at the old version, **only if** that referencing file is also being updated in this release. Otherwise leave older references pointing at the old version until that file's owner updates it.
4. Add the new file's raw GitHub URL to the corresponding `vocabulary-vX.json`.

## Model immutability (never edit a committed model file)

Published models are **strictly immutable**. Tributech Nodes (Demeter) store each model by exact DTMI and hash-check it on every vocabulary load: if the content served for an already-registered DTMI differs, the node logs `Wrong hashcode detected` / `Twin Model ... has changed but is not permitted to update` and **aborts loading that entire vocabulary manifest** — which then cascades into `No DtmiResolver provided` failures in later vocabulary sets that reference models from the aborted one.

Rules:

- **Never change the content of an existing model file** — no restructuring, no in-place `@id` bumps, not even inside the current-release `DTDL/Vx` folder. Any content change means a new `;N+1` version in a **new file**.
- When the model's file already lives in the current `DTDL/Vx` folder, add the new version as a *suffixed* file in the same folder (e.g. `opcua-source-5.json`, `opcua-health-message-3.json`) — don't create the next `Vx` folder unless the release line itself increments.
- **Inline `schemas` `@id`s are global within one vocabulary parse set — but only within that set.** Two files listed in the same `vocabulary-vX.json` must not both define the same `schemas` `@id`, even byte-identically — the DTDL parser rejects the whole set with `has more than one definition`. Byte-identical duplicates across *different* manifests parse fine and exist historically: the V2, V3, and V4 `simulated-source.json` all inline `command:response:simulated;2`, each listed in its own manifest.
- **Convention for new files: always bump every inline schema `@id` the file carries — even if the schema body is byte-identical, and even when the new file lands in a *different* manifest than its predecessor.** E.g. `opcua-source-5.json` defines `command:response:opcua;2` while `;4`'s file keeps `command:response:opcua;1`; the next `simulated-source` version defines `command:response:simulated;3` even if unchanged. This makes every `@id` in the repo globally unique, so any future manifest rearrangement or consolidation is collision-free by construction. Trade-off accepted: a bumped schema `;N` no longer implies the schema content changed. Never apply this retroactively to published files (immutability rule above).
- Removing a superseded file's URL from the vocabulary manifest makes that version unavailable to fresh installs ("models went missing"), so both versions stay listed.

**After creating or changing any model, validate it with the Demeter MCP tool `tribute_dtdl_model_validate`.** It resolves references against the live registry; a `TwinModelNotFound` for models that exist only in the working tree (not yet published) is expected and not a structural error — everything else must pass.

## `@context` conventions

- `DTDL/V1/**` uses DTDL spec v2: `"@context": "dtmi:dtdl:context;2"`.
- `DTDL/V2/**`, `DTDL/V3/**`, `DTDL/V4/**` use DTDL spec v4: `"@context": "dtmi:dtdl:context;4"`.
- Any interface that uses `ValueAnnotation` (i.e. telemetry/property annotated with `"annotates": "..."`, used heavily by health-message interfaces) must add the annotation extension to `@context` as an array:
  ```json
  "@context": ["dtmi:dtdl:context;4", "dtmi:dtdl:extension:annotation;2"]
  ```
  See [DTDL/V3/edge/edge-health-message.json](DTDL/V3/edge/edge-health-message.json) or [DTDL/V4/edge/edge-health-message.json](DTDL/V4/edge/edge-health-message.json). Interfaces without `ValueAnnotation` content use the plain single-string context.

## Vocabulary files (`vocabulary*.json`)

Each `vocabulary-vN.json` (and the unsuffixed `vocabulary.json` for V1) is a manifest listing the raw GitHub URLs of the DTDL files that belong to that version line — **not** a cumulative "latest state" list. `vocabulary-v4.json` currently only lists the 4 OPC UA files that exist under `DTDL/V4` (it doesn't re-list unchanged files from V1–V3). To resolve the *complete, current* model set for a consumer, you conceptually need to overlay `vocabulary.json` → `vocabulary-v2.json` → `vocabulary-v3.json` → `vocabulary-v4.json`, with later entries superseding earlier ones for the same DTMI base name.

When adding a new file to a `DTDL/Vx` folder, always add its raw URL to the matching `vocabulary-vX.json` (`vocabulary.json` for V1).

## Naming/category conventions

DTMI categories in use: `device`, `source`, `stream`, `parameter`, `options`, `healthmessage`, `command:response`, `sdk`, `oem`. Folder layout under each `DTDL/Vx` groups by transport/protocol (`edge/mqtt`, `edge/opcua`, `edge/rest`, `edge/simulated`, `edge/tcads`, `edge/syslog`, `edge/modbus`, `oem/emb`, `sdk`), each typically providing a `*-source.json`, `*-stream*.json`, optionally `*-parameter.json` and `*-health-message.json`.
