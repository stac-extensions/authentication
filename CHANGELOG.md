# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- `tokenExchange` pre-defined OAuth2 flow key for OAuth 2.0 Token Exchange
  ([RFC 8693](https://datatracker.ietf.org/doc/html/rfc8693)), with `tokenUrl` required
- `auth:refs` allowed on an Authentication Scheme Object, listing the scheme(s) that
  may supply this scheme's input — one-of semantics, matching `auth:refs` on Assets
  and Links; sequences are expressed as chains of references. Required with at least
  one entry for a scheme declaring a `tokenExchange` flow (the token obtained through
  whichever referenced scheme was used is the RFC 8693 `subject_token`), so the
  multi-step exchange is discoverable from the document; reference chains must be
  acyclic
- `subjectTokenType` field (optional) on the `tokenExchange` flow: the RFC 8693
  `subject_token_type` URN, for when the referenced scheme yields more than one kind
  of token; defaults from the `type` of the referenced scheme used

## [v1.1.0] - 2023-04-28

### Changed

- Clarify http and extend http authorization methods [#27](https://github.com/stac-extensions/authentication/pull/27)
- Split Authentication Flows Object into OAuth2 Flow Object and Signed URL Object
- The JSON Schema is much stricter compared to before, actually enforcing many restrictions documented in the written spec
- The Parameter Schemas must comply to JSON Schema draft-07 instead of OpenAPI Schema [#21](https://github.com/stac-extensions/authentication/issues/21)

### Fixed

- JSON Schema supports Catalogs
- Fixed various other issues in the JSON Schema
- Clarified which fields apply to which schema type
- Clarified required fields
- Fixed examples

### Removed

- Removed the provider-specific `planetaryComputer` and `earthdata` scheme types [#32](https://github.com/stac-extensions/authentication/pull/32)

## [v1.0.0] - 2023-11-07

### Added

- `responseField` to the authentication object. [#23](https://github.com/AtomicMaps/authentication/pull/23)
- PlanetaryComputer signedURL example.

- Added parameters object
- Support to describe signed URL request method and parmeters.

- Links examples in `README.md`

- `security:refs` parameter to Link objects in Items or Collections.

- Added `schemes` Item and Collection property which defines all the security schemas. 
- Added Asset parameter `refs` which specifies which schemes from `schemes` can be used for an asset.

### Changed

- Changed extension title to `authorization` and the field extension to `auth:`

- Changed `type` keys to camelCase

- Updated `item.json` and `collection.json` for OpenAPI
- Updated `schema.json` for for OpenAPI
- Updated `item.json` and `collection.json` to handle multiple authentication schemes
- Updated `schema.json` for the updated security object structure

- Updated schema to `schema` to limit string values to the list of client names.
- Re-added Updated schema to `schemas` make any `type` value valid

- Reformatted to align with the OpenAPI spec
- Changed parameters in camelCase to snake_case

### Deprecated

### Removed

- Removed summaries example

### Fixed

[Unreleased]: <https://github.com/stac-extensions/authentication/compare/v1.1.0...HEAD>
[v1.1.0]: <https://github.com/stac-extensions/authentication/compare/v1.0.0...v1.1.0>
[v1.0.0]: <https://github.com/stac-extensions/authentication/tree/v1.0.0>
