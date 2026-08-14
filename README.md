# Tributech Node - Data Asset Twins

The Tributech Node follows the approach of describing all entities of the system and their relations using the open-source [Digital Twins Definition Language (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v4/DTDL.v4.md) standard.

We support both [DTDL v2](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) and [DTDL v4](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v4/DTDL.v4.md) — each interface declares its spec version in its own `@context` (first-generation interfaces use `dtmi:dtdl:context;2`, all newer ones `dtmi:dtdl:context;4`).

## Repository structure

- `models/` — all DTDL interfaces, grouped by model family/protocol (`base/`, `edge/opcua/`, `edge/mqtt/`, `sdk/`, …). Each model has its own subfolder (e.g. `edge/opcua/opcua-source/`) holding **one immutable file per model version**, named `<model-name>-<version>.json` where the version suffix always equals the `;N` in the file's DTMI `@id`.
- `vocabularies/release-N.json` — one self-contained manifest per release, listing the raw GitHub URLs of the **complete** model set (latest version of every model) as of that release. Consumers read exactly one manifest; no merging across releases is needed.
- `vocabulary.json` — always the latest complete model set (identical content to the newest release manifest).
- `examples/` — example digital twin instances showing how the models are used together.

## Versioning

The trailing `;N` of a DTMI (e.g. `dtmi:io:tributech:source:opcua;5`) is a per-model version counter. Model files are never edited in place — a change creates `<model-name>-<N+1>.json` next to its predecessor. See [CLAUDE.md](CLAUDE.md) for the full versioning, naming, and `@context` conventions.

## Tooling

The Tributech Node provides various services and additional tooling to work with the given definitions. The open source service of our tooling can be found at the following repositories:

[Demeter](https://github.com/tributech-solutions/tributech-demeter)

### Learn more

To learn more about our vision for digital twins and their use-cases check out our blog series:

[Introduction to Digital Twins](https://www.tributech.io/blog/introduction-digital-twins)

[Digital Twins at Tributech](https://www.tributech.io/blog/digital-twins-at-tributech)
