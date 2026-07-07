---
name: github-actions-notify-action
description: Enforce standard structure for creating platform-agnostic notification composite actions (e.g., actions/notify/action.yml).
---

# GitHub Actions Platform-Agnostic Notify

When creating or modifying notification actions (specifically custom composite actions like `actions/notify/action.yml`), the objective is to build a **platform-agnostic** interface. Consuming workflows should not need to worry about the underlying REST calls or payload structures for different chat platforms (like Microsoft Teams, Slack, or Discord).

## Standard Structure for Notification Composite Actions

Your composite action (`action.yml`) should adhere to the following principles:

1.  **Platform Abstraction**: Provide an input to specify the target platform(s) (e.g., `teams`, `slack`, or `all`), or infer it based on which webhook URLs are provided.
2.  **Standardized Inputs**: Define generic inputs such as:
    *   `status`: The job status (e.g., `success`, `failure`, `cancelled`).
    *   `title` or `message`: The main content of the notification.
    *   `webhook_url` (or specific URLs like `teams_webhook_url`, `slack_webhook_url`).
3.  **Conditional Execution**: Route the notification to the correct platform using `if` conditions based on the inputs.
4.  **Action Type**: The action should be a `composite` action.

## Example Baseline: `actions/notify/action.yml`

Use the following as a reference architecture when building or updating the agnostic notify action.

```yaml
name: 'Agnostic Notify'
description: 'Send notifications to various platforms (Teams, Slack) based on job status'

inputs:
  status:
    description: 'The status of the job (success, failure, cancelled)'
    required: true
  teams_webhook_url:
    description: 'Microsoft Teams Webhook URL'
    required: false
  slack_webhook_url:
    description: 'Slack Webhook URL'
    required: false
  title:
    description: 'Custom title for the notification'
    required: false
    default: 'GitHub Actions Run Status'

runs:
  using: "composite"
  steps:
    # ---------------------------------------------------------
    # MICROSOFT TEAMS NOTIFICATION
    # ---------------------------------------------------------
    - name: Notify Microsoft Teams
      if: ${{ inputs.teams_webhook_url != '' }}
      shell: bash
      run: |
        COLOR="808080" # Default (Cancelled)
        if [ "${{ inputs.status }}" == "success" ]; then
          COLOR="28a745" # Green
        elif [ "${{ inputs.status }}" == "failure" ]; then
          COLOR="dc3545" # Red
        fi
        
        PAYLOAD=$(cat <<EOF
        {
          "@type": "MessageCard",
          "@context": "http://schema.org/extensions",
          "themeColor": "$COLOR",
          "summary": "${{ inputs.title }}",
          "sections": [{
            "activityTitle": "${{ inputs.title }}",
            "activitySubtitle": "Status: ${{ inputs.status }}",
            "facts": [
              { "name": "Repository:", "value": "${{ github.repository }}" },
              { "name": "Ref:", "value": "${{ github.ref }}" },
              { "name": "Actor:", "value": "${{ github.actor }}" }
            ],
            "markdown": true
          }]
        }
        EOF
        )
        
        curl -H "Content-Type: application/json" -d "$PAYLOAD" "${{ inputs.teams_webhook_url }}"

    # ---------------------------------------------------------
    # SLACK NOTIFICATION
    # ---------------------------------------------------------
    - name: Notify Slack
      if: ${{ inputs.slack_webhook_url != '' }}
      shell: bash
      run: |
        EMOJI="⚪" # Default (Cancelled)
        if [ "${{ inputs.status }}" == "success" ]; then
          EMOJI="✅"
        elif [ "${{ inputs.status }}" == "failure" ]; then
          EMOJI="❌"
        fi
        
        PAYLOAD=$(cat <<EOF
        {
          "text": "$EMOJI *${{ inputs.title }}*\n*Status:* ${{ inputs.status }}\n*Repo:* ${{ github.repository }}\n*Actor:* ${{ github.actor }}"
        }
        EOF
        )
        
        curl -X POST -H 'Content-type: application/json' --data "$PAYLOAD" "${{ inputs.slack_webhook_url }}"
```

## Consuming the Notify Action

Consumer workflows should call this action in a `post-run` step or at the end of a job, passing the `job.status` and the desired webhooks.

```yaml
      - name: Send Notification
        if: always() # Ensure this runs even if previous steps failed
        uses: ./actions/notify # Path to the local composite action
        with:
          status: ${{ job.status }}
          teams_webhook_url: ${{ secrets.TEAMS_WEBHOOK_URL }}
          title: 'Backend CI Deployment'
```
