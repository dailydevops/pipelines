# Daily DevOps & .NET - Pipelines

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub Actions](https://img.shields.io/badge/GitHub-Actions-2088FF?logo=github-actions&logoColor=white)](https://github.com/dailydevops/pipelines/actions)

This repository contains reusable GitHub Actions workflow templates for .NET applications. These templates help automate the process of building, testing, and deploying applications, following best practices and reducing boilerplate code across projects.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Main Workflow Templates](#main-workflow-templates)
  - [build-dotnet-fast.yml](#build-dotnet-fastyml)
  - [build-dotnet-single.yml](#build-dotnet-singleyml)
  - [build-dotnet-matrix.yml](#build-dotnet-matrixyml)
  - [cicd-dotnet.yml](#cicd-dotnetyml)
  - [publish-nuget.yml](#publish-nugetyml)
- [Step Templates](#step-templates)
  - [step-node-commitlint.yml](#step-node-commitlintyml)
  - [step-dotnet-version.yml](#step-dotnet-versionyml)
  - [step-dotnet-format.yml](#step-dotnet-formatyml)
  - [step-dotnet-build.yml](#step-dotnet-buildyml)
  - [step-dotnet-tests.yml](#step-dotnet-testsyml)
  - [step-dotnet-draft-release.yml](#step-dotnet-draft-releaseyml)
  - [step-dotnet-publish-nuget.yml](#step-dotnet-publish-nugetyml)
  - [step-dependabot-merge.yml](#step-dependabot-mergeyml)
- [Common Usage Examples](#common-usage-examples)
- [Additional Documentation](#additional-documentation)
- [Contributing](#contributing)
- [License](#license)

## Overview

The pipelines in this repository are designed to be composable and reusable. They provide:

- **Automated CI/CD** for .NET applications
- **Code quality checks** including formatting and linting
- **Testing** with coverage reporting and multiple OS support
- **Versioning** using GitVersion
- **Publishing** to NuGet with manual approval workflows
- **Dependabot integration** for automated dependency updates

All workflows are designed to work with .NET 8.x and 9.x by default, with configurable support for other versions.

## Prerequisites

Before using these workflows, ensure you have:

- A GitHub repository with a .NET solution or project
- GitHub Actions enabled in your repository
- Required secrets configured (see individual workflow documentation)
- (Optional) GitVersion configuration file (`GitVersion.yml`) in your repository root
- (Optional) Release Drafter configuration (`.github/release-drafter.yml`) for draft releases

### Common Secrets

Many workflows support the following optional secret:

- **`FETCH_TOKEN`**: A GitHub Personal Access Token (PAT) with repository access, used to fetch dependencies from private repositories. If not provided, the default `github.token` is used.

## Quick Start

To use any of these workflows in your repository, reference them in your workflow file:

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    uses: dailydevops/pipelines/.github/workflows/build-dotnet-single.yml@main
    with:
      solution: "src/YourApp.sln"
    secrets: inherit
```

For specific versions, replace `@main` with a tag like `@v1.0.0`.

## Main Workflow Templates

These are complete CI/CD workflows that orchestrate multiple jobs.

### `build-dotnet-fast.yml`

A fast CI pipeline that performs a single-pass build and test on a single OS. Ideal for quick feedback on pull requests.

**Features:**
- Commit linting
- Build and test in one job (no separate stages)
- Automatic Dependabot PR merging

**Usage:**

```yaml
jobs:
  build:
    uses: dailydevops/pipelines/.github/workflows/build-dotnet-fast.yml@main
    with:
      solution: "src/YourApp.sln"
    secrets: inherit
```

**Parameters:**

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `solution` | Path to the solution or project file | ✅ | |
| `disableCodeFormat` | Skip code formatting checks | ❌ | `false` |
| `disableCoverageUpload` | Skip uploading coverage reports | ❌ | `false` |
| `disableDependaMerge` | Disable automatic Dependabot merge | ❌ | `false` |
| `dotnetLogging` | Verbosity level for dotnet commands | ❌ | `minimal` |
| `dotnetVersion` | .NET SDK version(s) to install | ❌ | `8.x\n9.x` |
| `dotnetQuality` | .NET SDK quality channel | ❌ | `ga` |
| `enableCleanUpDockerDocker` | Free disk space by cleaning Docker images | ❌ | `false` |
| `runsOnBuild` | Runner label for build job | ❌ | `ubuntu-latest` |

**Secrets:**
- `FETCH_TOKEN` (optional): GitHub token for private repository access

---

### `build-dotnet-single.yml`

A comprehensive CI pipeline that runs on a single OS with separate stages for formatting, versioning, testing, and building.

**Features:**
- Commit linting
- Code formatting validation
- Version detection using GitVersion
- Testing with coverage reports
- Build with artifact publishing
- Automatic Dependabot PR merging

**Usage:**

```yaml
jobs:
  build:
    uses: dailydevops/pipelines/.github/workflows/build-dotnet-single.yml@main
    with:
      solution: "src/YourApp.sln"
    secrets: inherit
```

**Parameters:**

Same as `build-dotnet-fast.yml`.

**Secrets:**
- `FETCH_TOKEN` (optional): GitHub token for private repository access

---

### `build-dotnet-matrix.yml`

A comprehensive CI pipeline that runs tests across multiple operating systems (Windows, Linux, macOS) in a matrix strategy.

**Features:**
- All features from `build-dotnet-single.yml`
- Multi-OS testing (Windows, Linux, macOS)
- Configurable fail-fast behavior
- OS-specific test exclusions

**Usage:**

```yaml
jobs:
  build:
    uses: dailydevops/pipelines/.github/workflows/build-dotnet-matrix.yml@main
    with:
      solution: "src/YourApp.sln"
    secrets: inherit
```

**Parameters:**

All parameters from `build-dotnet-single.yml`, plus:

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `disableTestsOnLinux` | Skip tests on Linux | ❌ | `false` |
| `disableTestsOnMacOs` | Skip tests on macOS | ❌ | `false` |
| `disableTestsOnWindows` | Skip tests on Windows | ❌ | `false` |
| `failFast` | Stop all jobs if one fails | ❌ | `true` |

**Secrets:**
- `FETCH_TOKEN` (optional): GitHub token for private repository access

---

### `cicd-dotnet.yml`

A deprecated/experimental workflow template. This workflow is currently set to fail on purpose and should not be used.

---

### `publish-nuget.yml`

A workflow for publishing NuGet packages with manual approval. This workflow verifies a successful build on the main branch and requires manual approval before publishing to NuGet.

**Features:**
- Cross-workflow artifact downloading
- Build verification on main branch
- Manual approval via GitHub Environments
- Automatic package publishing to NuGet.org
- Automated release creation

**Usage:**

```yaml
name: Publish to NuGet

on:
  workflow_dispatch:

jobs:
  publish:
    uses: dailydevops/pipelines/.github/workflows/publish-nuget.yml@main
    with:
      artifactPattern: "release-packages-*"
      workflowName: "build-dotnet-single.yml"
      runId: "1234567890"  # Get this from the workflow run you want to publish
      environment: "nuget-production"
    secrets:
      NUGET_TOKEN: ${{ secrets.NUGET_TOKEN }}
      WORKFLOW_PACKAGES: ${{ secrets.GITHUB_TOKEN }}
```

**Parameters:**

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `artifactPattern` | Pattern to match artifact names | ✅ | `release-packages-*` |
| `workflowName` | Workflow name to check for builds | ✅ | `ci-dotnet.yml` |
| `runId` | ID of workflow run to download artifacts from | ✅ | (none) |
| `environment` | GitHub Environment for manual approval | ✅ | `nuget-production` |

> **Note:** Parameters marked as required have default values (except `runId`), so you only need to specify them if you want to override the defaults. The default `workflowName` value `ci-dotnet.yml` is a legacy reference - you should explicitly specify your actual build workflow name (e.g., `build-dotnet-single.yml`, `build-dotnet-matrix.yml`, or `build-dotnet-fast.yml`).

**Secrets:**

| Secret | Description | Required |
|--------|-------------|----------|
| `NUGET_TOKEN` | NuGet API key for publishing | ✅ |
| `WORKFLOW_PACKAGES` | GitHub token for workflow/artifact access | ✅ |

**Prerequisites:**
- GitHub Environment configured with manual approval reviewers
- Source workflow must produce NuGet package artifacts
- See [Manual Approval Setup Guide](./docs/MANUAL_APPROVAL_SETUP.md) for detailed setup instructions

---

## Step Templates

These are individual job templates that can be composed into custom workflows or used independently.

### `step-node-commitlint.yml`

Validates commit messages using commitlint to ensure they follow conventional commit standards.

**Usage:**

```yaml
jobs:
  commitlint:
    uses: dailydevops/pipelines/.github/workflows/step-node-commitlint.yml@main
    secrets: inherit
```

**Parameters:** None

**Secrets:**
- `FETCH_TOKEN` (optional): GitHub token for private repository access

**Requirements:**
- `.commitlintrc` or similar configuration file in repository root

---

### `step-dotnet-version.yml`

Detects the version of your application using GitVersion based on commit history and branching strategy.

**Usage:**

```yaml
jobs:
  version:
    uses: dailydevops/pipelines/.github/workflows/step-dotnet-version.yml@main
    with:
      dotnet-version: "8.x"
    secrets: inherit
  
  # Use the version in another job
  another-job:
    needs: version
    runs-on: ubuntu-latest
    steps:
      - run: echo "Version is ${{ needs.version.outputs.solution-version }}"
```

**Parameters:**

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `dotnet-version` | .NET SDK version to install | ❌ | `8.x` |
| `dotnet-quality` | .NET SDK quality channel | ❌ | `ga` |
| `runs-on` | Runner label | ❌ | `ubuntu-latest` |

**Outputs:**

| Output | Description |
|--------|-------------|
| `solution-version` | Detected semantic version (e.g., `1.2.3`) |

**Secrets:**
- `FETCH_TOKEN` (optional): GitHub token for private repository access

**Requirements:**
- `GitVersion.yml` configuration file in repository root

---

### `step-dotnet-format.yml`

Validates code formatting using CSharpier to ensure consistent code style.

**Usage:**

```yaml
jobs:
  format:
    uses: dailydevops/pipelines/.github/workflows/step-dotnet-format.yml@main
    with:
      dotnet-version: "8.x"
    secrets: inherit
```

**Parameters:**

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `dotnet-logging` | Verbosity level | ❌ | `quiet` |
| `dotnet-version` | .NET SDK version | ❌ | `8.x` |
| `dotnet-quality` | .NET SDK quality channel | ❌ | `ga` |
| `runs-on` | Runner label | ❌ | `ubuntu-latest` |

**Secrets:**
- `FETCH_TOKEN` (optional): GitHub token for private repository access

**Requirements:**
- CSharpier will be installed automatically

---

### `step-dotnet-build.yml`

Builds the .NET solution and uploads build artifacts as `release-packages`.

**Usage:**

```yaml
jobs:
  build:
    uses: dailydevops/pipelines/.github/workflows/step-dotnet-build.yml@main
    with:
      solution: "src/YourApp.sln"
    secrets: inherit
```

**Parameters:**

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `solution` | Path to solution or project file | ✅ | |
| `disablePublish` | Skip uploading artifacts | ❌ | `false` |
| `dotnet-logging` | Verbosity level | ❌ | `quiet` |
| `dotnet-version` | .NET SDK version | ❌ | `8.x` |
| `dotnet-quality` | .NET SDK quality channel | ❌ | `ga` |
| `runs-on` | Runner label | ❌ | `ubuntu-latest` |

**Secrets:**
- `FETCH_TOKEN` (optional): GitHub token for private repository access

**Artifacts:**
- `release-packages`: Contains built NuGet packages (if any)

---

### `step-dotnet-tests.yml`

Runs tests for the .NET solution with code coverage reporting.

**Features:**
- Test execution with coverage collection
- ReportGenerator for coverage reports
- Codecov integration
- Optional SonarQube support
- Optional Docker image cleanup for disk space

**Usage:**

```yaml
jobs:
  version:
    uses: dailydevops/pipelines/.github/workflows/step-dotnet-version.yml@main
    secrets: inherit
  
  test:
    needs: version
    uses: dailydevops/pipelines/.github/workflows/step-dotnet-tests.yml@main
    with:
      solution: "src/YourApp.sln"
      solution-version: ${{ needs.version.outputs.solution-version }}
    secrets: inherit
```

**Parameters:**

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `solution` | Path to solution or project file | ✅ | |
| `solution-version` | Version from step-dotnet-version | ✅ | |
| `enableSonarQube` | Enable SonarQube analysis | ❌ | `false` |
| `enableCleanUpDockerDocker` | Free disk space | ❌ | `false` |
| `disableCoverageUpload` | Skip coverage artifacts | ❌ | `false` |
| `dotnet-logging` | Verbosity level | ❌ | `quiet` |
| `dotnet-version` | .NET SDK version | ❌ | `8.x` |
| `dotnet-quality` | .NET SDK quality channel | ❌ | `ga` |
| `runs-on` | Runner label | ❌ | `ubuntu-latest` |
| `retry-max-attempts` | Max retry attempts | ❌ | `3` |
| `retry-timeout` | Timeout in minutes | ❌ | `15` |

**Secrets:**
- `FETCH_TOKEN` (optional): GitHub token for private repository access
- `CODECOV_TOKEN` (optional): Codecov token for coverage upload
- `SONAR_ORGANIZATION` (optional): SonarQube organization
- `SONAR_PROJECT` (optional): SonarQube project key
- `SONAR_TOKEN` (optional): SonarQube token

**Artifacts:**
- `reportgenerator-{os}-{hash}`: Generated coverage reports
- `coverage-{os}-{hash}`: Raw coverage data files
- `logs-{os}-{hash}`: Test logs and SARIF files

---

### `step-dotnet-draft-release.yml`

Creates or updates a draft release using Release Drafter when code is pushed to the main branch.

**Usage:**

```yaml
jobs:
  version:
    uses: dailydevops/pipelines/.github/workflows/step-dotnet-version.yml@main
    secrets: inherit
  
  release:
    needs: version
    uses: dailydevops/pipelines/.github/workflows/step-dotnet-draft-release.yml@main
    with:
      solution-version: ${{ needs.version.outputs.solution-version }}
    secrets: inherit
```

**Parameters:**

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `solution-version` | Version from step-dotnet-version | ✅ | |
| `runs-on` | Runner label | ❌ | `ubuntu-latest` |

**Secrets:**
- `FETCH_TOKEN` (optional): GitHub token for private repository access
- `GITHUB_TOKEN` (required): Automatically provided

**Requirements:**
- `.github/release-drafter.yml` configuration file
- Only runs on pushes to `main` branch (not pull requests)

---

### `step-dotnet-publish-nuget.yml`

Publishes NuGet packages from artifacts to NuGet.org with manual approval via GitHub Environments.

**Usage:**

```yaml
jobs:
  publish:
    uses: dailydevops/pipelines/.github/workflows/step-dotnet-publish-nuget.yml@main
    with:
      artifactPattern: "release-packages-*"
      environment: "nuget-production"
      runId: "123456789"
      workflowName: "build-dotnet-single.yml"
    secrets:
      NUGET_TOKEN: ${{ secrets.NUGET_TOKEN }}
      WORKFLOW_PACKAGES: ${{ secrets.GITHUB_TOKEN }}
```

**Parameters:**

| Parameter | Description | Required | Default |
|-----------|-------------|----------|---------|
| `artifactPattern` | Pattern to match artifacts | ✅ | `release-packages-*` |
| `environment` | GitHub Environment for approval | ✅ | `nuget-production` |
| `runId` | Workflow run ID to download from | ✅ | |
| `workflowName` | Source workflow name | ✅ | `ci-dotnet.yml` |

> **Note:** Parameters marked as required have default values (except `runId`), so you only need to specify them if you want to override the defaults. The default `workflowName` value `ci-dotnet.yml` is a legacy reference - you should explicitly specify your actual build workflow name (e.g., `build-dotnet-single.yml`, `build-dotnet-matrix.yml`, or `build-dotnet-fast.yml`).

**Secrets:**

| Secret | Description | Required |
|--------|-------------|----------|
| `NUGET_TOKEN` | NuGet API key | ✅ |
| `WORKFLOW_PACKAGES` | GitHub token for artifacts | ✅ |

**Note:** Consider using the higher-level `publish-nuget.yml` workflow instead for a more complete publishing solution.

---

### `step-dependabot-merge.yml`

Automatically merges Dependabot pull requests after successful CI checks.

**Usage:**

```yaml
jobs:
  dependabot:
    if: github.actor == 'dependabot[bot]'
    needs: [build, test]  # Add your CI jobs here
    uses: dailydevops/pipelines/.github/workflows/step-dependabot-merge.yml@main
    secrets: inherit
```

**Parameters:** None

**Secrets:**
- All secrets are inherited from the calling workflow
- Requires appropriate `GITHUB_TOKEN` permissions to merge PRs

**Requirements:**
- Only runs on pull requests from `dependabot[bot]`
- All required status checks must pass before merge

---

## Common Usage Examples

### Basic CI/CD Pipeline

```yaml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build-and-test:
    uses: dailydevops/pipelines/.github/workflows/build-dotnet-single.yml@main
    with:
      solution: "src/MyApp.sln"
    secrets: inherit
```

### Multi-OS Testing

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    uses: dailydevops/pipelines/.github/workflows/build-dotnet-matrix.yml@main
    with:
      solution: "src/MyApp.sln"
      disableTestsOnMacOs: true  # Skip macOS if not needed
    secrets: inherit
```

### Custom Pipeline with Individual Steps

```yaml
name: Custom Pipeline

on:
  push:
    branches: [main]
  pull_request:

jobs:
  commitlint:
    uses: dailydevops/pipelines/.github/workflows/step-node-commitlint.yml@main
    secrets: inherit

  version:
    uses: dailydevops/pipelines/.github/workflows/step-dotnet-version.yml@main
    secrets: inherit

  format:
    needs: [commitlint]
    uses: dailydevops/pipelines/.github/workflows/step-dotnet-format.yml@main
    secrets: inherit

  test:
    needs: [commitlint, version]
    uses: dailydevops/pipelines/.github/workflows/step-dotnet-tests.yml@main
    with:
      solution: "src/MyApp.sln"
      solution-version: ${{ needs.version.outputs.solution-version }}
    secrets: inherit

  build:
    needs: [test, format]
    uses: dailydevops/pipelines/.github/workflows/step-dotnet-build.yml@main
    with:
      solution: "src/MyApp.sln"
    secrets: inherit

  draft-release:
    if: github.ref == 'refs/heads/main'
    needs: [build, version]
    uses: dailydevops/pipelines/.github/workflows/step-dotnet-draft-release.yml@main
    with:
      solution-version: ${{ needs.version.outputs.solution-version }}
    secrets: inherit
```

### Publishing to NuGet with Approval

```yaml
name: Publish to NuGet

on:
  workflow_dispatch:

permissions:
  actions: read
  contents: write

jobs:
  publish:
    uses: dailydevops/pipelines/.github/workflows/publish-nuget.yml@main
    with:
      artifactPattern: "release-packages-*"
      workflowName: "build-dotnet-single.yml"
      runId: "1234567890"  # Replace with actual workflow run ID
      environment: "nuget-production"
    secrets:
      NUGET_TOKEN: ${{ secrets.NUGET_TOKEN }}
      WORKFLOW_PACKAGES: ${{ secrets.GITHUB_TOKEN }}
```

## Additional Documentation

- [Manual Approval Setup Guide](./docs/MANUAL_APPROVAL_SETUP.md) - Detailed instructions for configuring GitHub Environments for manual approval workflows

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

When contributing:
1. Ensure your changes are backwards compatible
2. Update documentation for any parameter or behavior changes
3. Follow conventional commit standards
4. Test your changes in a real project before submitting

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

Copyright (c) 2023-2025 Daily DevOps & .NET
