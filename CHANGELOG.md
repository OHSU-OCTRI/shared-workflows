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
- Added a workflow to release Node.js projects (CIS-3818)

### Changed

- Handle self-contained image builds without artifacts (RFS-256)
- Output image digest from image build workflows to facilitate GitOps workflows (CIS-3818)

### Fixed

- Fixed an issue where the next development version after release was set to the release version

### Dependencies

- Bumps [actions/checkout](https://github.com/actions/checkout) from 6 to 7
- Bumps [actions/download-artifact](https://github.com/actions/download-artifact) from 7 to 8
- Bumps [stefanzweifel/git-auto-commit-action](https://github.com/stefanzweifel/git-auto-commit-action) from 7.1.0 to 7.2.0
- Bumps [actions/setup-node](https://github.com/actions/setup-node) from 6 to 7
- Bumps [docker/login-action](https://github.com/docker/login-action) from 4 to 4.5.2
- Bumps [actions/setup-java](https://github.com/actions/setup-java) from 5 to 5.6.0
