# Shared GitHub Actions Library

This repository (`xlmriosx/shared-library`) contains a centralized collection of reusable GitHub Actions workflows and composite actions. These shared resources enforce standardized CI/CD practices across all projects in the organization.

## Available Workflows

The following reusable workflows are available in `.github/workflows/`:

- **CI Workflows**:
  - `ci-react.yml`: Standard CI for React projects (lint, test, sec scan).
  - `ci-python.yml`: Standard CI for Python projects (lint, test, sec scan).
- **Deployment Workflows**:
  - `AWS-S3-deploy.yml`: Builds a Node.js project and deploys the static files to an AWS S3 bucket.
  - `AWS-ECR-push.yml`: Builds a Docker image and pushes it to an AWS Elastic Container Registry (ECR).

## Available Actions

- `notify`: A platform-agnostic composite action to send build status notifications to Microsoft Teams and Slack.

## Usage

To use a workflow from this repository in another project, use the `uses` keyword pointing to the specific workflow and branch/tag:

```yaml
jobs:
  ci:
    uses: xlmriosx/shared-library/.github/workflows/ci-react.yml@main
    with:
      working-directory: './frontend'
    secrets:
      TEAMS_WEBHOOK_URL: ${{ secrets.TEAMS_WEBHOOK_URL }}
```

Please refer to the specific `.yml` files for the full list of inputs and required secrets.

## Dependabot

This repository is configured with Dependabot to automatically check for updates to our GitHub Actions dependencies on a weekly basis, ensuring we use the latest and most secure versions of all actions.
