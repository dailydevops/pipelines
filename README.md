# Daily DevOps & .NET - Pipelines

This repository contains several workflow templates for the daily usage inside the Daily DevOps & .NET team. We try to automate as much as possible to make our lives easier. For that reason, we have created several GitHub Actions workflows that will help us to automate the process of building, testing, and deploying our applications.

## Workflows

At the moment, we have concentrated on creating workflows for .NET applications. We have created two different workflows for .NET applications. These are a composition of several jobs that will help us to automate the process of building, testing, and deploying our applications.

All jobs are separated into templates that can be reused in other workflows. More details about the templates can be found in the [Templates](#templates) section.

### `ci-dotnet-fast.yml`

### `ci-dotnet.yml`

## Templates

### `step-dependabot-merge.yml`

This template is used to merge the pull request created by the dependabot. The template will check if the pull request is created by the dependabot and if the pull request is ready to be merged. If the conditions are met, the pull request will be merged.

To use this template, you need to add the following code to your workflow file:

```yaml
jobs:
  merge-dependabot:
    runs-on: ubuntu-latest
    steps:
      - uses: dailydevops/pipelines/.github/workflows/step-dependabot-merge.yml@0.12.16
        secrets: inherits
```

#### Secrets

This step requires the secret `DEPENDABOT` to be set in the repository. The secret should contain the GitHub token that has the permissions to merge the pull requests.

### `step-dotnet-build.yml`

This template is used to build the .NET application. The template will build the application using the `dotnet build` command, and upload the build artifacts to the GitHub as an artifact `release-packages`.

To use this template, you need to add the following code to your workflow file:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: dailydevops/pipelines/.github/workflows/step-dotnet-build.yml@0.12.16
        with:
          solution: "src/MyApp.sln"
```

#### Parameters

| Parameter        | Description                                    | Required | Default         |
| ---------------- | ---------------------------------------------- | :------: | --------------- |
| `disablePublish` | Disable the publishing of the build artifacts. |    ❌    | `false`         |
| `dotnet-logging` | The verbosity of the dotnet build command.     |    ❌    | `quiet`         |
| `dotnet-version` | The version of the dotnet SDK to use.          |    ❌    | `8.x`           |
| `dotnet-quality` | The quality of the dotnet SDK to use.          |    ❌    | `ga`            |
| `runs-on`        | The runner to use.                             |    ❌    | `ubuntu-latest` |
| `solution`       | The path to the solution or project file.      |    ✅    |                 |

#### Secrets

This step has an _optional_ secret `FETCH_TOKEN` that can be used to fetch the dependencies from the private repositories. The secret should contain the GitHub token that has the permissions to fetch the dependencies. If the secret is not set, the dependencies will be fetched using the default `{{ github.token }}`.

### `step-dotnet-draft-release.yml`

### `step-dotnet-format.yml`

### `step-dotnet-publish-nuget.yml`

### `publish-nuget.yml`

This template is used to publish .NET NuGet packages with manual approval and cross-workflow artifact support. The template downloads artifacts from a specified pipeline run, verifies the last successful build on the main branch, and requires manual approval before publishing.

To use this template, you need to add the following code to your workflow file:

```yaml
jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: dailydevops/pipelines/.github/workflows/publish-nuget.yml@main
        with:
          source-workflow-name: "ci-dotnet.yml"
          environment: "nuget-production"
        secrets:
          NUGET_TOKEN: ${{ secrets.NUGET_TOKEN }}
```

#### Parameters

| Parameter              | Description                                                                | Required | Default              |
| ---------------------- | -------------------------------------------------------------------------- | :------: | -------------------- |
| `source-repo`          | Repository containing the workflow run with artifacts.                    |    ❌    | `${{ github.repository }}` |
| `source-workflow-name` | Name of the workflow to check for successful builds.                      |    ❌    | `ci-dotnet.yml`      |
| `artifact-name`        | Name of the artifact to download.                                         |    ❌    | `release-packages`   |
| `environment`          | Environment name for manual approval (must be configured in repository).  |    ❌    | `nuget-production`   |

#### Secrets

| Secret       | Description                                           | Required |
| ------------ | ----------------------------------------------------- | :------: |
| `NUGET_TOKEN` | NuGet API key for publishing packages to nuget.org  |    ✅    |
| `GITHUB_TOKEN` | GitHub token for accessing workflow runs and artifacts |    ❌    |

#### Prerequisites

- The target repository must have a GitHub environment configured (e.g., `nuget-production`) with manual approval reviewers
- The source workflow must have completed successfully on the main branch
- The source workflow must produce artifacts with NuGet packages

For detailed instructions on setting up the GitHub environment for manual approval, see [Manual Approval Setup Guide](./docs/MANUAL_APPROVAL_SETUP.md).

### `step-dotnet-tests.yml`

### `step-dotnet-version.yml`

### `step-node-commitlint.yml`
