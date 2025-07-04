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

### `step-dotnet-tests.yml`

### `step-dotnet-version.yml`

### `step-node-commitlint.yml`
