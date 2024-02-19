<img align="right" width="100" height="74" src="https://user-images.githubusercontent.com/8277210/183290025-d7b24277-dfb4-4ce1-bece-7fe0ecd5efd4.svg" />

# Create PagerDuty Incident Action

[![Slack](https://img.shields.io/badge/Slack-4A154B?style=for-the-badge&logo=slack&logoColor=white)](https://join.slack.com/t/devex-community/shared_invite/zt-1bmf5621e-GGfuJdMPK2D8UN58qL4E_g)

This GitHub action allows you to quickly create incidents in PagerDuty via Port Actions with ease.

## Inputs
| Name                 | Description                                                                                          | Required | Default            |
|----------------------|------------------------------------------------------------------------------------------------------|----------|--------------------|
| token                | The Pager Duty Token to use to authenticate with the API with permissions to create incidents      | true     | -                  |
| portClientId         | The Port Client ID to use to authenticate with the API                                           | true     | -                  |
| portClientSecret     | The Port Client Secret to use to authenticate with the API                                       | true     | -                  |
| blueprintIdentifier | The blueprint identifier to use to populate the Port relevant entity                              | false    | pagerdutyIncident |
| incidentTitle        | The title of the incident to create                                                                | true     | -                  |
| extraDetails         | Extra details about the incident to create                                                        | false    | -                  |
| service              | The service id to correlate the incident to                                                       | true     | -                  |
| urgency              | The urgency of the incident to create                                                              | false    | high               |
| actorEmail           | The email of the actor to create the incident with                                                | true     | -                  |
| portRunId            | Port run ID to came from triggering the action                                                    | true     | -                  |


## Quickstart - Create a pager duty incident from the service catalog

Follow these steps to get started with the Golang template

1. Create the following GitHub action secrets
* `PAGER_TOKEN` - a token with permission to create incidents [learn more](https://support.pagerduty.com/docs/generating-api-keys#section-generating-a-personal-rest-api-key)
* `PORT_CLIENT_ID` - Port Client ID [learn more](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token)
* `PORT_CLIENT_SECRET` - Port Client Secret [learn more](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token) 

2. Install the Ports GitHub app from [here](https://github.com/apps/getport-io/installations/new).
3. Install Port's pager duty integration [learn more][https://github.com/port-labs/Port-Ocean/tree/main/integrations/pagerduty] 
>**Note** This step is not required for this example, but it will create all the blueprint boilerplate for you, and also update the catalog in real time with the new incident created.
4. After you installed the integration, the blueprints `pagerdutyService` and `pagerdutyIncident` will appear, create the following action with the following JSON file on the `pagerdutyService` blueprint:

```json
[
  {
    "identifier": "trigger_incident",
    "title": "Trigger Incident",
    "icon": "pagerduty",
    "userInputs": {
      "properties": {
        "title": {
          "icon": "DefaultProperty",
          "title": "Title",
          "type": "string"
        },
        "extra_details": {
          "title": "Extra Details",
          "type": "string"
        },
        "urgency": {
          "icon": "DefaultProperty",
          "title": "Urgency",
          "type": "string",
          "default": "high",
          "enum": [
            "high",
            "low"
          ],
          "enumColors": {
            "high": "yellow",
            "low": "green"
          }
        },
        "from": {
          "title": "From",
          "icon": "TwoUsers",
          "type": "string",
          "format": "user",
          "default": {
            "jqQuery": ".user.email"
          }
        }
      },
      "required": [
        "title",
        "urgency",
        "from"
      ],
      "order": [
        "title",
        "urgency",
        "from",
        "extra_details"
      ]
    },
    "invocationMethod": {
      "type": "GITHUB",
      "omitPayload": false,
      "omitUserInputs": true,
      "reportWorkflowStatus": true,
      "org": "port-pagerduty-example",
      "repo": "test",
      "workflow": "create-an-incident.yaml"
    },
    "trigger": "DAY-2",
    "description": "Notify users and teams about incidents in the service",
    "requiredApproval": false
  }
]
```
>**Note** Replace the invocation method with your own repository details.

5. Create a workflow file under .github/workflows/create-an-incident.yaml with the following content:
```yml
on:
  workflow_dispatch:
    inputs:
      port_payload:
        required: true
        description: "Port's payload, including details for who triggered the action and general context (blueprint, run id, etc...)"
        type: string
    secrets: 
      PAGER_TOKEN: 
        required: true
      PORT_CLIENT_ID:
        required: true
      PORT_CLIENT_SECRET:
        required: true
jobs: 
  trigger:
    runs-on: ubuntu-latest
    steps:
      - uses: port-labs/pagerduty-incident-gha@v1
        with:
          portClientId: ${{ secrets.PORT_CLIENT_ID }}
          portClientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          token: ${{ secrets.PAGER_TOKEN }}
          portRunId: ${{ fromJson(inputs.port_payload).context.runId }}
          incidentTitle: ${{ fromJson(inputs.port_payload).payload.properties.title }}
          extraDetails: ${{ fromJson(inputs.port_payload).payload.properties.extra_details }}
          urgency: ${{ fromJson(inputs.port_payload).payload.properties.urgency }}
          service: ${{ fromJson(inputs.port_payload).context.entity }}
          blueprintIdentifier: 'pagerdutyIncident'
```

6. Trigger the action from Port's [Self Serve](https://app.getport.io/self-serve)
![image](https://github.com/port-labs/pagerduty-incident-gha/assets/51213812/2cda51d4-4594-4f47-9ef4-3b2419b0351a)

7. Done! wait for the incident to be created in Pager Duty
![image](https://github.com/port-labs/pagerduty-incident-gha/assets/51213812/74cb8aad-e426-4ab1-b388-74b80a5d2eb1)

Congrats 🎉 You've created your first incident in PagerDuty from Port!


# Create PagerDuty Service Action

This GitHub action allows you to quickly create a service in PagerDuty via Port with ease.

## Inputs
| Name                 | Description                                                                                          | Required | Default            |
|----------------------|------------------------------------------------------------------------------------------------------|----------|--------------------|
| name         | A unique identifier or title used to reference and distinguish the service in PagerDuty     | true    | -                  |
| description              | A brief summary or explanation detailing the purpose or scope of the service in PagerDuty.                               | false     | -                  |
| escalation_policy              | A set of rules in PagerDuty that determines the sequence of notifications to team members in response to an incident, ensuring timely attention and action                                                              | true    | -               |



## Quickstart - Create a `PagerDuty service` from the service catalog

1. Create the following GitHub action secrets
* `PAGERDUTY_API_KEY` - PagerDuty API Key [learn more](https://support.pagerduty.com/docs/api-access-keys#section-generate-a-user-token-rest-api-key:~:text=the%20browser%20alert.-,Generate%20a%20User%20Token%20REST%20API%20Key,-%F0%9F%9A%A7)
* `PORT_CLIENT_ID` - Port Client ID [learn more](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token)
* `PORT_CLIENT_SECRET` - Port Client Secret [learn more](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token) 

2. Install the Ports GitHub app from [here](https://github.com/apps/getport-io/installations/new).
3. Install Port's pager duty integration [learn more][https://github.com/port-labs/Port-Ocean/tree/main/integrations/pagerduty]
>**Note** This step is not required for this example, but it will create all the blueprint boilerplate for you, and also update the catalog in real time with the new incident created.
4. After you installed the integration, the blueprints `pagerdutyService` and `pagerdutyIncident` will appear, create the following action with the following JSON file on the `pagerdutyService` blueprint:

```json

[
  {
    "identifier": "create_pagerduty_service",
    "title": "create_pagerduty_service",
    "icon": "pagerduty",
    "userInputs": {
      "properties": {
        "name": {
          "title": "name",
          "description": "Name of the PagerDuty Service",
          "icon": "pagerduty",
          "type": "string"
        },
        "description": {
          "title": "description",
          "description": "Description of the PagerDuty Service",
          "icon": "pagerduty",
          "type": "string"
        },
        "escalation_policy": {
          "title": "escalation_policy",
          "icon": "pagerduty",
          "type": "object"
        }
      },
      "required": [
        "name",
        "escalation_policy"
      ],
      "order": [
        "name",
        "description",
        "escalation_policy"
      ]
    },
    "invocationMethod": {
      "type": "GITHUB",
      "org": "my-org",
      "repo": "my-repo",
      "workflow": "create-a-service.yaml",
      "omitUserInputs": false,
      "omitPayload": false,
      "reportWorkflowStatus": true
    },
    "trigger": "CREATE",
    "description": "Create PagerDuty Service",
    "requiredApproval": false
  }
]

```
>**Note** Replace the invocation method with your own repository details.

5. Create a workflow file under .github/workflows/create-a-service.yaml with the following content:

```yml
name: Create PagerDuty Service
on:
  workflow_dispatch:
    inputs:
      name:
        description: 'Name of the PagerDuty Service'
        required: true
        type: string
      description:
        description: 'Description of the PagerDuty Service'
        required: false
        type: string
      escalation_policy:
        description: 'Escalation Policy for the service'
        required: true
        type: string

jobs:
  create-pagerduty-service:
    runs-on: ubuntu-latest
    steps:
      - name: Create Service in PagerDuty
        id : create_service_request
        uses: fjogeleit/http-request-action@v1
        with:
          url: 'https://api.pagerduty.com/services'
          method: 'POST'
          customHeaders: '{"Content-Type": "application/json", "Accept": "application/vnd.pagerduty+json;version=2", "Authorization": "Token token=${{ secrets.PAGERDUTY_API_KEY }}"}'
          data: >-
            {
              "service": {
                "name": "${{ github.event.inputs.name }}",
                "description": "${{ github.event.inputs.description }}",
                "escalation_policy": ${{ github.event.inputs.escalation_policy }}
              }
            } 

      - name: Create a log message
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: PATCH_RUN
          runId: ${{fromJson(inputs.port_payload).context.runId}}
          logMessage: |
             PagerDuty service created! ✅
             Requesting for oncalls
    
      - name: Request for oncalls for escalation_policy
        id: fetch_oncalls
        uses: fjogeleit/http-request-action@v1
        with:
          url: 'https://api.pagerduty.com/oncalls?include[]=users&escalation_policy_ids[]=${{ fromJson(github.event.inputs.escalation_policy).id }}'
          method: 'GET'
          customHeaders: '{"Content-Type": "application/json", "Accept": "application/json", "Authorization": "Token token=${{ secrets.PAGERDUTY_API_KEY }}"}'

      - name: Create a log message
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: PATCH_RUN
          runId: ${{fromJson(inputs.port_payload).context.runId}}
          logMessage: |
              Upserting Created PagerDuty Entity

      - name: UPSERT PagerDuty Entity
        uses: port-labs/port-github-action@v1
        with:
          identifier: "${{ fromJson(steps.create_service_request.outputs.response).service.id }}" 
          title: "${{ fromJson(steps.create_service_request.outputs.response).service.summary }}" 
          team: "[]"
          icon: pagerduty
          blueprint: pagerdutyService
          properties: |-
            {
              "status": "${{ fromJson(steps.create_service_request.outputs.response).service.status }}",
              "url": "${{ fromJson(steps.create_service_request.outputs.response).service.html_url }}",
              "oncall": "${{ fromJson(steps.fetch_oncalls).oncalls }}"
            }
          relations: "{}"
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: UPSERT
          runId: ${{fromJson(inputs.port_payload).context.runId}}

      - name: Create a log message
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: PATCH_RUN
          runId: ${{fromJson(inputs.port_payload).context.runId}}
          logMessage: |
              Upsert was successful ✅

```

6. Trigger the action from Port's [Self Serve](https://app.getport.io/self-serve)

Congrats 🎉 You've created your first `service` in PagerDuty from Port!

# Trigger A PagerDuty Incident Action

This GitHub action allows you to quickly trigger incidents in PagerDuty via Port Actions with ease.

## Inputs
| Name                 | Description                                                                                          | Required | Default            |
|----------------------|------------------------------------------------------------------------------------------------------|----------|--------------------|
| payload         | Fields adding additional context about the location, impact, and character of the PagerDuty event.     | false    | -                  |
| routing_key              | The GUID of one of your PagerDuty Events API V2 integrations                                | true     | -                  |
| event_action              | The type of event. Can be trigger, acknowledge or resolve                                                              | false    | -               |
| dedup_key              | Identifies the alert to trigger, acknowledge, or resolve                                                             | true unless the event_type is trigger    | -               | 


## Quickstart - Trigger a PagerDuty incident from the service catalog

Follow these steps to get started with the Golang template

1. Create the following GitHub action secrets
* `PORT_CLIENT_ID` - Port Client ID [learn more](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token)
* `PORT_CLIENT_SECRET` - Port Client Secret [learn more](https://docs.getport.io/build-your-software-catalog/sync-data-to-catalog/api/#get-api-token) 

2. Install the Ports GitHub app from [here](https://github.com/apps/getport-io/installations/new).
3. Install Port's pager duty integration [learn more][https://github.com/port-labs/Port-Ocean/tree/main/integrations/pagerduty]
>**Note** This step is not required for this example, but it will create all the blueprint boilerplate for you, and also update the catalog in real time with the new incident created.
4. After you installed the integration, the blueprints `pagerdutyService` and `pagerdutyIncident` will appear, create the following action with the following JSON file on the `pagerdutyService` blueprint:

```json

[
  {
    "identifier": "trigger_an_incident",
    "title": "trigger_an_incident",
    "icon": "pagerduty",
    "userInputs": {
      "properties": {
        "payload": {
          "title": "payload",
          "icon": "pagerduty",
          "type": "object"
        },
        "event_action": {
          "icon": "pagerduty",
          "title": "event_action",
          "type": "string"
        },
        "routing_key": {
          "icon": "pagerduty",
          "title": "routing_key",
          "type": "string"
        }
      },
      "required": [
        "routing_key",
        "event_action",
        "payload"
      ],
      "order": [
        "routing_key",
        "event_action",
        "payload"
      ]
    },
    "invocationMethod": {
      "type": "GITHUB",
      "org": "my-org-name",
      "repo": "my-repo",
      "workflow": "trigger-pagerduty-incident.yaml",
      "omitUserInputs": false,
      "omitPayload": false,
      "reportWorkflowStatus": true
    },
    "trigger": "DAY-2",
    "description": "Trigger a pagerduty incident using self service",
    "requiredApproval": false
  }
]

```
>**Note** Replace the invocation method with your own repository details.

5. Create a workflow file under .github/workflows/create-an-incident.yaml with the following content:

```yml
name: Trigger an Incident In PagerDuty
on:
  workflow_dispatch:
    inputs:
      payload:
        type: string
      event_action:
        type: string
      routing_key:
        type: string
      port_payload:
        required: true
        description: Port's payload, including details for who triggered the action and
          general context (blueprint, run id, etc...)
        type: string
jobs:
  create-entity-in-port-and-update-run:
    runs-on: ubuntu-latest
    steps: 

      - name: Inform starting of PagerDuty trigger 
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          operation: PATCH_RUN
          runId: ${{ fromJson(inputs.port_payload).context.runId }}
          logMessage: |
              About to trigger PagerDuty incident.. ⛴️
      - name: Send Event to PagerDuty
        id: trigger
        uses: fjogeleit/http-request-action@v1
        with:
          url: 'https://events.pagerduty.com/v2/enqueue'
          method: 'POST'
          customHeaders: '{"Content-Type": "application/json", "Accept": "application/json"}'
          data: >-
            {
              "payload": ${{ github.event.inputs.payload }},
              "event_action": "${{ github.event.inputs.event_action }}",
              "routing_key": "${{ github.event.inputs.routing_key }}"
            }  
      - name: Create a log message
        id: log-response
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: PATCH_RUN
          runId: ${{fromJson(inputs.port_payload).context.runId}}
          logMessage: |
             PagerDuty incident triggered! ✅
             "The incident id is: ${{ steps.trigger.outputs}}"
```

6. Trigger the action from Port's [Self Serve](https://app.getport.io/self-serve)

Congrats 🎉 You've created your first incident in PagerDuty from Port!


# Change Incident Owner In Pagerduty

This GitHub action allows you to quickly change incident owner in PagerDuty via Port Actions with ease.

## Inputs
| Name                 | Description                                                                                          | Required | Default            |
|----------------------|------------------------------------------------------------------------------------------------------|----------|--------------------|
| incident_id         | ID of the PagerDuty Incident     | true    | -                  |
| new_owner_user_id              | PagerDuty User ID of the new owner                                | true     | -                  |
| from              | The email address of a valid user associated with the account making the request.                                                              | true    | -               |

## Quickstart - Change Incident Owner In Pagerduty

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
  "identifier": "change_incident_owner",
  "title": "Change Incident Owner",
  "icon": "pagerduty",
  "userInputs": {
    "properties": {
      "incident_id": {
        "icon": "pagerduty",
        "title": "Incident Id",
        "description": "ID of the PagerDuty Incident",
        "type": "string",
        "blueprint": "pagerdutyIncident",
        "format": "entity"
      },
      "new_owner_user_id": {
        "title": "New Owner User ID",
        "description": "PagerDuty User ID of the new owner",
        "icon": "pagerduty",
        "type": "string"
      },
      "from": {
        "icon": "pagerduty",
        "title": "From",
        "description": "The email address of a valid user associated with the account making the request.",
        "type": "string"
      }
    },
    "required": [
      "new_owner_user_id",
      "incident_id",
      "from"
    ],
    "order": [
      "incident_id",
      "new_owner_user_id",
      "from"
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
  "description": "Change Incident Owner in pagerduty",
  "requiredApproval": false
}
```
>**Note** Replace the invocation method with your own repository details.

5. Create a workflow file under .github/workflows/change-incident-owner.yaml with the following content:

```yml
name: Change PagerDuty Incident Owner

on:
  workflow_dispatch:
    inputs:
      incident_id:
        description: ID of the PagerDuty Incident
        required: true
        type: string
      new_owner_user_id:
        description: PagerDuty User ID of the new owner
        required: true
        type: string
      from:
        description: The email address of a valid user associated with the account making the request.
        required: true
        type: string
      port_payload:
        required: true
        description: >-
          Port's payload, including details for who triggered the action and
          general context (blueprint, run id, etc...)

jobs:
  change-incident-owner:
    runs-on: ubuntu-latest
    steps:
      
      - name: Inform execution of request to change incident owner
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: PATCH_RUN
          runId: ${{fromJson(github.event.inputs.port_payload).context.runId}}
          logMessage: "About to make a request to pagerduty..."

      - name: Change Incident Owner in PagerDuty
        id: change_owner
        uses: fjogeleit/http-request-action@v1
        with:
          url: 'https://api.pagerduty.com/incidents/${{ github.event.inputs.incident_id }}'
          method: 'PUT'
          customHeaders: '{"Content-Type": "application/json", "Accept": "application/vnd.pagerduty+json;version=2", "Authorization": "Token token=${{ secrets.PAGERDUTY_API_KEY }}", "From": "${{ github.event.inputs.from }}"}'
          data: >-
            {
              "incident": {
                "type": "incident_reference",
                "assignments": [
                  {
                    "assignee": {
                      "id": "${{ github.event.inputs.new_owner_user_id }}",
                      "type": "user_reference"
                    }
                  }
                ]
              }
            }

      - name: Inform ingestion of pagerduty feature flag to Port
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: PATCH_RUN
          runId: ${{fromJson(github.event.inputs.port_payload).context.runId}}
          logMessage: "Reporting the updated incident back to port ..."

      - name: Upsert pagerduty entity to Port 
        uses: port-labs/port-github-action@v1
        with:
          identifier: "${{ fromJson(steps.change_owner.outputs.response).incident.id }}"
          title: "${{ fromJson(steps.change_owner.outputs.response).incident.title }}"
          blueprint: "pagerdutyIncident"
          properties: |-
            {
              "status": "${{ fromJson(steps.change_owner.outputs.response).incident.status }}",
              "url": "${{ fromJson(steps.change_owner.outputs.response).incident.self }}",
              "urgency": "${{ fromJson(steps.change_owner.outputs.response).incident.urgency }}",
              "responder": "${{ fromJson(steps.change_owner.outputs.response).incident.assignments[0].assignee.summary}}",
              "escalation_policy": "${{ fromJson(steps.change_owner.outputs.response).incident.escalation_policy.summary }}",
              "created_at": "${{ fromJson(steps.change_owner.outputs.response).incident.created_at }}",
              "updated_at": "${{ fromJson(steps.change_owner.outputs.response).incident.updated_at }}"
            }
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: UPSERT
          runId: ${{ fromJson(inputs.port_payload).context.runId }}

      - name: Inform completion of pagerduty feature flag ingestion into Port
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: PATCH_RUN
          runId: ${{fromJson(github.event.inputs.port_payload).context.runId}}
          logMessage: "Entity upserting was successful ✅"
```


# Acknowlege Incident In Pagerduty

This GitHub action allows you to quickly acknowledge incident in PagerDuty via Port Actions with ease.

## Inputs
| Name                 | Description                                                                                          | Required | Default            |
|----------------------|------------------------------------------------------------------------------------------------------|----------|--------------------
| from              | The email address of a valid user associated with the account making the request.                                                              | true    | -               |

## Quickstart - Acknowlege Incident In Pagerduty

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
  "identifier": "acknowlege_incident",
  "title": "Acknowlege Incident",
  "icon": "pagerduty",
  "userInputs": {
    "properties": {
      "from": {
        "icon": "pagerduty",
        "title": "From",
        "description": "User Email",
        "type": "string"
      }
    },
    "required": [],
    "order": [
      "from"
    ]
  },
  "invocationMethod": {
    "type": "GITHUB",
    "org": "mk-armah",
    "repo": "jira-actions",
    "workflow": "acknowledge-incidents.yaml",
    "omitUserInputs": false,
    "omitPayload": false,
    "reportWorkflowStatus": true
  },
  "trigger": "DAY-2",
  "description": "Acknowledge incident in pagerduty",
  "requiredApproval": false
}
```
>**Note** Replace the invocation method with your own repository details.

5. Create a workflow file under .github/workflows/acknowlege-incident.yaml with the following content:

```yml
name: Acknowledge Incident In PagerDuty
on:
  workflow_dispatch:
    inputs:
      from:
        description: The email address of a valid user associated with the account making the request.
        required: true
        type: string
      port_payload:
        required: true
        description: >-
          Port's payload, including details for who triggered the action and
          general context (blueprint, run id, etc...)

jobs:
  acknowledge_incident:
    runs-on: ubuntu-latest
    steps:
      
      - name: Log Executing Request to Acknowledge Incident
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: PATCH_RUN
          runId: ${{fromJson(github.event.inputs.port_payload).context.runId}}
          logMessage: "About to make a request to pagerduty..."

      - name: Request to Acknowledge Incident
        id: acknowledge_incident
        uses: fjogeleit/http-request-action@v1
        with:
          url: 'https://api.pagerduty.com/incidents'
          method: 'PUT'
          customHeaders: '{"Content-Type": "application/json", "Accept": "application/vnd.pagerduty+json;version=2", "Authorization": "Token token=${{ secrets.PAGERDUTY_API_KEY }}", "From": "${{ github.event.inputs.from }}"}'
          data: >-
              {
                "incidents": [
                  {
                    "id": "${{ fromJson(inputs.port_payload).context.entity }}",
                    "type": "incident_reference",
                    "status": "acknowledged"
                  }
                ]
              }

      - name: Log Before Processing Incident Response
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: PATCH_RUN
          runId: ${{fromJson(github.event.inputs.port_payload).context.runId}}
          logMessage: "Getting incident object from response received ..."

      - name: Log Before Upserting Entity
        uses: port-labs/port-github-action@v1
        with:
          clientId: ${{ secrets.PORT_CLIENT_ID }}
          clientSecret: ${{ secrets.PORT_CLIENT_SECRET }}
          baseUrl: https://api.getport.io
          operation: PATCH_RUN
          runId: ${{fromJson(github.event.inputs.port_payload).context.runId}}
          logMessage: "Reporting the updated incident back to port ..."

      - name: UPSERT Entity
        uses: port-labs/port-github-action@v1
        with:
          identifier: "${{ fromJson(steps.acknowledge_incident.outputs.response).incidents[0].id }}"
          title: "${{ fromJson(steps.acknowledge_incident.outputs.response).incidents[0].title }}"
          blueprint: ${{ fromJson(inputs.port_payload).context.blueprint }}
          properties: |-
            {
              "status": "${{ fromJson(steps.acknowledge_incident.outputs.response).incidents[0].status }}",
              "url": "${{ fromJson(steps.acknowledge_incident.outputs.response).incidents[0].self }}",
              "urgency": "${{ fromJson(steps.acknowledge_incident.outputs.response).incidents[0].urgency }}",
              "responder": "${{ fromJson(steps.acknowledge_incident.outputs.response).incidents[0].assignments[0].assignee.summary}}",
              "escalation_policy": "${{ fromJson(steps.acknowledge_incident.outputs.response).incidents[0].escalation_policy.summary }}",
              "created_at": "${{ fromJson(steps.acknowledge_incident.outputs.response).incidents[0].created_at }}",
              "updated_at": "${{ fromJson(steps.acknowledge_incident.outputs.response).incidents[0].updated_at }}"
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

Congrats 🎉 You've successfully acknowleged an incident in PagerDuty from Port!