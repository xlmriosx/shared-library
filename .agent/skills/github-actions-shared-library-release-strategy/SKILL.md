---
name: github-actions-shared-library-release-strategy
description: Enforce a tagging and release strategy (Semantic Versioning) for shared GitHub Actions libraries.
---

# GitHub Actions Shared Library Release Strategy

For a shared library to be safely consumed by other repositories, it must provide stable version tags. Relying purely on `@main` or `@master` can break downstream consumers if a backwards-incompatible change is merged.

## Standard Release Principles

When managing or advising on the release cycle of a shared GitHub Actions library, adhere to these principles:

1. **Semantic Versioning (SemVer)**: Use tags like `v1.2.3` (Major.Minor.Patch).
2. **Major Floating Tags**: Maintain major floating tags (e.g., `v1`, `v2`) that point to the latest minor/patch release of that major version. This allows consumers to receive non-breaking updates automatically.
3. **Release Automation**: Encourage the use of a release workflow (e.g., using `release-please-action` or a custom script) to automatically create GitHub Releases and update floating tags.

## Example Baseline: Release Workflow (`.github/workflows/release.yml`)

When setting up a new shared library, include a workflow to automate tag updates on merge to `main`.

```yaml
name: Release and Tag

on:
  push:
    branches:
      - main

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # Example using a standard automated release tool
      - name: Release Please
        id: release
        uses: googleapis/release-please-action@v4
        with:
          release-type: simple

      # If a new release was created, update the major floating tag (e.g., v1)
      - name: Update Major Tag
        if: ${{ steps.release.outputs.release_created }}
        run: |
          MAJOR_VERSION=$(echo "${{ steps.release.outputs.tag_name }}" | cut -d. -f1)
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git tag -f "$MAJOR_VERSION"
          git push -f origin "$MAJOR_VERSION"
```

## Consumer Guidance

Always instruct consumer repositories to pin to a major version tag (e.g., `@v1`) rather than `@main`, unless they explicitly need edge features or are actively developing the library.

**Example of correct consumer usage:**
```yaml
    uses: xlmriosx/shared-library/.github/workflows/ci-python.yml@v1
```
