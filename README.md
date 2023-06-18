# Secure Assets Extension Specification

- **Title:** Secure Assets
- **Identifier:** <https://stac-extensions.github.io/secure-assets/v1.0.0/schema.json>
- **Field Name Prefix:** -
- **Scope:** Asset
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @jamesfisher-gis

The Secure Assets extension to the [STAC](https://github.com/radiantearth/stac-spec) specification provides a standard way to specify if an asset 
resides in secure storage. An alternative method (i.e. presigned URLs) would be required to access these assets. Presigned URL generation would 
require calling out to an external API. The aim of the Secure Assets extension is to provide a standard field for clients to recognise secure assets
 and access them via the user's preferred method. 

The Secure Assets extension is similar to the 
[Alternative Assets extension](https://github.com/stac-extensions/alternate-assets/tree/main#alternate-asset-object) which aims to specify alternate 
locations (e.g., URLs) for assets. Secure Assets aims to specify if an alternative dynamic method (presigned URLs) is needed.

- Examples:
  - [Item example](examples/item.json): Shows the basic usage of the extension in a STAC Item
  - [Collection example](examples/collection.json): Shows the basic usage of the extension in a STAC Collection
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields

The fields in the table below can be used in these parts of STAC documents:
- [ ] Catalogs
- [ ] Collections
- [ ] Item Properties (incl. Summaries in Collections)
- [x] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [ ] Links

| Field Name           | Type                      | Description |
| -------------------- | ------------------------- | ----------- |
| sa:security   | Map<string, [SecureAsset Object](#secure-asset-object)>  | Keyword for asset security level.  |

### Additional Field Information

#### security

An Asset property used to specify a keyword that defines the security level of the Asset.The most common values may be "private" or "public".
 However, other keyword may be used to for alternative methods or levels of security.

### Secure Asset Object

An Asset with the Secure Assets extension will have the following fields

| Field Name  | Type   | Description |
| ----------- | ------ | ----------- |
| href           | string | **REQUIRED**. URI to the asset. Relative and abolsolute URI are both allowed |
| title           | string | The displayed title for clients and users. |
| sa:security           | string | Keyword for asset security level |

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

### Running tests

The same checks that run as checks on PR's are part of the repository and can be run locally to verify that changes are valid. 
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on 
your command line run:
```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:
```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:
```bash
npm run format-examples
```
