# AWS ECR Push Action

Builds, scans, and pushes a Docker image to Amazon Elastic Container Registry (ECR).

## Usage

```yaml
- name: Build and Push to ECR
  uses: xlmriosx/shared-library/actions/aws-ecr-push@main
  with:
    aws-region: 'us-east-1'
    aws-role-arn: 'arn:aws:iam::123456789012:role/my-github-actions-role'
    ecr-repository: 'my-ecr-repo'
    image-tag: ${{ github.sha }}
```

## Inputs

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `working-directory` | string | No | `.` | The working directory containing the Dockerfile. |
| `aws-region` | string | No | `us-east-1` | The AWS region. |
| `ecr-repository` | string | Yes | N/A | The name of the ECR repository. |
| `image-tag` | string | Yes | N/A | The tag of the image to deploy. |
| `aws-role-arn` | string | No | N/A | The AWS IAM Role ARN to assume via OIDC. |
| `aws-access-key-id` | string | No | N/A | AWS Access Key ID. |
| `aws-secret-access-key` | string | No | N/A | AWS Secret Access Key. |

## Outputs

| Name | Description |
|------|-------------|
| `image-uri` | The full URI of the pushed image. |
