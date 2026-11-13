# Shared GitHub Actions Workflows

This repository contains reusable GitHub Actions workflows for OCTRI repositories.

## Usage

Reference these workflows from a caller workflow using `workflow_call`:

```yaml
jobs:
  build:
    uses: OHSU-OCTRI/shared-workflows/.github/workflows/java-build.yaml@main
    with:
      java_version: "21"
    secrets: inherit
```

### OCTRI SNAPSHOT dependencies from GitHub Packages

To avoid Maven Central rate limits, the Java workflows (`java-build.yaml`, `java-test-build.yaml`, `java-playwright.yaml` and `java-release.yaml`) set `OCTRI_GITHUB_SNAPSHOTS=true` on every job that uses `setup-java`'s default `github` server. These jobs pass `GITHUB_TOKEN` to their Maven steps and have at least `packages: read` permission.

Consumer projects should define an env-activated Maven profile that, when `OCTRI_GITHUB_SNAPSHOTS` is `true`, resolves OCTRI SNAPSHOT dependencies from `https://maven.pkg.github.com/OHSU-OCTRI/*` using the server id `github`. Otherwise, the project falls back to Central's snapshot repository, as in local builds. The variable has no effect on projects without such a profile.

The `publish-to-maven-central` job in `java-build.yaml` does not set the variable. Its `setup-java` configuration only provides the `central` server, so it keeps resolving SNAPSHOTs from Central.

Library repositories that consumers depend on as SNAPSHOTs should publish them to GitHub Packages:

- Set `publish_to_github_packages: true` in the caller workflow for builds of `main`.
- Keep publishing SNAPSHOTs to Central (`publish_to_maven_central: true`), so local builds without a GitHub token can still resolve them.
- Grant each consuming repository read access to the package under the package's **Manage Actions access** settings.

## Workflows

### `container-image-build.yaml`

Builds a container image and pushes to GitHub Container Registry. Optionally downloads a named artifact for use in the build, such as that created by `java-build.yaml`.

**Inputs:**
| Input | Type | Required | Description |
|-------|------|----------|-------------|
| `platforms` | string | no | Comma-separated string of platforms to include in the image (default: linux/amd64,linux/arm64). |
| `artifact_name` | string | no | Optional artifact to download and use in the build, as created by `actions/upload-artifact`. |
| `artifact_path` | string | no | Optional destination path for the downloaded artifact. |
| `image_tags` | string | no | Optional newline-separated tag specification, as accepted by docker/metadata-action. By default, releases are tagged with the application version extracted from the git tag, and merges to the default branch are tagged `latest`. |

**Outputs:**
| Output | Type | Description |
|--------|------|-------------|
| `digest` | string | Digest of the image in `sha256:HASH_VALUE` format |

**Trigger:** `workflow_call`

**Required secrets/vars:** `GITHUB_TOKEN`, `IMAGE_NAME` (repository variable)

An `IMAGE_NAME` repository variable is required to push the image to the registry. This should be set to the full `ghcr.io/OHSU-OCTRI/...` image name.

---

### `dependabot-changelog-update.yaml`

Automatically appends Dependabot dependency updates to `CHANGELOG.md` when a PR is labeled `dependencies`.

**Trigger:** `workflow_call`

---

### `java-build.yaml`

Builds, tests, and publishes a Maven project to GitHub Packages.

See `java-test-build.yaml` for a similar workflow that does not publish the build artifacts.

**Trigger:** `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|-------|------|----------|-------------|
| `java_version` | string | yes | Java version for `setup-java` |
| `publish_to_github_packages` | boolean | no | Whether to publish build to GitHub packages. Defaults to false. |
| `publish_to_maven_central` | boolean | no | Whether to publish to Maven Central. Defaults to false. |

**Required secrets:** `MAVEN_GPG_PRIVATE_KEY`, `MAVEN_GPG_PASSPHRASE`, `MAVEN_USERNAME`, `MAVEN_PASSWORD`

If the optional `APP_JAR_NAME` repository variable is defined, the specified JAR file in the `target` directory is uploaded for use by the `java-image-build.yaml` workflow.

---

### `java-image-build.yaml`

Builds a container image and pushes to GitHub Container Registry. Use the `java-build.yaml` workflow to build and upload the application JAR file required by this workflow.

**Inputs:**
| Input | Type | Required | Description |
|-------|------|----------|-------------|
| `platforms` | string | no | Comma-separated string of platforms to include in the image (default: linux/amd64,linux/arm64). |

**Outputs:**
| Output | Type | Description |
|--------|------|-------------|
| `digest` | string | Digest of the image in `sha256:HASH_VALUE` format |

**Trigger:** `workflow_call`

**Required secrets/vars:** `GITHUB_TOKEN`, `APP_JAR_NAME` (repository variable), `IMAGE_NAME` (repository variable)

The `APP_JAR_NAME` repository variable must be set to the name of the application JAR file in the `target` directory to ensure that the JAR file is available for the container build, and the `IMAGE_NAME` repository is required to push the image to the registry.

---

### `java-playwright.yaml`

Runs end-to-end tests for a Maven project using [Playwright for Java](https://playwright.dev/java/).

Playwright traces and Failsafe reports from `target/playwright-java/` and `target/failsafe-reports/` are uploaded as the `playwright-report` artifact (retained for 14 days) unless the run is cancelled.

The project must:

- Declare the Playwright dependency with `test` scope.
- Define an `e2e` Maven profile that configures `maven-failsafe-plugin` to run the end-to-end `*IT` test classes during the `verify` phase.
- Manage its own test database instance, if one is needed (for example, using [Testcontainers](https://testcontainers.com/)).
- Manage the creation and authentication of test accounts.

**Trigger:** `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|-------|------|----------|-------------|
| `java_version` | string | yes | Java version for `setup-java` |

**Required secrets:** None

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

### `java-test-build.yaml`

Builds and tests a Maven project for pull requests (without requiring access to any secrets).

See `java-build.yaml` for a similar workflow that publishes the build artifacts to GitHub Packages and Maven Central.

**Trigger:** `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|-------|------|----------|-------------|
| `java_version` | string | yes | Java version for `setup-java` |

---

### `new-image-notification.yaml`

Uses `repository_dispatch` to notify another repository when an image is pushed.

**Trigger:** `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|-------|------|----------|-------------|
| `notify_repo` | string | yes | The repository to notify of the new image |
| `image_name` | string | yes | The name of the image updated |
| `image_digest` | string | yes | The new image's sha256 digest, as output by container-image-build.yaml or java-image-build.yaml |
| `version_tag` | string | no | Optional image version tag (e.g. for a release) |

**Required secrets:** `dispatch_token`, which must be a personal access token with `contents: write` permission on `notify_repo`.
---

### `node-audit.yaml`

Fixes security vulnerabilities in Node.js dependencies and creates a PR using a new branch. Output produced by `audit-step` is included in the PR body, and may indicate potential issues regarding breaking changes or dependency conflicts.

Most likely causes of failure include:

- Permission issues related to `PR_AUTOMATION_TOKEN`, or
- Errors caused by `git`/`gh` due to unexpected state of branches and PRs

**Trigger:** `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|-------|------|----------|-------------|
| `node_version` | string | yes | Node.js version for `setup-node` |
| `branch_name` | string | yes | Name for the PR HEAD branch. |
| `commit_title` | string | no | Title for the git commit. |

**Required secrets:** `PR_AUTOMATION_TOKEN` (organization PAT, requires read/write permissions for contents and pull requests)

---

### `node-release.yaml`

Performs a Node.js release: updates the changelog, tags a new version, and publishes a GitHub Release. Optionally force-updates a major version tag, useful for GitHub Actions.

**Trigger:** `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|-------|------|----------|-------------|
| `version` | string | yes | Version to release |
| `node_version` | string | yes | Node.js version to use with the `setup-node` action |
| `update_major_version_tag` | boolean | no | Force update major version tag (e.g. for actions). Defaults to false. |

**Required secrets:** `GITHUB_TOKEN`

---

### `node-test.yaml`

Runs Node.js tests. Your `package.json` file must have a script named `test:ci` that runs your test suite.

**Trigger:** `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|-------|------|----------|-------------|
| `node_version` | string | yes | Node.js version for `setup-node` |
| `run_type_checks` | boolean | no | Whether to run type checks. Defaults to false. Requires a script named `test:types`. |

**Required secrets:** None

---

### `update-manifests.yaml`

Updates the Kubernetes manifests in a repository in response to a new image being pushed. Updates the image digests in the named YAML files, commits the changes, and makes a tag if `version-tag` is set. Changes are then pushed back to the repository.

**Trigger:** `workflow_call`

**Inputs:**
| Input | Type | Required | Description |
|-------|------|----------|-------------|
| `image_name` | string | yes | Name of the image to update (e.g. ghcr.io/ohsu-octri/example) |
| `image_digest` | string | yes | New image digest to set in manifests (as output by java-image-build.yaml or container-image-build.yaml) |
| `yaml_files` | string | yes | List of YAML files to update. Comma or newline separated list of paths. |
| `version_tag` | string | no | Optional version tag to set in manifests (e.g. for staging/production) |

**Required secrets:** `GITHUB_TOKEN`