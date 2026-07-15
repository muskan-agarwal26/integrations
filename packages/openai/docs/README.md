# OpenAI

The OpenAI integration allows you to monitor OpenAI API usage metrics, collect organization audit logs, and collect ChatGPT Enterprise compliance logs. OpenAI is an AI research and deployment company that offers [API platform](https://openai.com/api) for their industry-leading foundation models.

With the OpenAI integration, you can track API usage metrics across their models, as well as for vector store and code interpreter. You can also collect audit logs from the OpenAI platform to monitor user actions, API key lifecycle events, and organization configuration changes. Additionally, you can collect ChatGPT Enterprise compliance logs to monitor application activity, authentication events, and Codex usage across your workspace or organization. You will use Kibana to visualize your data, create alerts if usage limits are approaching, view metrics when you troubleshoot issues, and analyze audit and compliance events for security and compliance. For example, you can track token usage and API calls per model, as well as login attempts, API key creation/deletion, and role assignments.

## Data collection

The OpenAI integration leverages the following OpenAI APIs for data collection:

- **Usage API**: The [OpenAI Usage API](https://platform.openai.com/docs/api-reference/usage) delivers comprehensive insights into your API activity, helping you understand and optimize your organization's OpenAI API usage.

- **Audit Logs API**: The [OpenAI Audit Logs API](https://platform.openai.com/docs/api-reference/audit-logs) collects organization audit logs, providing visibility into user actions, API key lifecycle events, login attempts, role assignments, and other platform activity for security oversight and compliance.

- **Compliance Logs Platform API**: The [OpenAI Compliance Logs Platform](https://help.openai.com/en/articles/9261474-compliance-api-for-enterprise-customers) collects ChatGPT Enterprise compliance logs including app_log, app_auth_log, auth_log, audit_log, codex_log, and codex_security_log at the workspace or organization level for security monitoring and regulatory compliance.

- **Rate Limits API**: The [OpenAI Rate Limits API](https://platform.openai.com/docs/api-reference/project-rate-limits) collects the configured per-project, per-model rate limits (requests, tokens and images per minute, plus daily and batch limits). Combined with usage data, this lets you monitor how close each project is to being throttled.

## Data streams

The OpenAI integration collects the following data streams:

- `audit`: Collects organization audit logs.
- `audio_speeches`: Collects audio speeches usage metrics.
- `audio_transcriptions`: Collects audio transcriptions usage metrics.
- `code_interpreter_sessions`: Collects code interpreter sessions usage metrics.
- `completions`: Collects completions usage metrics.
- `compliance`: Collects ChatGPT Enterprise compliance logs.
- `embeddings`: Collects embeddings usage metrics.
- `images`: Collects images usage metrics.
- `moderations`: Collects moderations usage metrics.
- `rate_limits`: Collects per-project, per-model rate limits.
- `vector_stores`: Collects vector stores usage metrics.

> Note: Users can view OpenAI metrics in the `logs-*` index pattern using Kibana Discover.

## Requirements

You need Elasticsearch for storing and searching your data and Kibana for visualizing and managing it.

You need an OpenAI account with a valid [Admin key](https://platform.openai.com/settings/organization/admin-keys) for programmatic access to the [OpenAI Usage API](https://platform.openai.com/docs/api-reference/usage) and [OpenAI Audit Logs API](https://platform.openai.com/docs/api-reference/audit-logs). To fetch audit logs, you must enable audit logging on the OpenAI platform in your organization settings under Data controls > Data retention. Audit logs also require Organization Owner permissions.

To collect compliance logs, you additionally need a **Compliance API key** authorized for enterprise logs. The Compliance Logs Platform is available to ChatGPT Enterprise customers, and the key must be granted the `chatgpt.enterprise.compliance_export.read` scope by OpenAI support. You also need the target workspace ID or organization ID whose logs you want to collect. See [Compliance API for enterprise customers](https://help.openai.com/en/articles/9261474-compliance-api-for-enterprise-customers) for details.

## Setup

For step-by-step instructions on how to set up an integration, see the [Getting started](https://www.elastic.co/guide/en/starting-with-the-elasticsearch-platform-and-its-solutions/current/getting-started-observability.html) guide.

### Generate an Admin key

To generate an Admin key, please generate a key or use an existing one from the [Admin keys](https://platform.openai.com/settings/organization/admin-keys) page. Use the Admin key to configure the OpenAI integration.

### Generate a Compliance API key

The Compliance Logs Platform is available to ChatGPT Enterprise customers. Request a Compliance API key from OpenAI and ensure it is authorized for enterprise logs (contact OpenAI support to grant the key the `chatgpt.enterprise.compliance_export.read` scope). Use this key, together with the workspace or organization ID whose logs you want to collect, to configure the `compliance` data stream. See [Compliance API for enterprise customers](https://help.openai.com/en/articles/9261474-compliance-api-for-enterprise-customers) for details.

## Collection behavior

Among the configuration options for the OpenAI integration, the following settings are particularly relevant: "Initial interval", "Bucket width" and "Finalization grace period" for usage metrics, and "Initial interval" and "Interval" for audit and compliance logs.

### Initial interval

- Controls the historical data collection window at startup
- Default value: 24 hours (`24h`)
- Purpose: Loads historical context when you first set up the integration

### Bucket width

A "bucket" refers to a time interval where OpenAI usage data is grouped together for reporting purposes. For example, with a 1-minute bucket width, usage metrics are aggregated minute by minute. With a 1-hour bucket width, all activity during that hour is consolidated into a single bucket. The [bucket width](https://platform.openai.com/docs/api-reference/usage/completions#usage-completions-bucket_width) determines your data's granularity and level of detail in your usage reporting.

- Controls the time-based aggregation of metrics
- Default: `1m` (1 minute)
- Options: `1m` (1 minute), `1h` (1 hour), `1d` (1 day)
- Affects API request frequency and data resolution

#### Impact on data resolution

- `1m` buckets provide the highest resolution metrics, with data arriving in near real-time (1-minute delay)
- `1h` buckets aggregate hourly, with data arriving less frequently (1-hour delay)
- `1d` buckets aggregate daily, with data arriving once per day (24-hour delay)

Data granularity relationship: `1m` > `1h` > `1d`

#### Storage considerations

Bucket width choice affects storage usage (in Elasticsearch) and data resolution:

- `1m`: Maximum granularity, higher storage needs, ideal for detailed analysis.
- `1h`: Medium granularity, moderate storage needs, good for hourly patterns.
- `1d`: Minimum granularity, lowest storage needs, suitable for long-term analysis.

Example: For 100 API calls to a particular model per hour:
- `1m` buckets: Up to 100 documents
- `1h` buckets: 1 aggregated document
- `1d` buckets: 1 daily document

#### API request impact

"Bucket width" and "Initial interval" directly affect API request frequency. When using a 1-minute bucket width, it's strongly recommended to set the "Initial interval" to a shorter duration, optimally 1-day, to ensure smooth performance. While our extensive testing demonstrates excellent results with a 6-month initial interval paired with a 1-day bucket width, the same level of success isn't achievable with 1-minute or 1-hour bucket widths. This is because the OpenAI Usage API returns different bucket quantities based on width (60 buckets per call for 1-minute, 24 for 1-hour, and 7 for 1-day widths). To achieve the best results when gathering historical data over long periods, using 1-day bucket width is the most effective method, ensuring a balance between data granularity and API limitations.

> For optimal results with historical data, use 1-day bucket widths for long periods (15+ days), 1-hour for medium periods (1-15 days), and 1-minute only for the most recent 24 hours of data.

### Finalization grace period

OpenAI's Usage API does not finalize a per-minute bucket the moment it ends — a bucket's token and request counts keep climbing for several minutes afterward as late usage is accounted for. To avoid ingesting these partial, still-changing counts, each of the six token- and request-based usage data streams — `completions`, `embeddings`, `moderations`, `images`, `audio_speeches`, and `audio_transcriptions` — waits a configurable **finalization grace period** before treating a bucket as final. (The `code_interpreter_sessions` and `vector_stores` streams do not report token or request counts, do not feed the rate limit headroom dashboard, and have no grace setting.)

> **Note:** This finalization lag is *observed* behavior, not a documented OpenAI guarantee. The Usage API reference does not specify a revision window or finalization delay, so the recommended `15m` value below is an empirically chosen buffer, not a figure published by OpenAI; the lag may change without notice.

- Controls how long to wait after a bucket's end time before the bucket is ingested
- Default value: `0s`. Buckets are ingested as soon as they are read, with no added delay. This keeps usage data fresh and is the least disruptive setting for existing deployments, but it can undercount usage during heavy bursts, because OpenAI is still revising a bucket's counts for several minutes after its end time and the bucket is read before those counts settle.
- Recommended for accurate counts: `15m`. Setting the grace to 15 minutes gives each per-minute bucket time to finalize before it is ingested, so the stored token and request counts match what OpenAI eventually reports. Choose this when accurate usage and rate limit headroom matter more than freshness — for example, when reconciling against the OpenAI dashboard or driving the headroom alert.
- How it works: buckets whose end time is younger than the grace period are skipped and re-fetched on a later poll, so only finalized counts are stored.
- Trade-off: a longer grace period is safer against undercounting (heavy bursts take OpenAI longer to finalize) but delays when usage first appears in Elasticsearch by up to the grace period. With `15m`, these six usage data streams — and the rate limit headroom panels and alert that read from them — trail real time by roughly 15 minutes. With the default `0s` there is no added lag, but the most recent minutes can be permanently undercounted: each bucket is ingested as soon as its minute closes — before OpenAI finishes revising it — and is not re-read afterward, so later upward revisions are never reflected.
- Where to change it: use the **Finalization grace period** setting for each of these six usage data streams.

> If usage metrics read lower than the OpenAI dashboard under high-volume bursts, increase the finalization grace period — `15m` is recommended. If you need the freshest possible dashboards and can tolerate undercounting the most recent minutes, keep the default `0s`.

#### Known limitation: residual undercount under high-volume bursts

The finalization grace period eliminates the bulk of the undercount, but it cannot fully remove it. OpenAI's per-minute usage finalization is non-monotonic: during high-volume bursts a bucket's counts can continue to be revised upward for hours — beyond any fixed grace period. Because each bucket is ingested once and not re-fetched after it is treated as final, those late revisions are not reflected, so a small residual undercount (observed at a few percent of a single minute's volume) may remain for the busiest buckets. This does not affect the headroom dashboard's ability to flag over-limit conditions, since the gap is small relative to the limit. If you require usage counts that exactly match the OpenAI dashboard, prefer a larger bucket width (`1h` or `1d`), which OpenAI finalizes more stably than `1m`.

### Collection process

With default settings (Interval: `5m`, Bucket width: `1m`, Initial interval: `24h`), the OpenAI integration follows this collection pattern:

1. Starts collection from (current_time - initial_interval)
2. Collects data up to (current_time - finalization_grace)
3. Skips buckets OpenAI has not yet finalized (those whose end time is within the finalization grace period) and re-fetches them on a later poll once they are final
4. Runs every 5 minutes by default (configurable)
5. From the second collection onward, resumes from the oldest not-yet-finalized bucket so late-arriving counts are captured without duplication

#### Example timeline

With these settings (Interval: `5m`, Bucket width: `1m`, Initial interval: `24h`) and a finalization grace period of `15m` (the recommended setting for accurate counts):

The integration starts at 10:00 AM and collects data from 10:00 AM the previous day up to 9:45 AM the current day — the most recent 15 minutes are still finalizing and are skipped. The next collection starts at 10:05 AM and resumes from the 9:45 AM bucket, re-fetching any of those buckets that have since been finalized and continuing up to 9:50 AM.

With the default grace period of `0s`, collection instead runs all the way up to the current time and no buckets are skipped — each bucket is ingested as soon as its minute closes. Those buckets appear immediately, but because they are read before OpenAI finishes revising them and are not re-read once ingested, any later upward revisions are not reflected and the counts can stay undercounted.

## Rate limit headroom

The `rate_limits` data stream collects the per-project, per-model limits OpenAI enforces (requests, tokens and images per minute, plus daily and batch limits). On its own a limit is just a number; it becomes actionable when compared against actual usage. The OpenAI dashboard ships two **Rate limit headroom** panels — one broken down per project and model, and an org-wide rollup by model across all active projects — plus a prebuilt **[OpenAI] Rate limit headroom low** threshold alert that do exactly this.

### How the comparison works

Limits and usage are joined on the exact `project_id`, but the `model` is **normalized** before joining: a trailing dated snapshot suffix (`-YYYY-MM-DD`) is stripped from both sides so that each model collapses to its base family. The Usage API reports per dated snapshot (for example `gpt-image-1-2025-04-23`, `omni-moderation-2024-09-26`), while the Rate Limits API often lists only the base family name (`gpt-image-1`, `omni-moderation`). Without normalization the dated usage row would find no matching limit and be dropped from the join, so the queries apply `REPLACE(<model>, "-[0-9]{4}-[0-9]{2}-[0-9]{2}$", "")` to align dated usage snapshots with base rate-limit names. When the Rate Limits API does report the dated snapshot as its own row, normalization simply collapses it onto the same base key as the family row.

Utilization is computed as `usage / limit`, where usage is the **peak** value over the look-back window:

1. Usage is summed per project, model and 1-minute bucket.
2. The peak (maximum) minute in the window is taken.
3. That peak is divided by the configured limit.

Only matching units are compared:

- tokens ↔ tokens per minute (TPM)
- requests ↔ requests per minute (RPM)
- images ↔ images per minute

Audio is deliberately left out. OpenAI enforces an audio limit (`max_audio_megabytes_per_1_minute`), but the Usage API reports audio only in seconds (`audio_transcriptions`) and characters (`audio_speeches`) — never in megabytes — so there is no comparable usage figure to divide by the limit. Audio therefore stays usage-only until a megabyte-denominated usage metric is available. Usage measured in characters or sessions likewise has no corresponding rate limit and stays usage-only.

> **Hard requirement:** the peak 1-minute calculation depends on the usage streams (`completions`, `embeddings`, `moderations`, ...) running with **`bucket_width: 1m`**. This is the default, but the value is user-editable — if it is changed to `1h` or `1d`, the headroom numbers will be wrong because a wider bucket smears per-minute peaks.

> **Note on aggregation:** OpenAI returns identical limits for both the model family (`gpt-4o-mini`) and its dated snapshot (`gpt-4o-mini-2024-07-18`). After normalization both collapse onto the same base `model` key, so the per-minute aggregation takes the limit as `MAX(...)` over the rows in each `project_id`/`model`/minute bucket. `MAX` (not `SUM`) is what prevents the duplicated family and snapshot limit rows from double-counting capacity — they report the same number, so the max equals either one. A base model that has a limit but no usage in the window appears as a 0%-utilization row.

### Org-wide rollup by model

The **Rate limit headroom - by model (org-wide)** panel answers a different question: "how is model X doing across the whole org?" It drops the `project_id` breakdown and aggregates by model alone. For each project it first takes that project's peak 1-minute usage over the window (exactly as the per-project panel does), then sums those per-project peaks into the org-wide usage figure.

Because each project's peak can fall in a different minute, this sum-of-peaks is an **indicative upper bound** rather than a true simultaneous org-wide peak — it assumes every project peaked at once, so it can read higher than the actual combined load in any single minute. This is intentional: it keeps the usage rollup aligned with the limit rollup (see below) and errs toward flagging pressure rather than hiding it.

OpenAI enforces rate limits **per project**, so there is no single org-wide throttle boundary to divide against. The rollup therefore presents its limit columns as a **synthetic aggregate** (each project's limit stabilized as the max over the window, then summed across projects) and labels both the limit and utilization columns accordingly (`limit (aggregate)`, `utilization (approx.)`). Use these for relative comparison and trend-spotting across models; the exact throttle distance for any individual project still lives in the per-project panel and the alert.

### Alert

The **[OpenAI] Rate limit headroom low** rule fires when peak 1-minute token usage (TPM) reaches 80% or more of the configured limit for a project and model. It is grouped by `project_id::model` and re-examines the most recent 15-minute window every 5 minutes.

The rule keys on the **peak** minute in that window, so even a single 1-minute spike at or above the threshold is enough to fire it — usage does not have to stay high. Because the 15-minute look-back is three times the 5-minute schedule, one breaching minute stays in view across the three consecutive runs that the `alertDelay` of 3 requires, so it satisfies the delay on its own. The net effect is that the alert fires roughly 10–15 minutes after a breaching minute rather than only on sustained pressure; the delay suppresses an alert from a breach seen on just one or two runs, not from a single high minute.

> **Scope:** this rule tracks **token** utilization (TPM) only. Request-per-minute (RPM) and image-per-minute headroom appear on the dashboard panels but are **not** alerted — a project can approach its RPM or image limit without firing this rule. Use the panels to watch those dimensions.

> **Dependency:** the comparison needs a current limit. If the `rate_limits` stream stops collecting (for example, an expired admin key or a projects-list API error), the limit ages out of the window, every utilization becomes null, and this rule silently stops firing rather than alerting on the gap. If you rely on the alert, also monitor the freshness of `event.dataset: openai.rate_limits`.

To tune the threshold, edit the `WHERE tpm_utilization >= 0.8` line in the rule's ES|QL query.

## Metrics reference

**ECS Field Reference**

Refer to this [document](https://www.elastic.co/guide/en/ecs/current/ecs-field-reference.html) for detailed information on ECS fields.

### Audit logs

The `audit` data stream captures organization audit logs.

An example event for `audit` looks as following:

```json
{
    "@timestamp": "2026-02-09T05:50:27.000Z",
    "agent": {
        "ephemeral_id": "673d5468-31d6-4fbd-9620-144246698edd",
        "id": "91b27281-9bde-42a5-a100-529612a7e293",
        "name": "elastic-agent-35386",
        "type": "filebeat",
        "version": "8.18.0"
    },
    "data_stream": {
        "dataset": "openai.audit",
        "namespace": "91451",
        "type": "logs"
    },
    "ecs": {
        "version": "9.3.0"
    },
    "elastic_agent": {
        "id": "91b27281-9bde-42a5-a100-529612a7e293",
        "snapshot": false,
        "version": "8.18.0"
    },
    "event": {
        "action": "login-succeeded",
        "agent_id_status": "verified",
        "category": [
            "iam"
        ],
        "dataset": "openai.audit",
        "id": "audit_log-iakaJWnLdMLe89Ml5xayb4RJ",
        "ingested": "2026-05-22T11:51:38Z",
        "kind": "event",
        "original": "{\"actor\":{\"session\":{\"ip_address\":\"163.116.213.142\",\"ip_address_details\":{\"asn\":\"55256\",\"city\":\"Mumbai\",\"country\":\"IN\",\"latitude\":\"19.07283\",\"longitude\":\"72.88261\",\"region\":\"Maharashtra\",\"region_code\":\"MH\"},\"ja3\":\"353a03072e841d5004dfaa78561ccc70\",\"ja4\":\"t13d691100_8b2139ff7677_f0e8fcc46740\",\"user\":{\"email\":\"muskan.agarwal@crestdata.ai\",\"id\":\"user-nuEwi8Vsrrddq53RTzEQ9p3c\"},\"user_agent\":\"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36\"},\"type\":\"session\"},\"effective_at\":1770616227,\"id\":\"audit_log-iakaJWnLdMLe89Ml5xayb4RJ\",\"object\":\"organization.audit_log\",\"type\":\"login.succeeded\"}",
        "type": [
            "change"
        ]
    },
    "input": {
        "type": "cel"
    },
    "observer": {
        "product": "OpenAI",
        "vendor": "OpenAI"
    },
    "openai": {
        "audit": {
            "actor": {
                "session": {
                    "ip_address": "163.116.213.142",
                    "ip_address_details": {
                        "asn": 55256,
                        "city": "Mumbai",
                        "country": "IN",
                        "latitude": 19.07283,
                        "longitude": 72.88261,
                        "region": "Maharashtra",
                        "region_code": "MH"
                    },
                    "ja3": "353a03072e841d5004dfaa78561ccc70",
                    "ja4": "t13d691100_8b2139ff7677_f0e8fcc46740",
                    "user": {
                        "email": "muskan.agarwal@crestdata.ai",
                        "id": "user-nuEwi8Vsrrddq53RTzEQ9p3c"
                    },
                    "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36"
                },
                "type": "session"
            },
            "effective_at": "2026-02-09T05:50:27.000Z",
            "id": "audit_log-iakaJWnLdMLe89Ml5xayb4RJ",
            "object": "organization.audit_log",
            "type": "login.succeeded"
        }
    },
    "related": {
        "ip": [
            "163.116.213.142"
        ],
        "user": [
            "muskan.agarwal@crestdata.ai",
            "user-nuEwi8Vsrrddq53RTzEQ9p3c"
        ]
    },
    "source": {
        "as": {
            "number": 55256
        },
        "geo": {
            "city_name": "Mumbai",
            "country_iso_code": "IN",
            "location": {
                "lat": 19.07283,
                "lon": 72.88261
            },
            "region_iso_code": "MH",
            "region_name": "Maharashtra"
        },
        "ip": "163.116.213.142"
    },
    "tags": [
        "preserve_original_event",
        "preserve_duplicate_custom_fields",
        "forwarded",
        "openai-audit"
    ],
    "tls": {
        "client": {
            "ja3": "353a03072e841d5004dfaa78561ccc70"
        }
    },
    "user": {
        "domain": "crestdata.ai",
        "email": "muskan.agarwal@crestdata.ai",
        "id": "user-nuEwi8Vsrrddq53RTzEQ9p3c"
    },
    "user_agent": {
        "original": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/144.0.0.0 Safari/537.36"
    }
}
```

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Date/time when the event originated. This is the date/time extracted from the event, typically representing when the event was generated by the source. If the event source has no original timestamp, this value is typically populated by the first time the event was received by the pipeline. Required field for all events. | date |
| data_stream.dataset | The field can contain anything that makes sense to signify the source of the data. Examples include `nginx.access`, `prometheus`, `endpoint` etc. For data streams that otherwise fit, but that do not have dataset set we use the value "generic" for the dataset value. `event.dataset` should have the same value as `data_stream.dataset`. Beyond the Elasticsearch data stream naming criteria noted above, the `dataset` value has additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.namespace | A user defined namespace. Namespaces are useful to allow grouping of data. Many users already organize their indices this way, and the data stream naming scheme now provides this best practice as a default. Many users will populate this field with `default`. If no value is used, it falls back to `default`. Beyond the Elasticsearch index naming criteria noted above, `namespace` value has the additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.type | An overarching type for the data stream. Currently allowed values are "logs" and "metrics". We expect to also add "traces" and "synthetics" in the near future. | constant_keyword |
| event.dataset | Name of the dataset. If an event source publishes more than one type of log or events (e.g. access log, error log), the dataset is used to specify which one the event comes from. It's recommended but not required to start the dataset name with the module name, followed by a dot, then the dataset name. | constant_keyword |
| event.module | Name of the module this data is coming from. If your monitoring agent supports the concept of modules or plugins to process events of a given source (e.g. Apache logs), `event.module` should contain the name of this module. | constant_keyword |
| input.type | Type of filebeat input. | keyword |
| log.offset | Log offset. | long |
| observer.product | The product name of the observer. | constant_keyword |
| observer.vendor | Vendor name of the observer. | constant_keyword |
| openai.audit.actor.api_key.id | The tracking id of the API key. | keyword |
| openai.audit.actor.api_key.service_account.id | The service account that performed the audit logged action. | keyword |
| openai.audit.actor.api_key.type | The type of API key. Can be either user or service_account. | keyword |
| openai.audit.actor.api_key.user.email | The user who performed the audit logged action. | keyword |
| openai.audit.actor.api_key.user.id | The user who performed the audit logged action. | keyword |
| openai.audit.actor.session.ip_address | The IP address from which the action was performed. | ip |
| openai.audit.actor.session.ip_address_details.asn |  | long |
| openai.audit.actor.session.ip_address_details.city |  | keyword |
| openai.audit.actor.session.ip_address_details.country |  | keyword |
| openai.audit.actor.session.ip_address_details.latitude |  | double |
| openai.audit.actor.session.ip_address_details.longitude |  | double |
| openai.audit.actor.session.ip_address_details.region |  | keyword |
| openai.audit.actor.session.ip_address_details.region_code |  | keyword |
| openai.audit.actor.session.ja3 |  | keyword |
| openai.audit.actor.session.ja4 |  | keyword |
| openai.audit.actor.session.user.email | The session in which the audit logged action was performed. The user who performed the audit logged action. The user email. | keyword |
| openai.audit.actor.session.user.id | The session in which the audit logged action was performed. The user who performed the audit logged action. The user id. | keyword |
| openai.audit.actor.session.user_agent |  | keyword |
| openai.audit.actor.type | The type of actor. Is either session or api_key. | keyword |
| openai.audit.effective_at | The Unix timestamp (in seconds) of the event. | date |
| openai.audit.id | The ID of this log. | keyword |
| openai.audit.object |  | keyword |
| openai.audit.type | The event type. | keyword |


### Audio speeches

The `audio_speeches` data stream captures audio speeches usage metrics.

An example event for `audio_speeches` looks as following:

```json
{
    "openai": {
        "audio_speeches": {
            "characters": 45
        },
        "base": {
            "start_time": "2024-09-05T00:00:00.000Z",
            "num_model_requests": 1,
            "project_id": "proj_dummy",
            "user_id": "user-dummy",
            "end_time": "2024-09-06T00:00:00.000Z",
            "model": "tts-1",
            "api_key_id": "key_dummy",
            "usage_object_type": "organization.usage.audio_speeches.result"
        }
    },
    "@timestamp": "2024-09-05T00:00:00.000Z",
    "ecs": {
        "version": "9.3.0"
    },
    "data_stream": {
        "namespace": "default",
        "type": "logs",
        "dataset": "openai.audio_speeches"
    },
    "event": {
        "agent_id_status": "verified",
        "ingested": "2025-01-28T21:56:19Z",
        "created": "2025-01-28T21:56:18.550Z",
        "kind": "metric",
        "dataset": "openai.audio_speeches"
    },
    "tags": [
        "forwarded",
        "openai-audio-speeches"
    ]
}
```

**Exported fields**

| Field | Description | Type | Unit |
|---|---|---|---|
| @timestamp | Event timestamp. | date |  |
| data_stream.dataset | Data stream dataset. | constant_keyword |  |
| data_stream.namespace | Data stream namespace. | constant_keyword |  |
| data_stream.type | Data stream type. | constant_keyword |  |
| openai.audio_speeches.characters | Number of characters processed | long |  |
| openai.audio_transcriptions.seconds | Number of seconds processed | long | s |
| openai.base.api_key_id | Identifier for the API key used | keyword |  |
| openai.base.end_time | End timestamp of the usage bucket | date |  |
| openai.base.model | Name of the OpenAI model used | keyword |  |
| openai.base.num_model_requests | Number of requests made to the model | long |  |
| openai.base.project_id | Identifier of the project | keyword |  |
| openai.base.start_time | Start timestamp of the usage bucket | date |  |
| openai.base.usage_object_type | Type of the usage record | keyword |  |
| openai.base.user_id | Identifier of the user | keyword |  |
| openai.completions.input_audio_tokens | Number of audio input tokens used, including cached tokens | long |  |
| openai.completions.input_cached_tokens | Number of text input tokens that has been cached from previous requests. For customers subscribe to scale tier, this includes scale tier tokens | long |  |
| openai.completions.input_tokens | Number of text input tokens used, including cached tokens. For customers subscribe to scale tier, this includes scale tier tokens | long |  |
| openai.completions.output_audio_tokens | Number of audio output tokens used | long |  |
| openai.completions.output_tokens | Number of text output tokens used. For customers subscribe to scale tier, this includes scale tier tokens | long |  |
| openai.embeddings.input_tokens | Number of input tokens used. | long |  |
| openai.images.images | Number of images processed | long |  |
| openai.images.size | Image size (dimension of the generated image) | keyword |  |
| openai.moderations.input_tokens | Number of input tokens used. | long |  |


### Audio transcriptions

The `audio_transcriptions` data stream captures audio transcriptions usage metrics.

An example event for `audio_transcriptions` looks as following:

```json
{
    "openai": {
        "audio_transcriptions": {
            "seconds": 2
        },
        "base": {
            "start_time": "2024-11-04T00:00:00.000Z",
            "num_model_requests": 1,
            "project_id": "proj_dummy",
            "user_id": "user-dummy",
            "end_time": "2024-11-05T00:00:00.000Z",
            "model": "whisper-1",
            "api_key_id": "key_dummy",
            "usage_object_type": "organization.usage.audio_transcriptions.result"
        }
    },
    "@timestamp": "2024-11-04T00:00:00.000Z",
    "ecs": {
        "version": "9.3.0"
    },
    "data_stream": {
        "namespace": "default",
        "type": "logs",
        "dataset": "openai.audio_transcriptions"
    },
    "event": {
        "agent_id_status": "verified",
        "ingested": "2025-01-28T21:56:24Z",
        "created": "2025-01-28T21:56:24.113Z",
        "kind": "metric",
        "dataset": "openai.audio_transcriptions"
    },
    "tags": [
        "forwarded",
        "openai-audio-transcriptions"
    ]
}
```

**Exported fields**

| Field | Description | Type | Unit |
|---|---|---|---|
| @timestamp | Event timestamp. | date |  |
| data_stream.dataset | Data stream dataset. | constant_keyword |  |
| data_stream.namespace | Data stream namespace. | constant_keyword |  |
| data_stream.type | Data stream type. | constant_keyword |  |
| openai.audio_speeches.characters | Number of characters processed | long |  |
| openai.audio_transcriptions.seconds | Number of seconds processed | long | s |
| openai.base.api_key_id | Identifier for the API key used | keyword |  |
| openai.base.end_time | End timestamp of the usage bucket | date |  |
| openai.base.model | Name of the OpenAI model used | keyword |  |
| openai.base.num_model_requests | Number of requests made to the model | long |  |
| openai.base.project_id | Identifier of the project | keyword |  |
| openai.base.start_time | Start timestamp of the usage bucket | date |  |
| openai.base.usage_object_type | Type of the usage record | keyword |  |
| openai.base.user_id | Identifier of the user | keyword |  |
| openai.completions.input_audio_tokens | Number of audio input tokens used, including cached tokens | long |  |
| openai.completions.input_cached_tokens | Number of text input tokens that has been cached from previous requests. For customers subscribe to scale tier, this includes scale tier tokens | long |  |
| openai.completions.input_tokens | Number of text input tokens used, including cached tokens. For customers subscribe to scale tier, this includes scale tier tokens | long |  |
| openai.completions.output_audio_tokens | Number of audio output tokens used | long |  |
| openai.completions.output_tokens | Number of text output tokens used. For customers subscribe to scale tier, this includes scale tier tokens | long |  |
| openai.embeddings.input_tokens | Number of input tokens used. | long |  |
| openai.images.images | Number of images processed | long |  |
| openai.images.size | Image size (dimension of the generated image) | keyword |  |
| openai.moderations.input_tokens | Number of input tokens used. | long |  |


### Code interpreter sessions

The `code_interpreter_sessions` data stream captures code interpreter sessions usage metrics.

An example event for `code_interpreter_sessions` looks as following:

```json
{
    "openai": {
        "code_interpreter_sessions": {
            "sessions": 16
        },
        "base": {
            "start_time": "2024-09-04T00:00:00.000Z",
            "project_id": "",
            "end_time": "2024-09-05T00:00:00.000Z",
            "usage_object_type": "organization.usage.code_interpreter_sessions.<dummy>"
        }
    },
    "@timestamp": "2024-09-04T00:00:00.000Z",
    "ecs": {
        "version": "9.3.0"
    },
    "data_stream": {
        "namespace": "default",
        "type": "logs",
        "dataset": "openai.code_interpreter_sessions"
    },
    "event": {
        "agent_id_status": "verified",
        "ingested": "2025-01-28T21:56:19Z",
        "created": "2025-01-28T21:56:17.099Z",
        "kind": "metric",
        "dataset": "openai.code_interpreter_sessions"
    },
    "tags": [
        "forwarded",
        "openai-code-interpreter-sessions"
    ]
}
```

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Event timestamp. | date |
| data_stream.dataset | Data stream dataset. | constant_keyword |
| data_stream.namespace | Data stream namespace. | constant_keyword |
| data_stream.type | Data stream type. | constant_keyword |
| openai.base.end_time | End timestamp of the usage bucket | date |
| openai.base.project_id | Identifier of the project | keyword |
| openai.base.start_time | Start timestamp of the usage bucket | date |
| openai.base.usage_object_type | Type of the usage record | keyword |
| openai.code_interpreter_sessions.sessions | Number of code interpreter sessions | long |


### Completions

The `completions` data stream captures completions usage metrics.

An example event for `completions` looks as following:

```json
{
    "openai": {
        "completions": {
            "output_audio_tokens": 0,
            "batch": false,
            "input_audio_tokens": 0,
            "input_cached_tokens": 0,
            "input_tokens": 22,
            "output_tokens": 149
        },
        "base": {
            "start_time": "2025-01-27T00:00:00.000Z",
            "num_model_requests": 1,
            "project_id": "proj_dummy",
            "user_id": "user-dummy",
            "end_time": "2025-01-28T00:00:00.000Z",
            "model": "gpt-4o-mini-2024-07-18",
            "api_key_id": "key_dummy",
            "usage_object_type": "organization.usage.completions.result"
        }
    },
    "@timestamp": "2025-01-27T00:00:00.000Z",
    "ecs": {
        "version": "9.3.0"
    },
    "data_stream": {
        "namespace": "default",
        "type": "logs",
        "dataset": "openai.completions"
    },
    "event": {
        "agent_id_status": "verified",
        "ingested": "2025-01-28T21:56:30Z",
        "created": "2025-01-28T21:56:29.346Z",
        "kind": "metric",
        "dataset": "openai.completions"
    },
    "tags": [
        "forwarded",
        "openai-completions"
    ]
}
```

**Exported fields**

| Field | Description | Type | Unit |
|---|---|---|---|
| @timestamp | Event timestamp. | date |  |
| data_stream.dataset | Data stream dataset. | constant_keyword |  |
| data_stream.namespace | Data stream namespace. | constant_keyword |  |
| data_stream.type | Data stream type. | constant_keyword |  |
| openai.audio_speeches.characters | Number of characters processed | long |  |
| openai.audio_transcriptions.seconds | Number of seconds processed | long | s |
| openai.base.api_key_id | Identifier for the API key used | keyword |  |
| openai.base.end_time | End timestamp of the usage bucket | date |  |
| openai.base.model | Name of the OpenAI model used | keyword |  |
| openai.base.num_model_requests | Number of requests made to the model | long |  |
| openai.base.project_id | Identifier of the project | keyword |  |
| openai.base.start_time | Start timestamp of the usage bucket | date |  |
| openai.base.usage_object_type | Type of the usage record | keyword |  |
| openai.base.usage_tokens | Total tokens consumed by this usage record, normalized across usage data streams for rate-limit headroom calculations. | long |  |
| openai.base.user_id | Identifier of the user | keyword |  |
| openai.completions.batch | Whether the request was processed as a batch | boolean |  |
| openai.completions.input_audio_tokens | Number of audio input tokens used, including cached tokens | long |  |
| openai.completions.input_cached_tokens | Number of text input tokens that has been cached from previous requests. For customers subscribe to scale tier, this includes scale tier tokens | long |  |
| openai.completions.input_tokens | Number of text input tokens used, including cached tokens. For customers subscribe to scale tier, this includes scale tier tokens | long |  |
| openai.completions.output_audio_tokens | Number of audio output tokens used | long |  |
| openai.completions.output_tokens | Number of text output tokens used. For customers subscribe to scale tier, this includes scale tier tokens | long |  |
| openai.embeddings.input_tokens | Number of input tokens used. | long |  |
| openai.images.images | Number of images processed | long |  |
| openai.images.size | Image size (dimension of the generated image) | keyword |  |
| openai.moderations.input_tokens | Number of input tokens used. | long |  |


### Compliance

The `compliance` data stream captures ChatGPT Enterprise compliance logs collected from the OpenAI Compliance Logs Platform, covering application (`APP_LOG`, `APP_AUTH_LOG`), authentication (`AUTH_LOG`), audit (`AUDIT_LOG`), and Codex (`CODEX_LOG`, `CODEX_SECURITY_LOG`) activity.

An example event for `compliance` looks as following:

```json
{
    "@timestamp": "2026-07-09T08:55:12.762Z",
    "agent": {
        "ephemeral_id": "65c6a217-9727-4e28-9c9f-836d21553f2a",
        "id": "73e82da3-7f18-4881-9282-2ba48f9c0ee2",
        "name": "elastic-agent-71633",
        "type": "filebeat",
        "version": "8.18.0"
    },
    "data_stream": {
        "dataset": "openai.compliance",
        "namespace": "73387",
        "type": "logs"
    },
    "ecs": {
        "version": "9.3.0"
    },
    "elastic_agent": {
        "id": "73e82da3-7f18-4881-9282-2ba48f9c0ee2",
        "snapshot": false,
        "version": "8.18.0"
    },
    "event": {
        "action": "login_success",
        "agent_id_status": "verified",
        "category": [
            "authentication"
        ],
        "dataset": "openai.compliance",
        "id": "c97ee69d-41a6-4587-91ee-1ccb1afab002",
        "ingested": "2026-07-15T06:20:16Z",
        "kind": "event",
        "original": "{\"action_data\":{\"action\":\"login_success\",\"auth_provider_name\":\"password\"},\"actor\":{\"type\":\"ACCOUNT_USER\",\"user_email\":\"user@example.org\",\"user_id\":\"user-TUvqhBX7HbQPRgHyEBt5WRcI\"},\"event_id\":\"c97ee69d-41a6-4587-91ee-1ccb1afab002\",\"principal\":{\"id\":\"be545252-ad04-4cfa-9ca5-deca58416151\",\"type\":\"CHATGPT_WORKSPACE\"},\"request_metadata\":{\"client_ip\":\"10.12.43.122\",\"client_user_agent\":\"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36\"},\"timestamp\":\"2026-07-09T08:55:12.762392Z\",\"type\":\"AUTH_LOG\"}",
        "type": [
            "info"
        ]
    },
    "input": {
        "type": "cel"
    },
    "openai": {
        "compliance": {
            "action_data": {
                "auth_provider_name": "password"
            },
            "actor": {
                "type": "ACCOUNT_USER"
            },
            "principal": {
                "type": "CHATGPT_WORKSPACE"
            },
            "type": "AUTH_LOG"
        }
    },
    "organization": {
        "id": "be545252-ad04-4cfa-9ca5-deca58416151"
    },
    "related": {
        "ip": [
            "10.12.43.122"
        ],
        "user": [
            "user@example.org",
            "user-TUvqhBX7HbQPRgHyEBt5WRcI"
        ]
    },
    "source": {
        "ip": "10.12.43.122"
    },
    "tags": [
        "preserve_original_event",
        "forwarded",
        "openai-compliance"
    ],
    "user": {
        "domain": "example.org",
        "email": "user@example.org",
        "id": "user-TUvqhBX7HbQPRgHyEBt5WRcI"
    },
    "user_agent": {
        "device": {
            "name": "Mac"
        },
        "name": "Other",
        "original": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36",
        "os": {
            "full": "Mac OS X 10.15.7",
            "name": "Mac OS X",
            "version": "10.15.7"
        }
    }
}
```

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Date/time when the event originated. This is the date/time extracted from the event, typically representing when the event was generated by the source. If the event source has no original timestamp, this value is typically populated by the first time the event was received by the pipeline. Required field for all events. | date |
| data_stream.dataset | The field can contain anything that makes sense to signify the source of the data. Examples include `nginx.access`, `prometheus`, `endpoint` etc. For data streams that otherwise fit, but that do not have dataset set we use the value "generic" for the dataset value. `event.dataset` should have the same value as `data_stream.dataset`. Beyond the Elasticsearch data stream naming criteria noted above, the `dataset` value has additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.namespace | A user defined namespace. Namespaces are useful to allow grouping of data. Many users already organize their indices this way, and the data stream naming scheme now provides this best practice as a default. Many users will populate this field with `default`. If no value is used, it falls back to `default`. Beyond the Elasticsearch index naming criteria noted above, `namespace` value has the additional restrictions:   \* Must not contain `-`   \* No longer than 100 characters | constant_keyword |
| data_stream.type | An overarching type for the data stream. Currently allowed values are "logs" and "metrics". We expect to also add "traces" and "synthetics" in the near future. | constant_keyword |
| event.dataset | Name of the dataset. If an event source publishes more than one type of log or events (e.g. access log, error log), the dataset is used to specify which one the event comes from. It's recommended but not required to start the dataset name with the module name, followed by a dot, then the dataset name. | constant_keyword |
| event.module | Name of the module this data is coming from. If your monitoring agent supports the concept of modules or plugins to process events of a given source (e.g. Apache logs), `event.module` should contain the name of this module. | constant_keyword |
| input.type | Type of filebeat input. | keyword |
| observer.product | The product name of the observer. | constant_keyword |
| observer.vendor | Vendor name of the observer. | constant_keyword |
| openai.compliance.action | Sub-action name. AUDIT_LOG: the audit action (e.g. WORKSPACE_TOGGLE_FEATURE). APP_AUTH_LOG: link/unlink. Allowed values: APP_AUTH_LOG: link, unlink | AUDIT_LOG: 150 action names (see AUDIT_LOG sheet). | keyword |
| openai.compliance.action_data.action | Auth sub-action (e.g. login_success, signup_success, password_reset_initiated). | keyword |
| openai.compliance.action_data.added_user_id | Auth user ID added (GROUP_EDIT). | keyword |
| openai.compliance.action_data.affected_connector_ids | Connector IDs whose workspace permissions were affected (APP_PUBLISH / WORKSPACE_TOGGLE_FEATURE). | keyword |
| openai.compliance.action_data.after | Pagination cursor for list actions (RFC3339 timestamp). | date |
| openai.compliance.action_data.after_string | Pagination cursor for list actions (Opaque token). | keyword |
| openai.compliance.action_data.agent_id | Workspace agent identifier. | keyword |
| openai.compliance.action_data.alert_recipients.email | Recipient email for usage alerts (SUBSCRIPTION_SET_USAGE_ALERTS). | keyword |
| openai.compliance.action_data.alert_recipients.type | Recipient type; email. Allowed values: email. | keyword |
| openai.compliance.action_data.alert_thresholds.credits | Credit threshold amount. | long |
| openai.compliance.action_data.alert_thresholds.type | Threshold type; absolute. Allowed values: absolute. | keyword |
| openai.compliance.action_data.allow_all_in_workspace | Whether the app is visible to the whole workspace (APP_UPDATE_ACCESS_POLICY). | boolean |
| openai.compliance.action_data.allowed_ips | CIDR entries in the IP allowlist (WORKSPACE_SET_\*_IP_ALLOWLIST). | keyword |
| openai.compliance.action_data.allowed_user_id | Auth user ID granted access (APP_ALLOW_USER). | keyword |
| openai.compliance.action_data.already_exists | True if a shared conversation already existed (CONVERSATION_SHARE). | boolean |
| openai.compliance.action_data.app_id | App / connector identifier. | keyword |
| openai.compliance.action_data.app_name | App / connector friendly name. | keyword |
| openai.compliance.action_data.auth_provider_name | Authentication provider used (e.g. password, google, saml). | keyword |
| openai.compliance.action_data.auth_user_id | Auth user ID of a memory owner (DELETE_MEMORY). | keyword |
| openai.compliance.action_data.automation_id | Automation identifier. | keyword |
| openai.compliance.action_data.before | Pagination cursor upper bound for list actions (RFC3339 timestamp). | date |
| openai.compliance.action_data.before_string | Pagination cursor upper bound for list actions (Opaque token). | keyword |
| openai.compliance.action_data.changed_fields | List of fields changed by the request. | keyword |
| openai.compliance.action_data.changed_fields_by_connector | Map of connector ID -\> changed fields (keys are connector IDs). | flattened |
| openai.compliance.action_data.classification | FedRAMP banner classification string (WORKSPACE_SET_FEDRAMP_BANNER). | keyword |
| openai.compliance.action_data.client_id | Codex remote-control client identifier. | keyword |
| openai.compliance.action_data.codex_environment_id | Codex environment identifier (GET_CODE_ENVIRONMENT). | keyword |
| openai.compliance.action_data.context_plugin_id | Plugin whose UI initiated the action. | keyword |
| openai.compliance.action_data.conversation_id | Conversation identifier. | keyword |
| openai.compliance.action_data.created_at | Creation time (epoch seconds). | date |
| openai.compliance.action_data.created_by_user_id | Auth user ID that created the service account (nullable). | keyword |
| openai.compliance.action_data.credential_id | Credential / personal access token identifier. | keyword |
| openai.compliance.action_data.credential_name | User-provided credential name. | keyword |
| openai.compliance.action_data.credential_type | Credential type; personal_access_token. Allowed values: personal_access_token. | keyword |
| openai.compliance.action_data.cursor | Pagination cursor (list actions). | keyword |
| openai.compliance.action_data.deactivated_at | Service account deletion time (epoch seconds) or null. | date |
| openai.compliance.action_data.deleted_user_id | Auth user ID deleted (USER_DELETE). | keyword |
| openai.compliance.action_data.disallowed_user_id | Auth user ID whose access was revoked (APP_DISALLOW_USER). | keyword |
| openai.compliance.action_data.domain_id | Managed domain identifier. | keyword |
| openai.compliance.action_data.domains | Allowed action domains for a GPT (may be null to clear). | keyword |
| openai.compliance.action_data.email_address | Single email address (invite actions). | keyword |
| openai.compliance.action_data.email_addresses | List of email addresses (invite / group actions). | keyword |
| openai.compliance.action_data.enabled | Enabled / active state after the change. | boolean |
| openai.compliance.action_data.environment_id | Environment identifier. | keyword |
| openai.compliance.action_data.event_type | Compliance event category: filter list (array) or resolved category (download, string). Allowed values: Compliance event categories (APP_LOG, APP_AUTH_LOG, AUDIT_LOG, AUTH_LOG, CODEX_LOG, CODEX_SECURITY_LOG, ...). | keyword |
| openai.compliance.action_data.expires_at | Credential expiration time (epoch seconds) or null. | date |
| openai.compliance.action_data.feature | Feature / flag identifier toggled (WORKSPACE_TOGGLE_FEATURE / WORKSPACE_SET_SHARING_PERMISSION). | keyword |
| openai.compliance.action_data.file_format | File output mode: url or id. Allowed values: url, id. | keyword |
| openai.compliance.action_data.file_id | File identifier. | keyword |
| openai.compliance.action_data.file_name | File name (FILE_DOWNLOAD). | keyword |
| openai.compliance.action_data.gpt_id | GPT identifier. | keyword |
| openai.compliance.action_data.group_id | Group identifier. | keyword |
| openai.compliance.action_data.hostname | Domain hostname added (DOMAIN_CREATE). | keyword |
| openai.compliance.action_data.id | Resource identifier (GPT / project) for the action. | keyword |
| openai.compliance.action_data.include_summary | Whether recording summary sections are included (DOWNLOAD_RECORDING_TRANSCRIPT). | boolean |
| openai.compliance.action_data.installation_policy | Plugin installation policy after the update. | keyword |
| openai.compliance.action_data.installation_policy_before | Plugin installation policy before the update. | keyword |
| openai.compliance.action_data.installed_by_default_role_ids | Role IDs the plugin is installed for by default (after). | keyword |
| openai.compliance.action_data.installed_by_default_role_ids_before | Role IDs the plugin was installed for by default (before). | keyword |
| openai.compliance.action_data.knowledge_app_type | App backend / type provisioned (APP_CREATE). | keyword |
| openai.compliance.action_data.limit | Requested page size (list actions). | long |
| openai.compliance.action_data.log_file_id | Compliance log file identifier (download actions). | keyword |
| openai.compliance.action_data.logged_out_user_id | Auth user ID logged out by admin (USER_LOGGED_OUT_BY_ADMIN). | keyword |
| openai.compliance.action_data.markdown | FedRAMP banner markdown body. | keyword |
| openai.compliance.action_data.memory_context_id | Memory context identifier (DELETE_MEMORY). | keyword |
| openai.compliance.action_data.memory_id | Memory identifier. | keyword |
| openai.compliance.action_data.message_id | Message identifier (rating actions). | keyword |
| openai.compliance.action_data.mime_type | MIME type of the file (FILE_DOWNLOAD). | keyword |
| openai.compliance.action_data.name | Resource / service-account / credential name. | keyword |
| openai.compliance.action_data.new_name | New / updated name. | keyword |
| openai.compliance.action_data.new_owner_email | Email of the new GPT owner (CUSTOM_GPT_CHANGE_OWNER). | keyword |
| openai.compliance.action_data.new_role | Role assigned to the user (USER_ROLE_UPDATED). | keyword |
| openai.compliance.action_data.new_workspace_permissions_by_connector | Map of connector ID -\> permission snapshot after the change. | flattened |
| openai.compliance.action_data.old_name | Previous name before the change. | keyword |
| openai.compliance.action_data.order | Sort order (list directory actions). | keyword |
| openai.compliance.action_data.overage_limit_credits | Credit overage limit: integer or the string "unlimited". Allowed values are integer, or "unlimited". | keyword |
| openai.compliance.action_data.owner_id | Owner filter / owner user ID (list agents etc.). | keyword |
| openai.compliance.action_data.owner_user_id | Auth user ID of the credential owner. | keyword |
| openai.compliance.action_data.plugin_id | Plugin identifier. | keyword |
| openai.compliance.action_data.previous_enabled | Service account active state before the update. | boolean |
| openai.compliance.action_data.previous_name | Service account name before the update (nullable). | keyword |
| openai.compliance.action_data.previous_role | Previous role for a share target (nullable) (SKILL_SHARE). | keyword |
| openai.compliance.action_data.previous_workspace_permissions_by_connector | Map of connector ID -\> permission snapshot before the change. | flattened |
| openai.compliance.action_data.principal_id | Principal (user or group) identifier for a share target. | keyword |
| openai.compliance.action_data.principal_type | Principal type: user or group. Allowed values: user, group. | keyword |
| openai.compliance.action_data.project_id | Project identifier. | keyword |
| openai.compliance.action_data.public_display_name | External display name (WORKSPACE_SET_IS_DISCOVERABLE). | keyword |
| openai.compliance.action_data.rating | Message rating: thumbs_up or thumbs_down. Allowed values: thumbs_up, thumbs_down. | keyword |
| openai.compliance.action_data.recording_id | Recording identifier. | keyword |
| openai.compliance.action_data.remote_thread_id | Codex remote-control thread identifier. | keyword |
| openai.compliance.action_data.removed_user_id | Auth user ID removed (GROUP_REMOVE_USER). | keyword |
| openai.compliance.action_data.requested_access_removals.recipient_type | Recipient type requested for removal (PROJECT_UPDATE_SHARING). Allowed values: private, email, user, group, workspace, workspace_link, link, marketplace. | keyword |
| openai.compliance.action_data.requested_access_removals.user_id | User ID requested for removal. | keyword |
| openai.compliance.action_data.requested_access_updates.capabilities | Capabilities requested for the recipient. | keyword |
| openai.compliance.action_data.requested_access_updates.recipient_type | Recipient type requested to add/update. Allowed values: private, email, user, group, workspace, workspace_link, link, marketplace. | keyword |
| openai.compliance.action_data.requested_access_updates.user_id | User ID requested to add/update. | keyword |
| openai.compliance.action_data.revoked_at | Revocation time (epoch seconds) or null. | date |
| openai.compliance.action_data.revoked_unified_sessions | Unified session IDs revoked (SESSION_REVOKE). | keyword |
| openai.compliance.action_data.role | Role granted / removed (manager or user; or invite role). Allowed values: manager, user (service-account/skill share targets). | keyword |
| openai.compliance.action_data.room_id | Project chat room identifier. | keyword |
| openai.compliance.action_data.scope_id | Connector scope identifier (DELETE_PROJECT_CONNECTOR_SCOPE). | keyword |
| openai.compliance.action_data.scopes | Product-access scopes on the credential (sorted, deduplicated). | keyword |
| openai.compliance.action_data.service_account_id | Service account auth user ID. | keyword |
| openai.compliance.action_data.session_id | Session identifier (SESSION_REVOKE). | keyword |
| openai.compliance.action_data.shared_conversation_id | Shared conversation identifier (CONVERSATION_SHARE / view). | keyword |
| openai.compliance.action_data.shared_to.capabilities | Capabilities granted to the recipient segment. | keyword |
| openai.compliance.action_data.shared_to.group_id | Group ID recipient (sharing). | keyword |
| openai.compliance.action_data.shared_to.recipient_type | Recipient type (UPDATE_SHARING): private/email/user/group/workspace/workspace_link/link/marketplace. Allowed values: private, email, user, group, workspace, workspace_link, link, marketplace. | keyword |
| openai.compliance.action_data.shared_to.type | Recipient segment type (CHANGE_VISIBILITY): private/email/user/group/workspace/workspace_link/link/marketplace. Allowed values: private, email, user, group, workspace, workspace_link, link, marketplace. | keyword |
| openai.compliance.action_data.shared_to.user_email | User email recipient when the segment type is user (sharing). | keyword |
| openai.compliance.action_data.shared_to.user_id | User ID recipient when the segment type is user (sharing). | keyword |
| openai.compliance.action_data.sign_in_endpoint | SAML sign-in endpoint URL / SCIM connection identifier. | keyword |
| openai.compliance.action_data.since_timestamp | Lower-bound timestamp filter (list actions). | date |
| openai.compliance.action_data.skill_id | Skill identifier. | keyword |
| openai.compliance.action_data.skill_name | Skill name. | keyword |
| openai.compliance.action_data.source_surface | Admin surface that initiated the action (admin_apps or admin_plugins). Allowed values: admin_apps, admin_plugins. | keyword |
| openai.compliance.action_data.state_changed | Whether the successful request performed the initial revoke transition. | boolean |
| openai.compliance.action_data.system_instruction | Workspace default system prompt (WORKSPACE_SET_SYSTEM_INSTRUCTION). | keyword |
| openai.compliance.action_data.task_id | Codex task identifier. | keyword |
| openai.compliance.action_data.textdoc_id | Canvas text document identifier. | keyword |
| openai.compliance.action_data.third_party_gizmo_id | Third-party GPT identifier. | keyword |
| openai.compliance.action_data.total_requested | Count of join requests approved (INVITE_ACCEPT_ALL). | long |
| openai.compliance.action_data.use_workspace_name_for_discovery | Whether the workspace name is reused for discovery. | boolean |
| openai.compliance.action_data.user_id | Auth user ID targeted by the action. | keyword |
| openai.compliance.action_data.value | Generic value payload (boolean/string) for toggle/set actions. | keyword |
| openai.compliance.action_data.value_bool | Boolean form of action_data.value (populated when the changed value is a boolean). | boolean |
| openai.compliance.action_data.workspace_access_state | Skill workspace access state: restricted / discoverable / enabled. Allowed values: restricted, discoverable, enabled. | keyword |
| openai.compliance.action_data.workspace_id | Workspace identifier. | keyword |
| openai.compliance.action_data.workspace_policy | Workspace policy document or identifier (WORKSPACE_SET_POLICY). | keyword |
| openai.compliance.action_privilege | Privilege level for the action: ADMIN or STANDARD_USER. Allowed values: ADMIN, STANDARD_USER. | keyword |
| openai.compliance.action_result | Outcome of the action: SUCCESS, BLOCKED, or ERROR. Allowed values: SUCCESS, BLOCKED, ERROR. | keyword |
| openai.compliance.actor.redacted_id | Redacted identifier of the API key (present when actor.type = API_KEY). | keyword |
| openai.compliance.actor.type | Type of actor that performed the action (e.g. ACCOUNT_USER, API_KEY). Allowed values: ACCOUNT_USER, API_KEY. | keyword |
| openai.compliance.actor.user_email | Email of the acting user (present when the actor is a user). | keyword |
| openai.compliance.actor.user_id | Auth user ID of the acting user (present when the actor is a user). | keyword |
| openai.compliance.app_id | Identifier of the connected app / connector. | keyword |
| openai.compliance.app_name | Friendly name of the app / connector. | keyword |
| openai.compliance.app_type | App backend type (e.g. SERVICE). Allowed values: SERVICE, OPEN_API, MCP, FIRST_PARTY_ECOSYSTEM, UNKNOWN. | keyword |
| openai.compliance.client_id | Codex client identifier associated with the event. Allowed values: CODEX_CLI, CODEX_WEB, CODEX_IDE_VSCODE, CODEX_GITHUB_ACTION, CODEX_SDK_TS, CODEX_SERVICE_EXEC, CODEX_UNKNOWN_DEFAULT. | keyword |
| openai.compliance.conversation_id | Conversation in which the app was invoked. | keyword |
| openai.compliance.event_details.access_token_id | Codex access token identifier. | keyword |
| openai.compliance.event_details.access_token_name | User-provided access token name. | keyword |
| openai.compliance.event_details.arguments.jql | Arguments passed to an app/MCP call (shape varies; example is a Jira JQL query). | keyword |
| openai.compliance.event_details.call_id | Identifier of the MCP/tool call. | keyword |
| openai.compliance.event_details.created_by_user_id | Auth user ID that created the resource. | keyword |
| openai.compliance.event_details.current_settings.default_model | Default model after the change. | keyword |
| openai.compliance.event_details.current_settings.network_access | Network-access policy after the change. | keyword |
| openai.compliance.event_details.decision | Approval decision for a suggested tool call (e.g. approved, denied). | keyword |
| openai.compliance.event_details.detail_type | Discriminator identifying the specific Codex event-details schema. | keyword |
| openai.compliance.event_details.elicitation_type | Type of MCP elicitation (e.g. oauth_consent). | keyword |
| openai.compliance.event_details.environment_fields.agent_settings.model | Model configured for the environment's agent. | keyword |
| openai.compliance.event_details.environment_fields.cache_settings.enabled | Whether environment caching is enabled. | boolean |
| openai.compliance.event_details.environment_fields.description | Environment description. | keyword |
| openai.compliance.event_details.environment_fields.env_var_keys | Names (keys only) of configured environment variables. | keyword |
| openai.compliance.event_details.environment_fields.label | Environment label / name. | keyword |
| openai.compliance.event_details.environment_fields.permissions.network_access | Environment network-access permission. | keyword |
| openai.compliance.event_details.environment_fields.repos | Repositories attached to the environment. | keyword |
| openai.compliance.event_details.environment_id | Codex environment identifier. | keyword |
| openai.compliance.event_details.error_code | Error code when the action failed (null on success). | keyword |
| openai.compliance.event_details.error_message | Error message when the action failed (null on success). | keyword |
| openai.compliance.event_details.expires_at | Expiration time, e.g. for access tokens. Format assumed RFC3339 (verify). | date |
| openai.compliance.event_details.finding_id | Security finding identifier. | keyword |
| openai.compliance.event_details.marketplace_name | Marketplace source for a plugin. | keyword |
| openai.compliance.event_details.model | Model used for the turn / prompt. | keyword |
| openai.compliance.event_details.name | Tool / function name. | keyword |
| openai.compliance.event_details.phase | Turn phase (e.g. prompt). | keyword |
| openai.compliance.event_details.plugin_creator_account_user_id | Account user ID that created the plugin. | keyword |
| openai.compliance.event_details.plugin_display_name | Plugin display name. | keyword |
| openai.compliance.event_details.plugin_id | Plugin identifier. | keyword |
| openai.compliance.event_details.plugin_name | Plugin (technical) name. | keyword |
| openai.compliance.event_details.plugin_release_id | Plugin release identifier. | keyword |
| openai.compliance.event_details.plugin_scope | Plugin scope (e.g. WORKSPACE). Allowed values: GLOBAL, WORKSPACE, USER. | keyword |
| openai.compliance.event_details.plugin_version | Plugin version. | keyword |
| openai.compliance.event_details.pr_url | Pull request URL associated with the finding / scan. | keyword |
| openai.compliance.event_details.previous_settings.default_model | Default model before the change. | keyword |
| openai.compliance.event_details.previous_settings.network_access | Network-access policy before the change. | keyword |
| openai.compliance.event_details.prompt_text | Prompt text submitted by the user (sensitive content). | keyword |
| openai.compliance.event_details.reasoning_effort | Reasoning effort setting (e.g. low/medium/high). | keyword |
| openai.compliance.event_details.response_text | Model response text (sensitive content). | keyword |
| openai.compliance.event_details.result_preview | Preview of a tool/call result. | keyword |
| openai.compliance.event_details.scan_configuration_fields.environment_id | Environment the scan configuration targets. | keyword |
| openai.compliance.event_details.scan_configuration_fields.lookback_days | Scan lookback window in days. | long |
| openai.compliance.event_details.scan_configuration_fields.notification_rules_created_count | Count of notification rules created. | long |
| openai.compliance.event_details.scan_configuration_fields.notification_rules_deleted_count | Count of notification rules deleted. | long |
| openai.compliance.event_details.scan_configuration_fields.notification_rules_updated_count | Count of notification rules updated. | long |
| openai.compliance.event_details.scan_configuration_fields.owner_id | Owner (auth user ID) of the scan configuration. | keyword |
| openai.compliance.event_details.scan_configuration_fields.repo_id | Repository identifier scanned. | keyword |
| openai.compliance.event_details.scan_configuration_fields.repo_url | Repository URL scanned. | keyword |
| openai.compliance.event_details.scan_configuration_fields.scan_type | Type of scan (e.g. secrets). Value set not fully specified. | keyword |
| openai.compliance.event_details.scan_configuration_fields.state | Scan configuration state (e.g. active). Value set not fully specified. | keyword |
| openai.compliance.event_details.scan_configuration_fields.workspace_id | Workspace of the scan configuration. | keyword |
| openai.compliance.event_details.scan_configuration_id | Scan configuration identifier. | keyword |
| openai.compliance.event_details.service_tier | Service tier used for the request. | keyword |
| openai.compliance.event_details.session_id | Codex session identifier. | keyword |
| openai.compliance.event_details.status | Outcome status (e.g. success, error). | keyword |
| openai.compliance.event_details.token_usage.cached_input_tokens | Cached input tokens used. | long |
| openai.compliance.event_details.token_usage.input_tokens | Input tokens used. | long |
| openai.compliance.event_details.token_usage.output_tokens | Output tokens generated. | long |
| openai.compliance.event_details.token_usage.reasoning_output_tokens | Reasoning output tokens generated. | long |
| openai.compliance.event_details.tool_call_id | Identifier of the tool call. | keyword |
| openai.compliance.event_details.tool_input | Input passed to the tool (shape varies). | keyword |
| openai.compliance.event_details.tool_meta.connector_id | Connector identifier associated with the tool. | keyword |
| openai.compliance.event_details.tool_meta.resource_id | Resource identifier associated with the tool. | keyword |
| openai.compliance.event_details.tool_name | Name of the tool invoked. | keyword |
| openai.compliance.event_details.tool_type | Tool type (e.g. filesystem). | keyword |
| openai.compliance.event_details.turn_id | Identifier of the conversation turn. | keyword |
| openai.compliance.event_details.updated_fields.assignee_user_email | Email of the finding assignee after update. | keyword |
| openai.compliance.event_details.updated_fields.criticality | Finding criticality after update (e.g. low/medium/high). Value set not fully specified. | keyword |
| openai.compliance.event_details.updated_fields.criticality_reason | Reason for the criticality. | keyword |
| openai.compliance.event_details.updated_fields.criticality_reason_updated | Whether criticality_reason changed in this update. | boolean |
| openai.compliance.event_details.updated_fields.environment_id | Environment identifier (updated field). | keyword |
| openai.compliance.event_details.updated_fields.lookback_days | Lookback days (updated field). | long |
| openai.compliance.event_details.updated_fields.notification_rules_created_count | Notification rules created count (updated field). | long |
| openai.compliance.event_details.updated_fields.notification_rules_deleted_count | Notification rules deleted count (updated field). | long |
| openai.compliance.event_details.updated_fields.notification_rules_updated_count | Notification rules updated count (updated field). | long |
| openai.compliance.event_details.updated_fields.owner_id | Owner id (updated field). | keyword |
| openai.compliance.event_details.updated_fields.project_overview_length | Length of the project overview text (updated). | long |
| openai.compliance.event_details.updated_fields.project_overview_updated | Whether the project overview changed. | boolean |
| openai.compliance.event_details.updated_fields.resolution_reason | Reason recorded when resolving the finding. | keyword |
| openai.compliance.event_details.updated_fields.resolution_reason_updated | Whether resolution_reason changed. | boolean |
| openai.compliance.event_details.updated_fields.state | State (updated field). | keyword |
| openai.compliance.event_details.updated_fields.status | Finding status after update (e.g. wontfix). Value set not fully specified. | keyword |
| openai.compliance.event_details.url | URL associated with the event (e.g. OAuth consent URL). | keyword |
| openai.compliance.event_id | Globally unique event identifier. Use for de-duplication (delivery is at-least-once). | keyword |
| openai.compliance.event_type | Codex event type/category discriminator (top-level). | keyword |
| openai.compliance.input.query | Query/input sent to the app (request log). Exact shape not fully specified. | keyword |
| openai.compliance.link_id | Identifier of the app connection (link) that was linked/unlinked. | keyword |
| openai.compliance.log_type | APP_LOG sub-type: request or response. Allowed values: request, response. | keyword |
| openai.compliance.output.display_title | Title of a returned result item (response log). | keyword |
| openai.compliance.output.display_url | URL of a returned result item (response log). | keyword |
| openai.compliance.principal.id | Identifier of the ChatGPT workspace (or organization) the event belongs to. | keyword |
| openai.compliance.principal.type | Principal that owns the event (e.g. CHATGPT_WORKSPACE). Allowed values: CHATGPT_WORKSPACE, API_PLATFORM_ORG. | keyword |
| openai.compliance.request_metadata.client_ip | Client IP address captured at request time. | ip |
| openai.compliance.request_metadata.client_ip_details.asn | Illustrative sub-field of client_ip_details (real shape unspecified). | keyword |
| openai.compliance.request_metadata.client_ip_details.country | Illustrative sub-field of client_ip_details (real shape unspecified). | keyword |
| openai.compliance.request_metadata.client_ja3 | JA3 TLS fingerprint of the client (may be empty). | keyword |
| openai.compliance.request_metadata.client_ja4 | JA4 TLS fingerprint of the client (may be empty). | keyword |
| openai.compliance.request_metadata.client_user_agent | Client User-Agent string captured at request time. | keyword |
| openai.compliance.request_metadata.destination_hostname | Destination hostname for the request (nullable). | keyword |
| openai.compliance.timestamp | Event time (RFC3339 / ISO-8601, UTC). | date |
| openai.compliance.type | Top-level event category (APP_LOG, APP_AUTH_LOG, AUDIT_LOG, AUTH_LOG, CODEX_LOG, CODEX_SECURITY_LOG). Allowed values: APP_LOG, APP_AUTH_LOG, AUDIT_LOG, AUTH_LOG, CODEX_LOG, CODEX_SECURITY_LOG (full platform also emits other categories). | keyword |
| openai.compliance.workspace_id | Workspace identifier associated with the event. | keyword |


### Embeddings

The `embeddings` data stream captures embeddings usage metrics.

An example event for `embeddings` looks as following:

```json
{
    "openai": {
        "embeddings": {
            "input_tokens": 16
        },
        "base": {
            "start_time": "2024-09-04T00:00:00.000Z",
            "num_model_requests": 2,
            "project_id": "",
            "user_id": "user-dummy",
            "end_time": "2024-09-05T00:00:00.000Z",
            "model": "text-embedding-ada-002-v2",
            "api_key_id": "key_dummy",
            "usage_object_type": "organization.usage.embeddings.result"
        }
    },
    "@timestamp": "2024-09-04T00:00:00.000Z",
    "ecs": {
        "version": "9.3.0"
    },
    "data_stream": {
        "namespace": "default",
        "type": "logs",
        "dataset": "openai.embeddings"
    },
    "event": {
        "agent_id_status": "verified",
        "ingested": "2025-01-28T21:56:19Z",
        "created": "2025-01-28T21:56:17.149Z",
        "kind": "metric",
        "dataset": "openai.embeddings"
    },
    "tags": [
        "forwarded",
        "openai-embeddings"
    ]
}
```

**Exported fields**

| Field | Description | Type | Unit |
|---|---|---|---|
| @timestamp | Event timestamp. | date |  |
| data_stream.dataset | Data stream dataset. | constant_keyword |  |
| data_stream.namespace | Data stream namespace. | constant_keyword |  |
| data_stream.type | Data stream type. | constant_keyword |  |
| openai.audio_speeches.characters | Number of characters processed | long |  |
| openai.audio_transcriptions.seconds | Number of seconds processed | long | s |
| openai.base.api_key_id | Identifier for the API key used | keyword |  |
| openai.base.end_time | End timestamp of the usage bucket | date |  |
| openai.base.model | Name of the OpenAI model used | keyword |  |
| openai.base.num_model_requests | Number of requests made to the model | long |  |
| openai.base.project_id | Identifier of the project | keyword |  |
| openai.base.start_time | Start timestamp of the usage bucket | date |  |
| openai.base.usage_object_type | Type of the usage record | keyword |  |
| openai.base.usage_tokens | Total tokens consumed by this usage record, normalized across usage data streams for rate-limit headroom calculations. | long |  |
| openai.base.user_id | Identifier of the user | keyword |  |
| openai.completions.input_audio_tokens | Number of audio input tokens used, including cached tokens | long |  |
| openai.completions.input_cached_tokens | Number of text input tokens that has been cached from previous requests. For customers subscribe to scale tier, this includes scale tier tokens | long |  |
| openai.completions.input_tokens | Number of text input tokens used, including cached tokens. For customers subscribe to scale tier, this includes scale tier tokens | long |  |
| openai.completions.output_audio_tokens | Number of audio output tokens used | long |  |
| openai.completions.output_tokens | Number of text output tokens used. For customers subscribe to scale tier, this includes scale tier tokens | long |  |
| openai.embeddings.input_tokens | Number of input tokens used. | long |  |
| openai.images.images | Number of images processed | long |  |
| openai.images.size | Image size (dimension of the generated image) | keyword |  |
| openai.moderations.input_tokens | Number of input tokens used. | long |  |


### Images

The `images` data stream captures images usage metrics.

An example event for `images` looks as following:

```json
{
    "openai": {
        "images": {
            "images": 1,
            "size": "1024x1024",
            "source": "image.generation"
        },
        "base": {
            "start_time": "2024-09-04T00:00:00.000Z",
            "num_model_requests": 1,
            "project_id": "",
            "user_id": "user-dummy",
            "end_time": "2024-09-05T00:00:00.000Z",
            "model": "dall-e-3",
            "api_key_id": "key_dummy",
            "usage_object_type": "organization.usage.images.result"
        }
    },
    "@timestamp": "2024-09-04T00:00:00.000Z",
    "ecs": {
        "version": "9.3.0"
    },
    "data_stream": {
        "namespace": "default",
        "type": "logs",
        "dataset": "openai.images"
    },
    "event": {
        "agent_id_status": "verified",
        "ingested": "2025-01-28T21:56:19Z",
        "created": "2025-01-28T21:56:17.758Z",
        "kind": "metric",
        "dataset": "openai.images"
    },
    "tags": [
        "forwarded",
        "openai-images"
    ]
}
```

**Exported fields**

| Field | Description | Type | Unit |
|---|---|---|---|
| @timestamp | Event timestamp. | date |  |
| data_stream.dataset | Data stream dataset. | constant_keyword |  |
| data_stream.namespace | Data stream namespace. | constant_keyword |  |
| data_stream.type | Data stream type. | constant_keyword |  |
| openai.audio_speeches.characters | Number of characters processed | long |  |
| openai.audio_transcriptions.seconds | Number of seconds processed | long | s |
| openai.base.api_key_id | Identifier for the API key used | keyword |  |
| openai.base.end_time | End timestamp of the usage bucket | date |  |
| openai.base.model | Name of the OpenAI model used | keyword |  |
| openai.base.num_model_requests | Number of requests made to the model | long |  |
| openai.base.project_id | Identifier of the project | keyword |  |
| openai.base.start_time | Start timestamp of the usage bucket | date |  |
| openai.base.usage_images | Number of images processed by a usage record, normalized across usage data streams. | long |  |
| openai.base.usage_object_type | Type of the usage record | keyword |  |
| openai.base.user_id | Identifier of the user | keyword |  |
| openai.completions.input_audio_tokens | Number of audio input tokens used, including cached tokens | long |  |
| openai.completions.input_cached_tokens | Number of text input tokens that has been cached from previous requests. For customers subscribe to scale tier, this includes scale tier tokens | long |  |
| openai.completions.input_tokens | Number of text input tokens used, including cached tokens. For customers subscribe to scale tier, this includes scale tier tokens | long |  |
| openai.completions.output_audio_tokens | Number of audio output tokens used | long |  |
| openai.completions.output_tokens | Number of text output tokens used. For customers subscribe to scale tier, this includes scale tier tokens | long |  |
| openai.embeddings.input_tokens | Number of input tokens used. | long |  |
| openai.images.images | Number of images processed | long |  |
| openai.images.size | Image size (dimension of the generated image) | keyword |  |
| openai.images.source | Source of the grouped usage result, possible values are `image.generation`, `image.edit`, `image.variation` | keyword |  |
| openai.moderations.input_tokens | Number of input tokens used. | long |  |


### Moderations

The `moderations` data stream captures moderations usage metrics.

An example event for `moderations` looks as following:

```json
{
    "openai": {
        "moderations": {
            "input_tokens": 16
        },
        "base": {
            "start_time": "2024-09-04T00:00:00.000Z",
            "num_model_requests": 2,
            "project_id": "",
            "user_id": "user-dummy",
            "end_time": "2024-09-05T00:00:00.000Z",
            "model": "text-moderation:2023-10-25",
            "api_key_id": "key_dummy",
            "usage_object_type": "organization.usage.moderations.result"
        }
    },
    "@timestamp": "2024-09-04T00:00:00.000Z",
    "ecs": {
        "version": "9.3.0"
    },
    "data_stream": {
        "namespace": "default",
        "type": "logs",
        "dataset": "openai.moderations"
    },
    "event": {
        "agent_id_status": "verified",
        "ingested": "2025-01-28T21:56:19Z",
        "created": "2025-01-28T21:56:17.099Z",
        "kind": "metric",
        "dataset": "openai.moderations"
    },
    "tags": [
        "forwarded",
        "openai-moderations"
    ]
}
```

**Exported fields**

| Field | Description | Type | Unit |
|---|---|---|---|
| @timestamp | Event timestamp. | date |  |
| data_stream.dataset | Data stream dataset. | constant_keyword |  |
| data_stream.namespace | Data stream namespace. | constant_keyword |  |
| data_stream.type | Data stream type. | constant_keyword |  |
| openai.audio_speeches.characters | Number of characters processed | long |  |
| openai.audio_transcriptions.seconds | Number of seconds processed | long | s |
| openai.base.api_key_id | Identifier for the API key used | keyword |  |
| openai.base.end_time | End timestamp of the usage bucket | date |  |
| openai.base.model | Name of the OpenAI model used | keyword |  |
| openai.base.num_model_requests | Number of requests made to the model | long |  |
| openai.base.project_id | Identifier of the project | keyword |  |
| openai.base.start_time | Start timestamp of the usage bucket | date |  |
| openai.base.usage_object_type | Type of the usage record | keyword |  |
| openai.base.usage_tokens | Total tokens consumed by this usage record, normalized across usage data streams for rate-limit headroom calculations. | long |  |
| openai.base.user_id | Identifier of the user | keyword |  |
| openai.completions.input_audio_tokens | Number of audio input tokens used, including cached tokens | long |  |
| openai.completions.input_cached_tokens | Number of text input tokens that has been cached from previous requests. For customers subscribe to scale tier, this includes scale tier tokens | long |  |
| openai.completions.input_tokens | Number of text input tokens used, including cached tokens. For customers subscribe to scale tier, this includes scale tier tokens | long |  |
| openai.completions.output_audio_tokens | Number of audio output tokens used | long |  |
| openai.completions.output_tokens | Number of text output tokens used. For customers subscribe to scale tier, this includes scale tier tokens | long |  |
| openai.embeddings.input_tokens | Number of input tokens used. | long |  |
| openai.images.images | Number of images processed | long |  |
| openai.images.size | Image size (dimension of the generated image) | keyword |  |
| openai.moderations.input_tokens | Number of input tokens used. | long |  |


### Rate limits

The `rate_limits` data stream captures per-project, per-model rate limits.

An example event for `rate_limits` looks as following:

```json
{
    "@timestamp": "2026-05-26T06:40:21.000Z",
    "openai": {
        "rate_limits": {
            "id": "rl_gpt4o_mini",
            "object": "project.rate_limit",
            "model": "gpt-4o-mini-2024-07-18",
            "project_id": "proj_test_a",
            "project_name": "Test Project A",
            "project_status": "active",
            "max_requests_per_1_minute": 500,
            "max_tokens_per_1_minute": 200000,
            "max_images_per_1_minute": 50000,
            "max_requests_per_1_day": 10000,
            "max_tokens_per_1_day": 9000000,
            "batch_1_day_max_input_tokens": 2000000,
            "collected_at": "2026-05-26T06:40:21Z"
        }
    },
    "ecs": {
        "version": "9.3.0"
    },
    "data_stream": {
        "namespace": "default",
        "type": "logs",
        "dataset": "openai.rate_limits"
    },
    "event": {
        "agent_id_status": "verified",
        "ingested": "2026-05-26T06:40:30Z",
        "created": "2026-05-26T06:40:21.000Z",
        "kind": "metric",
        "dataset": "openai.rate_limits"
    },
    "tags": [
        "forwarded",
        "openai-rate-limits"
    ]
}
```

**Exported fields**

| Field | Description | Type |
|---|---|---|
| @timestamp | Event timestamp. | date |
| data_stream.dataset | Data stream dataset. | constant_keyword |
| data_stream.namespace | Data stream namespace. | constant_keyword |
| data_stream.type | Data stream type. | constant_keyword |
| event.dataset | Event dataset. | constant_keyword |
| input.type | Input type. | keyword |
| openai.base.model | Name of the OpenAI model used. | keyword |
| openai.base.num_model_requests | Number of requests made to the model. | long |
| openai.base.project_id | Identifier of the project. | keyword |
| openai.base.usage_images | Number of images processed by a usage record, normalized across usage data streams. | long |
| openai.base.usage_tokens | Total tokens consumed by a usage record, normalized across usage data streams. | long |
| openai.rate_limits.batch_1_day_max_input_tokens | Maximum batch input tokens per day, when provided by OpenAI. | long |
| openai.rate_limits.collected_at | Time when the rate-limit document was collected. | date |
| openai.rate_limits.id | OpenAI rate limit identifier. | keyword |
| openai.rate_limits.max_audio_megabytes_per_1_minute | Maximum audio megabytes per minute, when provided by OpenAI. | long |
| openai.rate_limits.max_images_per_1_minute | Maximum images per minute, when provided by OpenAI. | long |
| openai.rate_limits.max_requests_per_1_day | Maximum requests per day, when provided by OpenAI. | long |
| openai.rate_limits.max_requests_per_1_minute | Maximum requests per minute. | long |
| openai.rate_limits.max_tokens_per_1_day | Maximum tokens per day, when provided by OpenAI. | long |
| openai.rate_limits.max_tokens_per_1_minute | Maximum tokens per minute. | long |
| openai.rate_limits.model | OpenAI model name this limit applies to. | keyword |
| openai.rate_limits.object | OpenAI object type. Expected value is project.rate_limit. | keyword |
| openai.rate_limits.project_id | OpenAI project ID (injected from projects list). | keyword |
| openai.rate_limits.project_name | OpenAI project name (injected from projects list). | keyword |
| openai.rate_limits.project_status | OpenAI project status (injected from projects list). | keyword |


### Vector stores

The `vector_stores` data stream captures vector stores usage metrics.

An example event for `vector_stores` looks as following:

```json
{
    "openai": {
        "vector_stores": {
            "usage_bytes": 16
        },
        "base": {
            "start_time": "2024-09-04T00:00:00.000Z",
            "project_id": "",
            "end_time": "2024-09-05T00:00:00.000Z",
            "usage_object_type": "organization.usage.vector_stores.<dummy>"
        }
    },
    "@timestamp": "2024-09-04T00:00:00.000Z",
    "ecs": {
        "version": "9.3.0"
    },
    "data_stream": {
        "namespace": "default",
        "type": "logs",
        "dataset": "openai.vector_stores"
    },
    "event": {
        "agent_id_status": "verified",
        "ingested": "2025-01-28T21:56:19Z",
        "created": "2025-01-28T21:56:17.099Z",
        "kind": "metric",
        "dataset": "openai.vector_stores"
    },
    "tags": [
        "forwarded",
        "openai-vector-stores"
    ]
}
```

**Exported fields**

| Field | Description | Type | Unit |
|---|---|---|---|
| @timestamp | Event timestamp. | date |  |
| data_stream.dataset | Data stream dataset. | constant_keyword |  |
| data_stream.namespace | Data stream namespace. | constant_keyword |  |
| data_stream.type | Data stream type. | constant_keyword |  |
| openai.base.end_time | End timestamp of the usage bucket | date |  |
| openai.base.project_id | Identifier of the project | keyword |  |
| openai.base.start_time | Start timestamp of the usage bucket | date |  |
| openai.base.usage_object_type | Type of the usage record | keyword |  |
| openai.vector_stores.usage_bytes | Vector stores usage in bytes | long | byte |

