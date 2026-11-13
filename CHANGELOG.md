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
- Add new workflows to automate manifest updates when images are pushed (CIS-3850)
- Allow overriding container image tag configuration in `container-image-build.yaml`
- Add workflow that runs Playwright for Java e2e test (CIS-3948)

### Changed

- Handle self-contained image builds without artifacts (RFS-256)
- Output image digest from image build workflows to facilitate GitOps workflows (CIS-3818)
- Remove redundant branch checks from workflows that build images
- Updated parameter names for deprecated `server-user` and `server-password` in `java-build.yaml`
- Java workflows set `OCTRI_GITHUB_SNAPSHOTS=true` to resolve OCTRI SNAPSHOT dependencies from GitHub Packages
- `java-build.yaml` (`build-java-source` job), `java-test-build.yaml` and `java-playwright.yaml` replace `setup-java`'s Maven cache with `actions/cache` steps to reduce downloads from Maven Central
- `java-build.yaml` (`build-java-source` job), `java-test-build.yaml` and `java-playwright.yaml` remove the project's own artifacts from the local Maven repository before saving the cache

### Fixed

- Fixed an issue where the next development version after release was set to the release version
- Fixed a bug in the `new-image-notification.yaml` workflow where the normalized version wasn't used (CIS-3850)
- Use `MAVEN_ARGS` in java-build.yaml to circumvent an issue with Maven Publications until next release of setup-java action (CIS-3914)
- Resolves issue with Maven Publication, removing the `MAVEN_ARGS` temporary fix in favor of a new parameter to `setup-java` (CIS-3922)
- Fixed issue related to `server-id` values defined in `mvn-server-repository-origins`

### Dependencies

- Bumps [actions/checkout](https://github.com/actions/checkout) from 6 to 7
- Bumps [actions/download-artifact](https://github.com/actions/download-artifact) from 7 to 8
- Bumps [stefanzweifel/git-auto-commit-action](https://github.com/stefanzweifel/git-auto-commit-action) from 7.1.0 to 7.2.0
- Bumps [actions/setup-node](https://github.com/actions/setup-node) from 6 to 7
- Bumps [docker/login-action](https://github.com/docker/login-action) from 4 to 4.5.2
- Bumps [actions/setup-java](https://github.com/actions/setup-java) from 5 to 5.6.0
- Bumps [actions/setup-java] from 5.6.0 to 6.1.0 (CIS-3922)
