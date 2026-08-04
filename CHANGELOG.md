# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- Initial commit
- Add workflow that runs Node.js tests
- Add infrastructure for container image builds (RFS-256)
- Add a workflow that builds the Maven project without publishing (CIS-3773)
- Added a workflow to audit and fix Node.js dependencies (CIS-3816)

### Changed

- Handle self-contained image builds without artifacts (RFS-256)

### Dependencies

- Bumps [actions/checkout](https://github.com/actions/checkout) from 6 to 7
- Bumps [actions/download-artifact](https://github.com/actions/download-artifact) from 7 to 8
