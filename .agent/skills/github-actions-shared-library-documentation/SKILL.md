---
name: github-actions-shared-library-documentation
description: Enforce the creation of comprehensive, standardized README.md files for every reusable workflow and composite action in a shared library.
---

# GitHub Actions Shared Library Documentation

A shared library is only as good as its documentation. When creating or modifying reusable workflows (in `.github/workflows/`) or composite actions (in `actions/*/action.yml`), you must provide a clear, standardized `README.md` file in the respective directories.

## Standard README Structure

Every `README.md` for an action or workflow must include:
1. **Title and Description**: A clear explanation of what the action/workflow does.
2. **Usage Example**: A copy-pasteable YAML snippet showing how to consume the workflow/action.
3. **Inputs Table**: A markdown table documenting all inputs.
4. **Secrets/Outputs Table** (if applicable): A markdown table for required secrets or returned outputs.

## Example Baseline: Action `README.md`

Use the following as a template for documenting any primitive action or reusable workflow.

```markdown
# AWS ECR Login Action

Logs into Amazon Elastic Container Registry (ECR) to allow subsequent Docker build and push commands.

## Usage

\`\`\`yaml
- name: Login to ECR
  uses: xlmriosx/shared-library/actions/registry-login-aws@v1
  with:
    aws-region: 'us-east-1'
    role-to-assume: 'arn:aws:iam::123456789012:role/my-github-actions-role'
\`\`\`

## Inputs

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| \`aws-region\` | string | Yes | N/A | The AWS region where the ECR repository is located. |
| \`role-to-assume\` | string | Yes | N/A | The AWS IAM Role ARN to assume via OIDC. |

## Outputs

| Name | Description |
|------|-------------|
| \`registry-url\` | The URL of the authenticated ECR registry. |
```

## Guidelines
- Always generate a `README.md` alongside any new `action.yml` or `.github/workflows/*.yml`.
- Keep the "Usage" snippet updated and ensure it reflects the latest version tag (e.g., `@v1` or `@main`).
- If inputs have default values, state them explicitly in the table.
