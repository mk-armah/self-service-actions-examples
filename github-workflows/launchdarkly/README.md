<img align="right" width="100" height="74" src="https://user-images.githubusercontent.com/8277210/183290025-d7b24277-dfb4-4ce1-bece-7fe0ecd5efd4.svg" />

# Toggle A LaunchDarkly Feature Flag

[![Slack](https://img.shields.io/badge/Slack-4A154B?style=for-the-badge&logo=slack&logoColor=white)](https://join.slack.com/t/devex-community/shared_invite/zt-1bmf5621e-GGfuJdMPK2D8UN58qL4E_g)

This GitHub action allows you to quickly toggle LaunchDarkly Feature Flags via Port Actions with ease.


## Inputs
| Name                 | Description                                                                                          | Required | Default            |
|----------------------|------------------------------------------------------------------------------------------------------|----------|--------------------|
| feature_flag_key         | LaunchDarkly Feature Flag Key                                                       | true    | -                  |
| project_key              | The LaunchDarkly Project Key where the feature flag exists                                                     | true     | -                  |
| environment_key              | The LaunchDarkly Environment Key where the flag exists                                                              | true    | -               |
| flag_state           | Desired state of the feature flag (true for enabled, false for disabled)                                                | true     | true                  |


## Quickstart - Toggle a lauchdarkly feature flag from the service catalog

1. Create the following GitHub action secrets
* `LAUNCHDARKLY_ACCESS_TOKEN` - a token with permission to toggle a feature flag in LaunchDarkly [learn more](https://docs.launchdarkly.com/home/account-security/api-access-tokens)
* `PORT_CLIENT_ID` - Port Client ID [learn more](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token)
* `PORT_CLIENT_SECRET` - Port Client Secret [learn more](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token) 

2. Install the Ports GitHub app from [here](https://github.com/apps/getport-io/installations/new).
3. Install Port's LaunchDarkly integration [learn more](https://github.com/port-labs/Port-Ocean/tree/main/integrations/launchdarkly)
>**Note** This step is not required for this example, but it will create all the blueprint boilerplate for you, and also update the feature flag catalog in real time.
4. After you installed the integration, the blueprints `launchDarklyProject`, `launchDarklyFeatureFlag`, `launchDarklyAuditLog` and `launchDarklyEnvironment` will appear, create the following action with the following JSON file on the `launchDarklyFeatureFlag` blueprint:

```json
{
  "identifier": "toggle_a_feature_flag",
  "title": "toggle-a-feature-flag",
  "icon": "Launchdarkly",
  "userInputs": {
    "properties": {
      "feature_flag_key": {
        "icon": "Launchdarkly",
        "title": "feature_flag_key",
        "description": "LaunchDarkly Feature Flag Key",
        "type": "string",
        "blueprint": "launchDarklyFeatureFlag",
        "format": "entity"
      },
      "project_key": {
        "title": "project_key",
        "description": "LaunchDarkly Project Key",
        "icon": "Launchdarkly",
        "type": "string",
        "blueprint": "launchDarklyProject",
        "format": "entity"
      },
      "environment_key": {
        "icon": "Launchdarkly",
        "description": "LaunchDarkly Environment Key where the flag exists",
        "title": "environment_key",
        "type": "string",
        "blueprint": "launchDarklyEnvironment",
        "format": "entity"
      },
      "flag_state": {
        "title": "flag_state",
        "description": "Desired state of the feature flag (true for enabled, false for disabled)",
        "icon": "Launchdarkly",
        "type": "boolean",
        "default": true
      }
    },
    "required": [
      "feature_flag_key",
      "project_key"
    ],
    "order": [
      "feature_flag_key",
      "project_key",
      "environment_key",
      "flag_state"
    ]
  },
  "invocationMethod": {
    "type": "GITHUB",
    "org": "mk-armah",
    "repo": "jira-actions",
    "workflow": "toggle-a-feature-flag.yaml",
    "omitUserInputs": false,
    "omitPayload": false,
    "reportWorkflowStatus": true
  },
  "trigger": "DAY-2",
  "description": "Toggle a Feature Flag in launchdarkly",
  "requiredApproval": false
}
```
>**Note** Replace the invocation method with your own repository details.

5. Create a workflow file under .github/workflows/toggle_a_feature_flag.yaml with the following content:
```yml
name: Toggle LaunchDarkly Feature Flag

on:
  workflow_dispatch:
    inputs:
      feature_flag_key:
        description: 'LaunchDarkly Feature Flag Key'
        required: true
        type: string
      project_key:
        description: 'LaunchDarkly Project Key'
        required: true
        type: string
      environment_key:
        description: 'LaunchDarkly Environment Key where the flag exists'
        required: true
        type: string
      flag_state:
        description: 'Desired state of the feature flag (true for enabled, false for disabled)'
        required: true
        type: boolean
      port_payload:
        description: "Port's payload, including details for who triggered the action and general context"
        required: true
        type: string

jobs:
  toggle-feature-flag:
    runs-on: ubuntu-latest
    steps:
      - name: Inform toggling of Launchdarkly feature flag
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: PATCH_RUN
          runId: ${{fromJson(github.event.inputs.port_payload).context.runId}}
          logMessage: "Attempting to toggle feature flag '${{ github.event.inputs.feature_flag_key }}' in '${{ github.event.inputs.environment_key }}' environment to ${{ github.event.inputs.flag_state }}."

      - name: Toggle feature flag in Launchdarkly
        id: "toggle_feature_flag"
        uses: fjogeleit/http-request-action@v1
        with:
          url: 'https://app.launchdarkly.com/api/v2/flags/${{ github.event.inputs.project_key }}/${{ github.event.inputs.feature_flag_key }}'
          method: 'PATCH'
          customHeaders: '{"Authorization": "${{ secrets.LAUNCHDARKLY_ACCESS_TOKEN }}", "Content-Type": "application/json"}'
          data: >-
            [{
              "op": "replace",
              "path": "/environments/${{ github.event.inputs.environment_key }}/on",
              "value": ${{ github.event.inputs.flag_state }}
            }]

      - name: Convert creationDate to date-time format
        id: format_date
        run: |
          timestamp="${{ fromJson(steps.toggle_feature_flag.outputs.response).creationDate }}"
          epoch_seconds=$(($timestamp / 1000))
          formatted_date=$(date -u -d "@${epoch_seconds}" +"%Y-%m-%dT%H:%M:%SZ")
          echo "creationDate=${formatted_date}" >> $GITHUB_OUTPUT
          echo "creationDate=${formatted_date}"
          
      - name: Inform ingestion of Launchdarkly feature flag to Port
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: PATCH_RUN
          runId: ${{fromJson(github.event.inputs.port_payload).context.runId}}
          logMessage: "Attempting to upsert entity to Port"
          
      - name: Upsert Launchdarkly entity to Port
        uses: port-labs/port-github-action@v1
        with:
          identifier: "${{ fromJson(steps.toggle_feature_flag.outputs.response).key }}"
          title: "${{ fromJson(steps.toggle_feature_flag.outputs.response).description }}"
          blueprint: "launchDarklyFeatureFlag"
          properties: |-
            {
              "kind": "${{ fromJson(steps.toggle_feature_flag.outputs.response).kind }}",
              "description": "${{ fromJson(steps.toggle_feature_flag.outputs.response).description }}",
              "creationDate": "${{ steps.format_date.outputs.creationDate }}",
              "includeInSnippet": ${{ fromJson(steps.toggle_feature_flag.outputs.response).includeInSnippet }},
              "clientSideAvailability": ${{ toJson(fromJson(steps.toggle_feature_flag.outputs.response).clientSideAvailability) }},
              "temporary": ${{ fromJson(steps.toggle_feature_flag.outputs.response).temporary }},
              "tags": ${{ toJson(fromJson(steps.toggle_feature_flag.outputs.response).tags) }},
              "maintainer": ${{ toJson(fromJson(steps.toggle_feature_flag.outputs.response)._maintainer) }},
              "environments": ${{ toJson(fromJson(steps.toggle_feature_flag.outputs.response).environments) }},
              "variations": ${{ toJson(fromJson(steps.toggle_feature_flag.outputs.response).variations) }},
              "customProperties": ${{ toJson(fromJson(steps.toggle_feature_flag.outputs.response).customProperties) }},
              "archived": ${{ fromJson(steps.toggle_feature_flag.outputs.response).archived }},
              "projectKey": "${{ github.event.inputs.project_key }}"
            }
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: UPSERT
          runId: ${{ fromJson(inputs.port_payload).context.runId }}
          
      - name: Inform completion of Launchdarkly feature flag ingestion into Port
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: PATCH_RUN
          runId: ${{fromJson(github.event.inputs.port_payload).context.runId}}
          logMessage: "Feature flag '${{ github.event.inputs.feature_flag_key }}' in '${{ github.event.inputs.environment_key }}' environment set to ${{ github.event.inputs.flag_state }}."
```

Congrats 🎉 You've toggled your first launchdarkly feature flag from Port!