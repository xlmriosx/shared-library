---
name: github-actions-shared-library-development
description: Enforce standard structures for GitHub Actions in shared library development, incorporating parallel workflows, setup caching, primitive composite actions, and security scanning.
---

# GitHub Actions Shared Library Development

When creating or modifying GitHub Actions workflows for shared libraries, adhere to a highly optimized, parallelized, and secure structure. Ensure that jobs are clearly defined, execute in parallel where possible, leverage setup caching to speed up execution, and include mandatory security scans.

**Objective:** The primary goal of these workflows is to be designed as **Reusable Workflows** (`workflow_call`). They are developed centrally in the shared library so that other repositories can easily consume them.

## Standard Structure

Every workflow should generally include:
1.  **Name & Triggers**: Descriptive names and use `workflow_call` as the primary trigger so they can be consumed by other repositories. Define necessary `inputs` and `secrets`.
2.  **Primitive Composite Actions**: Break down complex or repetitive logic (e.g., Docker build & push, registry login, sending notifications) into local, primitive composite actions located in `actions/<action-name>/action.yml`. Reusable workflows should then orchestrate these primitives.
3.  **Parallel Jobs**: Design jobs (e.g., `lint`, `test`, `security-scan`) to run concurrently unless they explicitly depend on each other.
4.  **Setup & Caching**: Always use the native caching capabilities of the setup actions (e.g., `actions/setup-python`, `actions/setup-node`) to cache dependencies and speed up workflow runs.
5.  **Security Scans**: Include a dedicated job to scan dependencies and code for known vulnerabilities.
6.  **Cloud Authentication**: When deploying to cloud providers (AWS, Azure, GCP), design workflows to support both OIDC (OpenID Connect) and traditional long-lived credentials (API Keys/Secrets). Use conditional inputs to allow consuming repositories to choose either authentication method seamlessly.

## Primitive Composite Actions

Instead of writing long, inline shell scripts directly inside jobs, encapsulate specific domain tasks into primitive composite actions. 

**Examples of Primitives:**
- `actions/registry-login/action.yml` (Handles AWS ECR / Azure ACR logins)
- `actions/docker-build-push/action.yml` (Handles Docker build and push logic)
- `actions/notify/action.yml` (Handles agnostic chat notifications)

**Example usage within a reusable workflow:**
```yaml
      - name: Login to Container Registry
        uses: xlmriosx/shared-library/actions/registry-login@main
        with:
          registry-type: 'aws'
          
      - name: Build and Push Docker Image
        uses: xlmriosx/shared-library/actions/docker-build-push@main
        with:
          image-name: ${{ inputs.image-name }}
          tag: ${{ github.sha }}
```

## Example Baseline: Python Shared Library CI (`ci-python.yml`)

Use the following highly-parallel `ci-python` workflow as a reference architecture. Notice how it uses `workflow_call` to accept inputs from the consuming repository.

```yaml
name: Python Shared Library CI

on:
  workflow_call:
    inputs:
      working-directory:
        required: true
        type: string
      python-version:
        required: false
        type: string
        default: '3.12'
      pytest-target:
        required: false
        type: string
        default: 'tests'
    secrets:
      TEAMS_WEBHOOK_URL:
        required: false

jobs:
  lint:
    name: Lint Code
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ inputs.python-version }}
          cache: 'pip'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install flake8

      - name: Lint with flake8
        run: flake8 . --count --show-source --statistics

  security-scan:
    name: Security Scan
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ inputs.python-version }}
          cache: 'pip'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install bandit safety
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi

      - name: Run Bandit (Code Scanning)
        run: bandit -r . -c bandit.yaml || bandit -r .

      - name: Run Safety (Dependency Scanning)
        run: safety check || echo "No requirements found or safety check bypassed"

  test:
    name: Run Tests
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ inputs.python-version }}
          cache: 'pip'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install pytest pytest-cov
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi

      - name: Test with pytest
        run: pytest ${{ inputs.pytest-target }} --cov=./ --cov-report=xml
        
      # Example of calling a primitive composite action for notification
      - name: Notify on Completion
        if: always()
        uses: xlmriosx/shared-library/actions/notify@main
        with:
          status: ${{ job.status }}
          teams_webhook_url: ${{ secrets.TEAMS_WEBHOOK_URL }}
```

## Consuming the Reusable Workflow

Other repositories should consume the shared workflow as follows:

```yaml
name: CI

on:
  pull_request:
    branches:
      - dev

jobs:
  backend:
    uses: xlmriosx/shared-library/.github/workflows/ci-python.yml@main
    with:
      working-directory: 'backend'
      python-version: '3.12'
      pytest-target: 'tests'
    secrets:
      TEAMS_WEBHOOK_URL: ${{ secrets.TEAMS_WEBHOOK_URL }}
```

## Guidelines for adapting to other languages
-   **Reusable Workflows**: Always use `on: workflow_call` and define the appropriate inputs (`node-version`, `go-version`, etc.) for the specific language.
-   **Primitive Actions**: Break down complex CI/CD steps into `actions/*/action.yml` so they can be reused across different workflows (e.g., Python CI and Node.js CI could both share the same Docker Build & Push primitive action).
-   **Parallelization**: Regardless of language, separate `lint`, `test`, and `security-scan` into distinct jobs to execute them concurrently.
-   **Setup Action & Cache**: Leverage the native `cache` input provided by the official setup action.
-   **Security Tools**: Swap `bandit` and `safety` with standard tools for the respective ecosystem.
