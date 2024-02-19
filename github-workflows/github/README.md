<img align="right" width="100" height="74" src="https://user-images.githubusercontent.com/8277210/183290025-d7b24277-dfb4-4ce1-bece-7fe0ecd5efd4.svg" />

# Estimate Dora Metrics for a Service

[![Slack](https://img.shields.io/badge/Slack-4A154B?style=for-the-badge&logo=slack&logoColor=white)](https://join.slack.com/t/devex-community/shared_invite/zt-1bmf5621e-GGfuJdMPK2D8UN58qL4E_g)

# Estimate Dora Metrics for a Service

This GitHub action allows you to quickly estimate dora metrics for a service (repository) via Port Actions with ease.

      start_time:
        description: The start time for the override, in ISO 8601 format (e.g., 2023-01-01T01:00:00Z)
        required: true
        type: string
      end_time:
        description: The end time for the override, in ISO 8601 format (e.g., 2023-01-01T01:00:00Z).
        required: true
        type: string
      new_on_call_user_id:
        description: The ID of the user who will be taking over the on-call duty
        required: true
        type: string

## Inputs
| Name                 | Description                                                                                          | Required | Default            |
|----------------------|------------------------------------------------------------------------------------------------------|----------|--------------------|
| start_time         | The start time for the override, in ISO 8601 format (e.g., 2023-01-01T01:00:00Z)     | true    | -                  |
| end_time              | The end time for the override, in ISO 8601 format (e.g., 2023-01-01T01:00:00Z)                                | true     | -                  |
| new_on_call_user_id              | The ID of the user who will be taking over the on-call duty                                                              | true    | -               |

## Quickstart - Estimate Dora Metrics for a Service

1. Create the following GitHub action secrets
* `PAGERDUTY_API_KEY` - PagerDuty API Key [learn more](https://support.pagerduty.com/docs/
* `PORT_CLIENT_ID` - Port Client ID [learn more](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token)
* `PORT_CLIENT_SECRET` - Port Client Secret [learn more](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token)

2. Install the Ports GitHub app from [here](https://github.com/apps/getport-io/installations/new).
3. Install Port's pager duty integration [learn more](https://github.com/port-labs/Port-Ocean/tree/main/integrations/pagerduty)
>**Note** This step is not required for this example, but it will create all the blueprint boilerplate for you, and also update the catalog in real time with the new incident created.
4. After you installed the integration, the blueprints `pagerdutyService` and `pagerdutyIncident` will appear, create the following action with the following JSON file on the `pagerdutyIncident` blueprint:

```json
{
  "identifier": "change_on_call_user",
  "title": "Change On-Call User",
  "icon": "pagerduty",
  "userInputs": {
    "properties": {
      "start_time": {
        "type": "string",
        "title": "Start Time",
        "description": "The start time for the override, in ISO 8601 format (e.g., 2023-01-01T01:00:00Z)",
        "icon": "pagerduty",
        "format": "date-time"
      },
      "end_time": {
        "title": "End Time",
        "description": "The end time for the override, in ISO 8601 format (e.g., 2023-01-01T01:00:00Z).",
        "icon": "pagerduty",
        "type": "string",
        "format": "date-time"
      },
      "new_on_call_user_id": {
        "icon": "pagerduty",
        "description": "The ID of the user who will be taking over the on-call duty",
        "title": "On Call User Id",
        "type": "string"
      }
    },
    "required": [
      "start_time",
      "end_time",
      "new_on_call_user_id"
    ],
    "order": [
      "start_time",
      "end_time",
      "new_on_call_user_id"
    ]
  },
    "invocationMethod": {
    "type": "GITHUB",
    "org": "your-github-organization",
    "repo": "your-github-repo",
    "workflow": "change-incident-owner.yaml",
    "omitUserInputs": false,
    "omitPayload": false,
    "reportWorkflowStatus": true
  },
  "trigger": "DAY-2",
  "description": "Change who is on call in pagerduty",
  "requiredApproval": false
}
```
>**Note** Replace the invocation method with your own repository details.

5. Create a workflow file under .github/workflows/dora-metrics.yaml with the following content:

```yml
name: Change Who is On Call In PagerDuty
on:
  workflow_dispatch:
    inputs:
      start_time:
        description: The start time for the override, in ISO 8601 format (e.g., 2023-01-01T01:00:00Z)
        required: true
        type: string
      end_time:
        description: The end time for the override, in ISO 8601 format (e.g., 2023-01-01T01:00:00Z).
        required: true
        type: string
      new_on_call_user_id:
        description: The ID of the user who will be taking over the on-call duty
        required: true
        type: string
      port_payload:
        required: true
        description: >-
          Port's payload, including details for who triggered the action and
          general context (blueprint, run id, etc...)

jobs:
  change-on-call-user:
    runs-on: ubuntu-latest
    steps:
      
      - name: Log Executing Request to Changing On-Call Owner
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: PATCH_RUN
          runId: ${{fromJson(github.event.inputs.port_payload).context.runId}}
          logMessage: "Making request to pagerduty ..."

      - name: Request Schedule Override
        uses: fjogeleit/http-request-action@v1
        with:
          url: "https://api.pagerduty.com/schedules/${{fromJson(github.event.inputs.port_payload).context.entity}}/overrides"
          method: 'POST'
          customHeaders: '{"Content-Type": "application/json", "Accept": "application/json", "Authorization": "Token token=${{ secrets.PAGERDUTY_API_KEY }}"}'
          data: >-
            {
              "overrides": [
                {
                  "start": "${{ github.event.inputs.start_time }}",
                  "end": "${{ github.event.inputs.end_time }}",
                  "user": {
                    "id": "${{ github.event.inputs.new_on_call_user_id }}",
                    "type": "user_reference"
                  },
                  "time_zone": "UTC"
                }
              ]
            }

      - name: Log Before Requesting for Updated Schedule
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: PATCH_RUN
          runId: ${{fromJson(github.event.inputs.port_payload).context.runId}}
          logMessage: "Getting updated schedule from pagerduty ..."

      - name: Request For Changed Schedule
        id: new_schedule
        uses: fjogeleit/http-request-action@v1
        with:
          url: 'https://api.pagerduty.com/schedules/${{fromJson(github.event.inputs.port_payload).context.entity}}'
          method: 'GET'
          customHeaders: '{"Content-Type": "application/json", "Accept": "application/json", "Authorization": "Token token=${{ secrets.PAGERDUTY_API_KEY }}"}'

      - name: Extract Users From New Schedule
        id: extract_users
        run: |
          USERS_JSON=$(echo '${{ steps.new_schedule.outputs.response }}' | jq -c '[.schedule.users[].summary]')
          echo "user_summaries=$USERS_JSON" >> $GITHUB_ENV
        shell: bash
  
      - name: Log Before Upserting Schedule to Port
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: PATCH_RUN
          runId: ${{fromJson(github.event.inputs.port_payload).context.runId}}
          logMessage: "Ingesting updated schedule to port..."
          
      - name: UPSERT Entity
        uses: port-labs/port-github-action@v1
        with:
          identifier: "${{ fromJson(steps.new_schedule.outputs.response).schedule.id }}"
          title: "${{ fromJson(steps.new_schedule.outputs.response).schedule.name }}"
          blueprint: ${{fromJson(github.event.inputs.port_payload).context.blueprint}}
          properties: |-
            {
              "url": "${{ fromJson(steps.new_schedule.outputs.response).schedule.html_url }}",
              "timezone": "${{ fromJson(steps.new_schedule.outputs.response).schedule.time_zone }}",
              "description": "${{ fromJson(steps.new_schedule.outputs.response).schedule.description}}",
              "users": ${{ env.user_summaries }}
            }
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: UPSERT
          runId: ${{ fromJson(inputs.port_payload).context.runId }}

      - name: Log After Upserting Entity
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: PATCH_RUN
          runId: ${{fromJson(github.event.inputs.port_payload).context.runId}}
          logMessage: "Entity upserting was successful ✅"
```

Congrats 🎉 You've successfully changed who is on call for the first time from Port!