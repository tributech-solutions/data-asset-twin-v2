# Tributech Node - Data Asset Twins

The Tributech Node follows the approach of describing all entities of the system and their relations using the open-source [Digital Twins Definition Language (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v4/DTDL.v4.md) standard.

Currently we support both [DTDL v2](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) (DTDL/V1 folder) and [DTDL v4](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v4/DTDL.v4.md) (DTDL/V2, DTDL/V3 and DTDL/V4 folders).

## Versioning

Each `DTDL/Vx` folder holds the models that were introduced or changed for that release — a model only reappears in a newer `Vx` folder if it actually changed; otherwise older interfaces keep referencing it at its last version. Each release also has a matching `vocabulary*.json` manifest (`vocabulary.json` for V1, `vocabulary-v2.json`, `vocabulary-v3.json`, `vocabulary-v4.json`) listing the raw file URLs added or changed in that version. See [CLAUDE.md](CLAUDE.md) for the full versioning and `@context` conventions.

## Tooling

The Tributech Node provides various services and additional tooling to work with the given definitions. The open source service of our tooling can be found at the following repositories:

[Demeter](https://github.com/tributech-solutions/tributech-demeter)


### Learn more

To learn more about our vision for digital twins amd their use-cases check out our blog series:

[Introduction to Digital Twins](https://www.tributech.io/blog/introduction-digital-twins)

[Digital Twins at Tributech](https://www.tributech.io/blog/digital-twins-at-tributech)

