# Trivy Security Scan Action

Runs Aquasecurity Trivy vulnerability scanner to find vulnerabilities in the filesystem dependencies or in a built Docker image.

## Usage

```yaml
- name: Run Trivy FS Scan
  uses: xlmriosx/shared-library/actions/trivy-scan@main
  with:
    scan-type: 'fs'
    scan-ref: '.'
```

## Inputs

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `scan-type` | string | No | `fs` | Type of scan (`fs` or `image`). |
| `scan-ref` | string | No | `.` | Reference to scan (directory path for fs, image name for image). |
| `severity` | string | No | `CRITICAL,HIGH` | Severities to report. |
| `exit-code` | string | No | `1` | Exit code when vulnerabilities are found. |
