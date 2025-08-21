# Setting Up Manual Approval for NuGet Publishing

This document explains how to configure GitHub environments for manual approval when using the `publish-nuget.yml` template.

## Prerequisites

- Repository admin access to configure environments
- Understanding of GitHub repository settings

## Configuration Steps

### 1. Create GitHub Environment

1. Go to your repository on GitHub
2. Navigate to **Settings** → **Environments**
3. Click **New environment**
4. Enter the environment name (default: `nuget-production`)
5. Click **Configure environment**

### 2. Configure Environment Protection Rules

In the environment configuration:

1. **Required reviewers**: Add users/teams who can approve deployments
   - Minimum 1 reviewer recommended
   - Consider adding multiple reviewers for redundancy

2. **Wait timer** (optional): Set a delay before deployment can proceed
   - Useful for additional verification time

3. **Deployment branches** (optional): Restrict which branches can deploy
   - Recommended: `main` branch only for production deployments

### 3. Environment Secrets (if needed)

If you need environment-specific secrets:

1. In the environment configuration, scroll to **Environment secrets**
2. Add any environment-specific secrets (e.g., different NUGET_TOKEN for staging vs production)

## Usage Example

Once the environment is configured, use the template like this:

```yaml
jobs:
  publish-nuget:
    uses: dailydevops/pipelines/.github/workflows/publish-nuget.yml@main
    with:
      environment: "nuget-production"  # Your configured environment name
    secrets:
      NUGET_TOKEN: ${{ secrets.NUGET_TOKEN }}
```

## Approval Process

When the workflow runs:

1. The **verify-build-status** job runs automatically
2. The **publish-nuget** job waits for manual approval
3. Designated reviewers receive a notification
4. Reviewers can approve/reject from the GitHub Actions UI
5. Once approved, the publishing proceeds

## Best Practices

- Use descriptive environment names (e.g., `nuget-production`, `nuget-staging`)
- Set up multiple reviewers to avoid single points of failure
- Document your approval process for team members
- Consider using branch protection rules in addition to environment protection
- Regularly review and audit approved deployments

## Troubleshooting

### "Environment not found" error
- Verify the environment name matches exactly (case-sensitive)
- Ensure the environment is created in the correct repository

### "No reviewers configured" error
- Add at least one reviewer to the environment protection rules
- Verify reviewers have appropriate repository permissions

### Approval notifications not received
- Check GitHub notification settings
- Verify reviewer email addresses and notification preferences
- Consider using teams instead of individual users for broader coverage