# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).


## [2.0] (https://api.departureboard.io/api/v2.0/) - 10-09-2019
### Added
1. Added the "length" field to `subsequentCallingPoints` and `previousCallingPoints`.

### Changed
1. The `subsequentCallingPoints` array has been moved into an `subsequentCallingPointsList` array to support services that seperate, and consequently have two sets of subsequentCallingPoints.
2. The `previousCallingPoints` array has been moved into an `previousCallingPointsList` array to support services that seperate, and consequently have two sets of previousCallingPoints.

## [1.0] (https://api.departureboard.io/api/v1.0/) - 16-06-2019
### Added
- Initial Release
