# AWS Lambda Update Action

Updates an AWS Lambda function with a new container image from Amazon Elastic Container Registry (ECR). It handles AWS authentication, constructs the ECR image URI, and updates the Lambda function code.

## Usage

```yaml
- name: Update Lambda Image
  uses: xlmriosx/shared-library/actions/aws-lambda-update@main
  with:
    role-to-assume: 'arn:aws:iam::123456789012:role/my-github-actions-role'
    aws-region: 'sa-east-1'
    function-name: 'my-lambda-function'
    ecr-repository: 'my-ecr-repo'
    image-tag: ${{ github.sha }}
```

## Inputs

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `role-to-assume` | string | Yes | N/A | The AWS IAM Role ARN to assume via OIDC. |
| `aws-region` | string | Yes | N/A | The AWS region. |
| `function-name` | string | Yes | N/A | The name of the Lambda function to update. |
| `ecr-repository` | string | Yes | N/A | The name of the ECR repository. |
| `image-tag` | string | No | `${{ github.sha }}` | The tag of the image to deploy. |
