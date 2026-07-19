# LiteLLM Integration

## Overview

[LiteLLM](https://www.litellm.ai/) is a **unified LLM gateway and proxy** that provides a unified interface for accessing multiple large language models (LLMs) from different providers. It offers comprehensive audit logging, rate limiting, and cost tracking across hybrid LLM deployments — combining authentication, authorization, and detailed audit trails into a unified platform for **critical LLM infrastructure monitoring and compliance**.

The LiteLLM integration for Elastic collects spend tracking logs using the **LiteLLM Spend Tracking API** or through **AWS S3/SQS** cloud storage, and visualizes them in Kibana.

### Compatibility

The LiteLLM integration is compatible with **LiteLLM version 1.91.0 and above**.

### How it works

This integration supports two collection methods:

- **Direct API polling** via CEL input, which periodically queries the LiteLLM Spend Tracking API using API key authentication with cursor-based pagination.
- **Cloud storage** via AWS S3/SQS, for organizations that export spend tracking logs from LiteLLM to an AWS S3 bucket.

## What data does this integration collect?

The LiteLLM integration collects the following types of data:

| Data stream | Description | Source |
|---|---|---|
| `spend_tracking` | Spend and usage tracking records retrieved from the LiteLLM `/spend/logs/v2` API or S3 buckets, including token usage, costs, errors, and authentication events for each LLM request. | `/spend/logs/v2` API or AWS S3 |

### Supported use cases

Integrating LiteLLM with Elastic provides centralized visibility into LLM API usage, cost tracking, and error events across your LiteLLM deployment, enabling efficient cost monitoring, investigation, and compliance reporting within Kibana dashboards.

## What do I need to use this integration?

### From LiteLLM (API collection)

To collect data via the Spend Tracking API, you need an **API key** with permission to access the `/spend/logs/v2` endpoint.

1. Sign in to your LiteLLM deployment admin panel.
2. Navigate to **API Keys** or **Credentials**.
3. Create or select an API key with permission to read spend tracking logs from the `/spend/logs/v2` endpoint.

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

> **Note:** Spend tracking logs are exported to S3 in JSON format at regular intervals.

For more information on configuring S3 export in LiteLLM, refer to the [LiteLLM S3 export documentation](https://docs.litellm.ai/docs/proxy/logging).

## How do I deploy this integration?

This integration supports both Elastic Agentless-based and Agent-based installations.

### Agentless-based installation

Agentless integrations allow you to collect data without having to manage Elastic Agent in your cloud. They make manual agent deployment unnecessary, so you can focus on your data instead of the agent that collects it. For more information, refer to [Agentless integrations](https://www.elastic.co/guide/en/serverless/current/security-agentless-integrations.html) and the [Agentless integrations FAQ](https://www.elastic.co/guide/en/serverless/current/agentless-integration-troubleshooting.html).

Agentless deployments are only supported in Elastic Serverless and Elastic Cloud environments. This functionality is in beta and is subject to change. Beta features are not subject to the support SLA of official GA features.

### Agent-based installation

Elastic Agent must be installed. For more details, check the Elastic Agent [installation instructions](https://www.elastic.co/guide/en/fleet/current/elastic-agent-installation.html). You can install only one Elastic Agent per host.

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

* **No data collected (CEL)**: Verify that the LiteLLM `/spend/logs/v2` API is enabled and reachable from the Elastic Agent host. Confirm that the configured URL and API key are correct.
* **No data collected (S3)**: Verify that the S3 bucket exists and is accessible from the Elastic Agent host. Confirm that AWS credentials have S3:GetObject and S3:ListBucket permissions.
* **Authentication failures (CEL)**: Ensure the API key has permission to read spend tracking logs from the `/spend/logs/v2` endpoint. Verify the key has not been revoked or expired.
* **Authentication failures (S3)**: Verify AWS credentials are correct and have appropriate S3 permissions. Check that cross-account role assumption is configured correctly if using a role in a different account.
* **SSL certificate errors**: If LiteLLM or AWS endpoints use self-signed certificates, extract the certificate and configure it under the SSL settings of the integration, or add it to the Elastic Agent's trusted certificate store.
* **Pagination or rate limiting (CEL)**: If data collection is incomplete, verify batch size and interval settings. LiteLLM may enforce rate limits on the `/spend/logs/v2` endpoint.
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
- [LiteLLM API Reference (OpenAPI)](https://litellm-api.up.railway.app)
- [LiteLLM Model Hub](https://docs.litellm.ai/docs/proxy/model_management)

### Spend Tracking

The `spend_tracking` data stream provides LLM request usage and cost records collected from LiteLLM via API or S3.

#### Spend Tracking fields

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
| event.dataset | Identifies the LiteLLM spend tracking log dataset. | constant_keyword |
| event.module | Module that generated the event. | constant_keyword |
| input.type | Type of filebeat input. | keyword |
| lite_llm.spend_tracking.api_key | Serialized API-key identifier stored with the spend log; LiteLLM hashes real keys before persistence. | keyword |
| lite_llm.spend_tracking.cache_hit | String representation of whether the response was served from cache. | keyword |
| lite_llm.spend_tracking.cache_key | LiteLLM cache key or cache-disabled sentinel for the request. | keyword |
| lite_llm.spend_tracking.completion_start_time | Timestamp when completion generation or the first streamed token began. | date |
| lite_llm.spend_tracking.completion_tokens | Number of output or completion tokens generated by the call. | long |
| lite_llm.spend_tracking.messages | Serialized request messages sent to the model; LiteLLM may also emit list or object forms. | keyword |
| lite_llm.spend_tracking.metadata.additional_usage_values | Provider-specific usage metrics outside the standard token counters. | flattened |
| lite_llm.spend_tracking.metadata.applied_guardrails | Names of guardrails applied to the request. | keyword |
| lite_llm.spend_tracking.metadata.attempted_retries | Number of retries attempted for the request. | long |
| lite_llm.spend_tracking.metadata.batch_models | Models used for batch cost calculation. | keyword |
| lite_llm.spend_tracking.metadata.cold_storage_object_key | S3 or GCS object key used to retrieve archived request data. | keyword |
| lite_llm.spend_tracking.metadata.cost_breakdown | Detailed input, output, tool, cache, reasoning, and total cost breakdown. | keyword |
| lite_llm.spend_tracking.metadata.error_information.error_code | Error code reported for the failed request. | keyword |
| lite_llm.spend_tracking.metadata.error_information.error_message | Human-readable message for the failed request. | text |
| lite_llm.spend_tracking.metadata.error_information.error_rate_limit_category | Source category of a rate-limit failure, such as provider or LiteLLM rate limiting. | keyword |
| lite_llm.spend_tracking.metadata.error_information.error_rate_limit_type | Rate-limit dimension exceeded, such as requests, tokens, concurrency, budget, or iterations. | keyword |
| lite_llm.spend_tracking.metadata.error_information.llm_provider | LLM provider associated with the error. | keyword |
| lite_llm.spend_tracking.metadata.eval_information | Inferred: Evaluation metadata associated with the request. | flattened |
| lite_llm.spend_tracking.metadata.guardrail_information.guardrail_mode | Guardrail execution hook or mode; the observed value is a scalar, while LiteLLM also permits list or object forms. | keyword |
| lite_llm.spend_tracking.metadata.guardrail_information.guardrail_response.detected_entities | Inferred: Entity categories detected by the PII guardrail. | keyword |
| lite_llm.spend_tracking.metadata.guardrail_information.guardrail_response.flagged | Inferred: Whether the guardrail response flagged the request content. | boolean |
| lite_llm.spend_tracking.metadata.guardrail_information.guardrail_status | Guardrail execution status; current LiteLLM definitions use success, guardrail_intervened, guardrail_failed_to_respond, or not_run, while this sample also contains blocked. | keyword |
| lite_llm.spend_tracking.metadata.litellm_overhead_time_ms | LiteLLM processing overhead in milliseconds. | double |
| lite_llm.spend_tracking.metadata.max_retries | Maximum number of retries configured for the request. | long |
| lite_llm.spend_tracking.metadata.mcp_tool_call_metadata | MCP tool-call arguments, result, server identity, and cost metadata. | keyword |
| lite_llm.spend_tracking.metadata.model_map_information | LiteLLM model-map key and resolved model information. | keyword |
| lite_llm.spend_tracking.metadata.proxy_server_request | Inferred: Serialized proxy request information retained with the spend log. | keyword |
| lite_llm.spend_tracking.metadata.requester_ip_address | Requester IP address copied into LiteLLM metadata. | ip |
| lite_llm.spend_tracking.metadata.spend_logs_metadata | Custom key-value metadata supplied for spend logging. | flattened |
| lite_llm.spend_tracking.metadata.status | Status copied into LiteLLM spend metadata. | keyword |
| lite_llm.spend_tracking.metadata.usage_object | Raw usage object returned by the LLM provider. | flattened |
| lite_llm.spend_tracking.metadata.user_api_key | API-key identifier retained in spend metadata. | keyword |
| lite_llm.spend_tracking.metadata.user_api_key_alias | Human-readable alias of the LiteLLM virtual API key. | keyword |
| lite_llm.spend_tracking.metadata.user_api_key_org_id | Organization identifier associated with the API key. | keyword |
| lite_llm.spend_tracking.metadata.user_api_key_project_alias | Human-readable alias of the project associated with the API key. | keyword |
| lite_llm.spend_tracking.metadata.user_api_key_project_id | Identifier of the project associated with the API key. | keyword |
| lite_llm.spend_tracking.metadata.user_api_key_team_id | Team identifier associated with the API key. | keyword |
| lite_llm.spend_tracking.metadata.vector_store_request_metadata | Vector-store request metadata associated with the LLM call. | keyword |
| lite_llm.spend_tracking.model_group | LiteLLM routing model group used for the request. | keyword |
| lite_llm.spend_tracking.model_id | Identifier of the LiteLLM model deployment used for the call. | keyword |
| lite_llm.spend_tracking.prompt_tokens | Number of input or prompt tokens used by the call. | long |
| lite_llm.spend_tracking.request_duration_ms | Total request duration stored in milliseconds. | long |
| lite_llm.spend_tracking.response | Model response retained in the spend log; LiteLLM may also emit list or object forms. | keyword |
| lite_llm.spend_tracking.response_cost | Calculated response cost in US dollars. | double |
| lite_llm.spend_tracking.response_time | Total response time in seconds; for streaming calls this may represent time to first token. | double |
| lite_llm.spend_tracking.session_id | Identifier used to group related LiteLLM requests into one session. | keyword |
| lite_llm.spend_tracking.spend | Spend charged for the request in US dollars. | double |
| lite_llm.spend_tracking.total_tokens | Total number of input and output tokens used by the call. | long |
| lite_llm.spend_tracking.user_api_key_hash | Hash of the LiteLLM virtual API key used for the request. | keyword |
| log.offset | Log offset. | long |
| observer.product | Product name of the observer that generated the event. | constant_keyword |
| observer.vendor | Vendor name of the observer that generated the event. | constant_keyword |


### Example event

#### Spend Tracking

An example event for `spend_tracking` looks as following:

```json
{
    "@timestamp": "2026-07-18T10:11:19.909Z",
    "agent": {
        "ephemeral_id": "bae89555-6f44-4e24-899b-1fb48a4e5105",
        "id": "faabe81e-f245-4791-a743-b178774b4139",
        "name": "elastic-agent-56227",
        "type": "filebeat",
        "version": "8.18.0"
    },
    "aws": {
        "s3": {
            "bucket": {
                "arn": "arn:aws:s3:::elastic-package-lite-llm-spend-tracking-bucket-72730",
                "name": "elastic-package-lite-llm-spend-tracking-bucket-72730"
            },
            "object": {
                "key": "spend-tracking.log"
            }
        }
    },
    "cloud": {
        "region": "us-east-2"
    },
    "data_stream": {
        "dataset": "lite_llm.spend_tracking",
        "namespace": "46564",
        "type": "logs"
    },
    "ecs": {
        "version": "9.4.0"
    },
    "elastic_agent": {
        "id": "faabe81e-f245-4791-a743-b178774b4139",
        "snapshot": false,
        "version": "8.18.0"
    },
    "error": {
        "type": "ProxyModelNotFoundError"
    },
    "event": {
        "agent_id_status": "verified",
        "category": [
            "api"
        ],
        "dataset": "lite_llm.spend_tracking",
        "duration": 0,
        "end": "2026-07-18T10:11:19.909Z",
        "id": "7d8f7a77-9f46-47b2-997e-0a0892fe2c7a",
        "ingested": "2026-07-19T02:54:36Z",
        "kind": "event",
        "original": "{\"request_id\":\"7d8f7a77-9f46-47b2-997e-0a0892fe2c7a\",\"call_type\":\"\",\"api_key\":\"8de15ff28eedf53bd4cf2e65b8e980b9231acc3f894648a9fa3c2dbf0ab1a6d9\",\"spend\":0,\"total_tokens\":0,\"prompt_tokens\":0,\"completion_tokens\":0,\"start_time\":\"2026-07-18T10:11:19.909+00:00\",\"end_time\":\"2026-07-18T10:11:19.909+00:00\",\"model\":\"gemma-4-31b-it\",\"user\":\"default_user_id\",\"status\":\"failure\",\"metadata\":{\"error_information\":{\"error_code\":\"400\",\"error_class\":\"ProxyModelNotFoundError\",\"error_message\":\"Invalid model name passed in model=gemma-4-31b-it\"}}}",
        "outcome": "failure",
        "start": "2026-07-18T10:11:19.909Z",
        "type": [
            "access"
        ]
    },
    "gen_ai": {
        "request": {
            "model": "gemma-4-31b-it"
        }
    },
    "input": {
        "type": "aws-s3"
    },
    "lite_llm": {
        "spend_tracking": {
            "api_key": "8de15ff28eedf53bd4cf2e65b8e980b9231acc3f894648a9fa3c2dbf0ab1a6d9",
            "completion_tokens": 0,
            "metadata": {
                "error_information": {
                    "error_code": "400",
                    "error_message": "Invalid model name passed in model=gemma-4-31b-it"
                }
            },
            "prompt_tokens": 0,
            "spend": 0,
            "total_tokens": 0
        }
    },
    "log": {
        "file": {
            "path": "https://elastic-package-lite-llm-spend-tracking-bucket-72730.s3.us-east-2.amazonaws.com/spend-tracking.log"
        },
        "offset": 1900
    },
    "related": {
        "user": [
            "default_user_id"
        ]
    },
    "tags": [
        "collect_sqs_logs",
        "preserve_original_event",
        "forwarded",
        "litellm-spend_tracking"
    ],
    "user": {
        "id": "default_user_id"
    }
}
```

### Inputs used

These inputs are used in the integration:

- [CEL](https://www.elastic.co/docs/reference/beats/filebeat/filebeat-input-cel)
- [AWS S3](https://www.elastic.co/docs/reference/beats/filebeat/filebeat-input-aws-s3)

### API usage

This integration uses the following API:

- `Spend Tracking`: Collects spend tracking records via the **LiteLLM Spend Tracking API** (endpoint: `/spend/logs/v2`) or via **AWS S3/SQS** for organizations that export spend tracking data from LiteLLM to an S3 bucket.
