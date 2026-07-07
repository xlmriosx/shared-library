---
name: github-actions-shared-library-agnostic-scaffold
description: Enforce a specific naming convention scaffold for GitHub Actions workflows depending on their purpose (CI, cloud deployments, specific tools).
---

# GitHub Actions Agnostic Scaffold

When creating GitHub Actions workflow files, adhere to the following scaffolding and naming conventions for the `.yml` files in the `.github/workflows` directory.

## 1. Cloud Deployments
For deployments to cloud providers, use the format: `<CLOUD>-<SERVICE>-<action>.yml`
- **`<CLOUD>`**: The cloud provider (e.g., AWS, AZ, GCP).
- **`<SERVICE>`**: The specific service being deployed to or interacted with (e.g., S3, ECR, ACR, ECS).
- **`<action>`**: The action being performed (e.g., deploy, push).

**Examples:**
- `AWS-S3-deploy.yml`
- `AWS-ECR-push.yml`
- `AZ-ACR-push.yml`
- `AWS-ECS-deploy.yml`

## 2. Tool-Specific Cloud Deployments
If the deployment uses a specific tool (like Helm, Terraform) within a cloud service, append it to the action: `<CLOUD>-<SERVICE>-deploy-<tool>.yml`

**Examples:**
- `AZ-AKS-deploy-helm.yml`
- `AWS-EKS-deploy-terraform.yml`

## 3. Continuous Integration (CI)
For CI workflows related to a specific programming language or framework, use the format: `ci-<language/framework>.yml`

**Examples:**
- `ci-python.yml`
- `ci-react.yml`
- `ci-node.yml`

Always verify that the workflow names strictly follow these conventions when creating or updating GitHub Actions in this project.
