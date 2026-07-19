# LiteLLM Integration

## Overview

[LiteLLM](https://www.litellm.ai/) is a **unified LLM gateway and proxy** that provides a unified interface for accessing multiple large language models (LLMs) from different providers. It offers comprehensive audit logging, rate limiting, and cost tracking across hybrid LLM deployments — combining authentication, authorization, and detailed audit trails into a unified platform for **critical LLM infrastructure monitoring and compliance**.

The LiteLLM integration for Elastic collects audit logs using the **LiteLLM Audit API** or through **AWS S3/SQS** cloud storage, and visualizes them in Kibana.

### Compatibility

The LiteLLM integration is compatible with **LiteLLM version 1.91.0 and above**.

### How it works

This integration supports two collection methods:

- **Direct API polling** via CEL input, which periodically queries the LiteLLM Audit API using API key authentication with cursor-based pagination.
- **Cloud storage** via AWS S3/SQS, for organizations that export audit logs from LiteLLM to an AWS S3 bucket.

## What data does this integration collect?

The LiteLLM integration collects the following types of data:

| Data stream | Description | Source |
|---|---|---|
| `audit` | Audit log records retrieved from the LiteLLM `/audit` API or S3 buckets, including API usage, authentication events, configuration changes, and user actions across LiteLLM_VerificationToken, LiteLLM_UserTable, and LiteLLM_TeamTable entities. | `/audit` API or AWS S3 |

### Supported use cases

Integrating LiteLLM with Elastic provides centralized visibility into LLM API usage, authentication events, and configuration changes across your LiteLLM deployment, enabling efficient monitoring, investigation, and compliance reporting within Kibana dashboards.

## What do I need to use this integration?

### From LiteLLM (API collection)

To collect data via the Audit API, you need an **API key** with permission to access the `/audit` endpoint.

1. Sign in to your LiteLLM deployment admin panel.
2. Navigate to **API Keys** or **Credentials**.
3. Create or select an API key with permission to read audit logs from the `/audit` endpoint.

For more information on configuring API keys in LiteLLM, refer to the [LiteLLM API documentation](https://docs.litellm.ai/docs/proxy/virtual_keys).

### From LiteLLM (AWS S3 collection)

To collect data using AWS S3, configure LiteLLM to export logs to an S3 bucket, then point Elastic at that bucket.

#### Step 1: Set up AWS infrastructure:

1. Create an **S3 bucket** and note the bucket name and region.
2. Create an **IAM access policy** with these permissions:
   - Read: `GetObject`, `ListBucket`
   - SQS (optional): `ReceiveMessage`, `DeleteMessage`
3. Create an **IAM user** with programmatic access, attach the policy, and save the **Access Key ID** and **Secret Access Key**.

#### Step 2: Configure S3 export in LiteLLM:

1. Sign in to your LiteLLM deployment admin panel.
2. Navigate to **Configuration** > **S3 Export** or **Cloud Storage Settings**.
3. Select **Enable S3 Export** and choose your bucket.
4. Enter the **Access Key ID**, **Secret Access Key**, **Bucket** name, and **Region**.
5. Set the data format to **ECS - Elastic Common Schema**.
6. Click **Validate Settings**, then **Save Settings**.

> **Note:** Audit logs are exported to S3 in JSON format at regular intervals.

For more information on configuring S3 export in LiteLLM, refer to the [LiteLLM S3 export documentation](https://docs.litellm.ai/docs/proxy/multiple_admins).

## How do I deploy this integration?

This integration supports both Elastic Agentless-based and Agent-based installations.

### Agentless-based installation

Agentless integrations allow you to collect data without having to manage Elastic Agent in your cloud. They make manual agent deployment unnecessary, so you can focus on your data instead of the agent that collects it. For more information, refer to [Agentless integrations](https://www.elastic.co/guide/en/serverless/current/security-agentless-integrations.html) and the [Agentless integrations FAQ](https://www.elastic.co/guide/en/serverless/current/agentless-integration-troubleshooting.html).

Agentless deployments are only supported in Elastic Serverless and Elastic Cloud environments. This functionality is in beta and is subject to change. Beta features are not subject to the support SLA of official GA features.

### Agent-based installation

Elastic Agent must be installed. For more details, check the Elastic Agent [installation instructions](docs-content://reference/fleet/install-elastic-agents.md). You can install only one Elastic Agent per host.

### Configure

1. In the top search bar in Kibana, search for **Integrations**.
2. In the search bar, type **LiteLLM**.
3. Select the **LiteLLM** integration from the search results.
4. Select **Add LiteLLM** to add the integration.
5. Enable and configure only the collection methods which you will use.

    * To **Collect logs using LiteLLM API (CEL)**:

        - Set the **URL** to the base URL of your LiteLLM instance (e.g., `https://litellm.example.com`).
        - Set the **API Key** obtained from your LiteLLM admin panel.
        - Optionally adjust **Initial Interval**, **Interval**, **Batch Size**, and **HTTP Client Timeout**.

    * To **Collect logs using AWS S3**:

        - Set the **Bucket ARN** of the S3 bucket configured in LiteLLM S3 Export Settings.
        - Set **AWS Access Key ID** and **Secret Access Key** for an IAM user with read access to the bucket.
        - Optionally configure **Queue URL** (SQS) if using event-driven notifications instead of bucket polling.

6. Select **Save and continue** to save the integration.

## Troubleshooting

* **No data collected (CEL)**: Verify that the LiteLLM `/audit` API is enabled and reachable from the Elastic Agent host. Confirm that the configured URL and API key are correct.
* **No data collected (S3)**: Verify that the S3 bucket exists and is accessible from the Elastic Agent host. Confirm that AWS credentials have S3:GetObject and S3:ListBucket permissions.
* **Authentication failures (CEL)**: Ensure the API key has permission to read audit logs from the `/audit` endpoint. Verify the key has not been revoked or expired.
* **Authentication failures (S3)**: Verify AWS credentials are correct and have appropriate S3 permissions. Check that cross-account role assumption is configured correctly if using a role in a different account.
* **SSL certificate errors**: If LiteLLM or AWS endpoints use self-signed certificates, extract the certificate and configure it under the SSL settings of the integration, or add it to the Elastic Agent's trusted certificate store.
* **Pagination or rate limiting (CEL)**: If data collection is incomplete, verify batch size and interval settings. LiteLLM may enforce rate limits on the `/audit` endpoint.
* **S3 access denied**: Verify IAM permissions include S3:GetObject, S3:ListBucket, and optionally SQS:ReceiveMessage, SQS:DeleteMessage (for SQS-based collection).
* **SQS visibility timeout**: If messages are being reprocessed, increase the visibility timeout to allow sufficient processing time.

For help with Elastic ingest tools, check [Common problems](https://www.elastic.co/docs/troubleshoot/ingest/fleet/common-problems).

### Validation

#### Dashboard populated

1. In the top search bar in Kibana, search for **Dashboards**.
2. In the search bar, type **LiteLLM**, and verify the dashboard information is populated.

## Scaling

For more information on architectures that can be used for scaling this integration, check the [Ingest Architectures](https://www.elastic.co/docs/manage-data/ingest/ingest-reference-architectures) documentation.

## Reference

### Vendor documentation links

- [LiteLLM Proxy Server Documentation](https://docs.litellm.ai/docs/proxy/quick_start)
- [LiteLLM API Authentication](https://docs.litellm.ai/docs/proxy/virtual_keys)
- [LiteLLM Audit Logs API](https://docs.litellm.ai/docs/proxy/audit_logs)
- [LiteLLM Rate Limiting and Budgets](https://docs.litellm.ai/docs/proxy/user_management)

### Audit

The `audit` data stream provides audit log records collected from LiteLLM via API or S3.

#### Audit fields

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Date and time when the event occurred. | date |
| aws.s3.bucket.arn | The AWS S3 bucket ARN. | keyword |
| aws.s3.bucket.name | The AWS S3 bucket name. | keyword |
| aws.s3.object.key | The AWS S3 Object key. | keyword |
| data_stream.dataset | Dataset name associated with the data stream. | constant_keyword |
| data_stream.namespace | Namespace used to group related data streams. | constant_keyword |
| data_stream.type | Type of data stream, such as logs or metrics. | constant_keyword |
| event.dataset | Identifies the LiteLLM audit log dataset. | constant_keyword |
| event.module | Module that generated the event. | constant_keyword |
| input.type | Type of filebeat input. | keyword |
| lite_llm.audit.before_value.access_group_ids | List of access group IDs. | keyword |
| lite_llm.audit.before_value.aliases | Aliases mapping. | flattened |
| lite_llm.audit.before_value.allowed_cache_controls | List of allowed cache controls. | keyword |
| lite_llm.audit.before_value.allowed_routes | List of allowed routes (VerificationToken only). | keyword |
| lite_llm.audit.before_value.auto_rotate | Auto rotation flag (VerificationToken only). | boolean |
| lite_llm.audit.before_value.config | Configuration object. | flattened |
| lite_llm.audit.before_value.created_at | Creation timestamp. | date |
| lite_llm.audit.before_value.created_by | Created by user. | keyword |
| lite_llm.audit.before_value.key_alias | Key alias (VerificationToken only). | keyword |
| lite_llm.audit.before_value.key_name | Key name (VerificationToken only). | keyword |
| lite_llm.audit.before_value.max_budget | Maximum budget for the team. | long |
| lite_llm.audit.before_value.metadata | Custom metadata. | flattened |
| lite_llm.audit.before_value.model_max_budget | Per-model max budget. | flattened |
| lite_llm.audit.before_value.model_spend | Per-model spend tracking. | flattened |
| lite_llm.audit.before_value.models | List of models. | keyword |
| lite_llm.audit.before_value.permissions | Permissions mapping (VerificationToken only). | flattened |
| lite_llm.audit.before_value.policies | List of policies. | keyword |
| lite_llm.audit.before_value.rotation_count | Number of rotations (VerificationToken only). | long |
| lite_llm.audit.before_value.router_settings | Router settings (VerificationToken only). | flattened |
| lite_llm.audit.before_value.soft_budget_cooldown | Soft budget cooldown flag (VerificationToken only). | boolean |
| lite_llm.audit.before_value.spend | Spend value. | double |
| lite_llm.audit.before_value.teams | List of teams (UserTable only). | keyword |
| lite_llm.audit.before_value.token | Token identifier (VerificationToken only). | keyword |
| lite_llm.audit.before_value.updated_at | Last update timestamp. | date |
| lite_llm.audit.before_value.updated_by | Updated by user. | keyword |
| lite_llm.audit.before_value.user_id | User ID (UserTable only). | keyword |
| lite_llm.audit.before_value.user_role | User role (UserTable only). | keyword |
| lite_llm.audit.changed_by_api_key | API key that was used to perform the action. | keyword |
| lite_llm.audit.object_id | ID of the affected entity (team, user, key, or model UUID). No single ECS entity fits across all `table_name` values, so this is kept as a custom field rather than mapped to a `\*.id` field. | keyword |
| lite_llm.audit.table_name | Table where the change occurred (LiteLLM_UserTable or LiteLLM_VerificationToken). | keyword |
| lite_llm.audit.updated_values.access_group_ids | List of access group IDs. | keyword |
| lite_llm.audit.updated_values.aliases | Aliases mapping. | flattened |
| lite_llm.audit.updated_values.allowed_cache_controls | List of allowed cache controls. | keyword |
| lite_llm.audit.updated_values.allowed_routes | List of allowed routes (VerificationToken only). | keyword |
| lite_llm.audit.updated_values.auto_rotate | Auto rotation flag (VerificationToken only). | boolean |
| lite_llm.audit.updated_values.config | Configuration object. | flattened |
| lite_llm.audit.updated_values.created_at | Creation timestamp. | date |
| lite_llm.audit.updated_values.created_by | Created by user (set when action is created). | keyword |
| lite_llm.audit.updated_values.key | API key value (VerificationToken only). | keyword |
| lite_llm.audit.updated_values.key_alias | Key alias (VerificationToken only). | keyword |
| lite_llm.audit.updated_values.key_name | Key name (VerificationToken only). | keyword |
| lite_llm.audit.updated_values.max_budget | Maximum budget for the team. | long |
| lite_llm.audit.updated_values.metadata | Custom metadata. | flattened |
| lite_llm.audit.updated_values.model_max_budget | Per-model max budget. | flattened |
| lite_llm.audit.updated_values.model_spend | Per-model spend tracking. | flattened |
| lite_llm.audit.updated_values.models | List of models. | keyword |
| lite_llm.audit.updated_values.object_permission.agent_access_groups | List of agent access group IDs. | keyword |
| lite_llm.audit.updated_values.object_permission.agents | List of agent IDs. | keyword |
| lite_llm.audit.updated_values.object_permission.mcp_access_groups | List of MCP access group IDs. | keyword |
| lite_llm.audit.updated_values.object_permission.mcp_servers | List of MCP server IDs. | keyword |
| lite_llm.audit.updated_values.object_permission.mcp_toolsets | List of MCP toolset IDs. | keyword |
| lite_llm.audit.updated_values.permissions | Permissions mapping (VerificationToken only). | flattened |
| lite_llm.audit.updated_values.policies | List of policies. | keyword |
| lite_llm.audit.updated_values.rotation_count | Number of rotations (VerificationToken only). | long |
| lite_llm.audit.updated_values.router_settings.model_group_alias | Model group alias mapping. | flattened |
| lite_llm.audit.updated_values.soft_budget_cooldown | Soft budget cooldown flag (VerificationToken only). | boolean |
| lite_llm.audit.updated_values.spend | Spend value. | double |
| lite_llm.audit.updated_values.team_id | ID of the team (TeamTable only). | keyword |
| lite_llm.audit.updated_values.teams | List of teams (UserTable only). | keyword |
| lite_llm.audit.updated_values.token | Token identifier (VerificationToken only). | keyword |
| lite_llm.audit.updated_values.token_id | Token ID (VerificationToken only). | keyword |
| lite_llm.audit.updated_values.updated_at | Last update timestamp. | date |
| lite_llm.audit.updated_values.updated_by | Updated by user (set when action is created). | keyword |
| lite_llm.audit.updated_values.user_id | User ID (UserTable only). | keyword |
| lite_llm.audit.updated_values.user_role | User role (UserTable only). | keyword |
| log.offset | Log offset. | long |
| observer.product | Product name of the observer that generated the event. | constant_keyword |
| observer.vendor | Vendor name of the observer that generated the event. | constant_keyword |


### Example event

#### Audit

An example event for `audit` looks as following:

```json
{
    "@timestamp": "2026-07-03T11:30:59.385Z",
    "agent": {
        "ephemeral_id": "b7a16d12-e599-4d3c-be00-1f3e4915426a",
        "id": "2cf37096-07d9-4398-802e-62d4968b52cc",
        "name": "elastic-agent-76806",
        "type": "filebeat",
        "version": "8.18.0"
    },
    "aws": {
        "s3": {
            "bucket": {
                "arn": "arn:aws:s3:::elastic-package-lite-llm-audit-bucket-49613",
                "name": "elastic-package-lite-llm-audit-bucket-49613"
            },
            "object": {
                "key": "audit.log"
            }
        }
    },
    "cloud": {
        "region": "us-east-2"
    },
    "data_stream": {
        "dataset": "lite_llm.audit",
        "namespace": "21645",
        "type": "logs"
    },
    "ecs": {
        "version": "9.4.0"
    },
    "elastic_agent": {
        "id": "2cf37096-07d9-4398-802e-62d4968b52cc",
        "snapshot": false,
        "version": "8.18.0"
    },
    "event": {
        "action": "updated",
        "agent_id_status": "verified",
        "category": [
            "configuration"
        ],
        "created": "2026-06-26T11:03:33.193Z",
        "dataset": "lite_llm.audit",
        "id": "f6bbb635-8079-46bc-a7c7-49b38bfceb7a",
        "ingested": "2026-07-18T04:51:24Z",
        "kind": "event",
        "original": "{\"id\": \"f6bbb635-8079-46bc-a7c7-49b38bfceb7a\", \"updated_at\": \"2026-07-03T11:30:59.385000Z\", \"changed_by\": \"ops@berri.ai\", \"changed_by_api_key\": \"\", \"action\": \"updated\", \"table_name\": \"LiteLLM_UserTable\", \"object_id\": \"default_user_id\", \"before_value\": {\"spend\": 0.01098520000000001, \"teams\": [\"45d1cbaf-cf75-4052-9f51-a241d2518873\"], \"models\": [], \"user_id\": \"default_user_id\", \"metadata\": {}, \"policies\": [], \"user_role\": \"proxy_admin\", \"created_at\": \"2026-06-26T11:03:33.193000Z\", \"updated_at\": \"2026-07-03T11:12:06.577000Z\", \"model_spend\": {}, \"model_max_budget\": {}, \"allowed_cache_controls\": []}, \"updated_values\": {\"spend\": 0.01098520000000001, \"teams\": [\"45d1cbaf-cf75-4052-9f51-a241d2518873\"], \"models\": [], \"user_id\": \"default_user_id\", \"metadata\": {}, \"policies\": [], \"user_role\": \"proxy_admin\", \"created_at\": \"2026-06-26T11:03:33.193000Z\", \"updated_at\": \"2026-07-03T11:30:59.375000Z\", \"model_spend\": {}, \"model_max_budget\": {}, \"allowed_cache_controls\": []}}",
        "type": [
            "change"
        ]
    },
    "input": {
        "type": "aws-s3"
    },
    "lite_llm": {
        "audit": {
            "before_value": {
                "created_at": "2026-06-26T11:03:33.193Z",
                "spend": 0.01098520000000001,
                "teams": [
                    "45d1cbaf-cf75-4052-9f51-a241d2518873"
                ],
                "updated_at": "2026-07-03T11:12:06.577Z",
                "user_id": "default_user_id",
                "user_role": "proxy_admin"
            },
            "object_id": "default_user_id",
            "table_name": "LiteLLM_UserTable",
            "updated_values": {
                "created_at": "2026-06-26T11:03:33.193Z",
                "spend": 0.01098520000000001,
                "teams": [
                    "45d1cbaf-cf75-4052-9f51-a241d2518873"
                ],
                "updated_at": "2026-07-03T11:30:59.375Z",
                "user_id": "default_user_id",
                "user_role": "proxy_admin"
            }
        }
    },
    "log": {
        "file": {
            "path": "https://elastic-package-lite-llm-audit-bucket-49613.s3.us-east-2.amazonaws.com/audit.log"
        },
        "offset": 1474
    },
    "related": {
        "user": [
            "ops@berri.ai",
            "berri.ai",
            "default_user_id"
        ]
    },
    "tags": [
        "collect_sqs_logs",
        "preserve_original_event",
        "forwarded",
        "litellm-audit"
    ],
    "user": {
        "domain": "berri.ai",
        "email": "ops@berri.ai",
        "id": "default_user_id"
    }
}
```

### Inputs used

These inputs are used in the integration:

- [CEL](https://www.elastic.co/docs/reference/beats/filebeat/filebeat-input-cel)
- [AWS S3](https://www.elastic.co/docs/reference/beats/filebeat/filebeat-input-aws-s3)

### API usage

This integration dataset uses the following API:

- `Audit`: Collects audit logs via the **LiteLLM Audit API** (endpoint: `/audit`) or via **AWS S3/SQS** for organizations that export logs from LiteLLM to an S3 bucket.

### ILM Policy

To facilitate audit log data, the source data stream-backed index `.ds-logs-lite_llm.audit-*` is allowed to contain duplicates from each polling interval or S3 collection cycle. The ILM policy `logs-lite_llm.audit-default_policy` is added to this source index so it doesn't lead to unbounded growth. This means that in this source index data will be deleted after `30 days` from ingested date.