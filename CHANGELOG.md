# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial commit of README.md
- Added secure asset item.json and collection.json examples
- Updated schema.json

- Added `security:schemas` Item and Collection property which defines all the security schemas.
- Added Asset parameter `security:refs` which specifies which schemes from `security:schemes` can be used for an asset.

### Changed

- Changed `type` keys to camelCase

- README.md updates for OpenAPI
- Updated secure asset item.json and collection.json for OpenAPI
- Updated schema.json for for OpenAPI

- README.md updates
- Updated secure asset item.json and collection.json to handle multiple authentification schemes
- Updated schema.json for the updated security object structure

- Updated README.md descriptions of the library

- Updated schema to secure_asset:schema to limit string values to the list of client names.

- Reformatted to align with the OpenAPI spec
- Re-added Updated schema to security:schemas make any `type` value valid
- Changed parameters in camelCase to snake_case

### Deprecated

### Removed

- Removed summaries example

### Fixed

[Unreleased]: <https://github.com/stac-extensions/secure-assets/compare/v1.0.0...HEAD>
