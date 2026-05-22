# Shared GitHub Actions Workflows

This repository contains reusable GitHub Actions workflows for OCTRI repositories.

## Workflows

### `java-build.yaml`

Builds, tests, and publishes a Maven project to GitHub Packages.

**Trigger:** `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|-------|------|----------|-------------|
| `java_version` | string | yes | Java version for `setup-java` |
| `publish_to_github_packages` | boolean | no | Whether to publish build to GitHub packages. Defaults to false. |
| `publish_to_maven_central` | boolean | no | Whether to publish to Maven Central. Defaults to false. |

**Required secrets:** `MAVEN_GPG_PRIVATE_KEY`, `MAVEN_GPG_PASSPHRASE`, `MAVEN_USERNAME`, `MAVEN_PASSWORD`

---

### `java-release.yaml`

Performs a full Maven release: updates the changelog, cuts a Git tag, publishes a GitHub Release, updates Kubernetes manifests, and triggers artifact builds.

**Trigger:** `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|-------|------|----------|-------------|
| `version` | string | yes | Version to release (e.g. `1.2.3`) |
| `development_version` | string | yes | Next development version (e.g. `1.3.0`; `-SNAPSHOT` is appended automatically) |
| `java_version` | string | yes | Java version for `setup-java` |
| `update_manifests` | boolean | no | Whether to update image versions in Kubernetes manifests. Defaults to false. Requires the `IMAGE_NAME` repository variable to be defined. |

**Required secrets/vars:** `GITHUB_TOKEN`, `IMAGE_NAME` (repository variable)

---

### `dependabot-changelog-update.yaml`

Automatically appends Dependabot dependency updates to `CHANGELOG.md` when a PR is labeled `dependencies`.

**Trigger:** `workflow_call`

## Usage

Reference these workflows from a caller workflow using `workflow_call`:

```yaml
jobs:
  build:
    uses: OHSU-OCTRI/shared-actions-workflows/.github/workflows/java-build.yaml@main
    with:
      java_version: "21"
    secrets: inherit
```
