# Approach 1: Tool Definitions

Three HTTP tools to create in HappyRobot (**Tools > Creating Tools**).
Each section below contains the tool name, description, method, URL,
headers, and parameter schema.

---

## Tool 1: `get_raw_shipments`

**Description for the LLM:**
> Fetch raw shipment timelines from the AI Control Tower. Returns
> shipments with their 8 process steps, each containing target_min,
> actual_min, timestamps, and cutoff_time. Use this to investigate
> delivery performance, find late shipments, and identify root causes.
> Always use limit to keep responses manageable.

**Method:** `GET`

**URL:** `https://plan.supply-science.com/api/v1/monitoring/tools/raw-shipments`

**Headers:**

| Header | Value |
|---|---|
| `X-Api-Key` | `{{env.CT_API_KEY}}` |

**Parameters:**

| Name | Type | Required | Description |
|---|---|---|---|
| `destination_city` | string | no | One of: Shanghai, Beijing, Seoul, Tokyo, Dubai, Singapore, Bangkok, Riyadh |
| `status` | string | no | Filter: created, transmitted, picked, loaded, in_transit_airport, in_flight, in_clearance, out_for_delivery, delivered, late |
| `window_hours` | integer | no | Only shipments ordered within the last N hours |
| `limit` | integer | no | Max shipments to return (1-500). Default 100. Use 5-20 for analysis. |
| `team` | string | no | Warehouse, Road_Transport, Air_Freight, or IT |
| `step_name` | string | no | Filter by step: transmission, pickpack, loading, warehouse_airport, flight, clearance, truck_prep, airport_store |
| `min_duration_min` | number | no | At least one step with actual_min >= this value |
| `offset` | integer | no | Skip first N results for pagination |
| `order_by` | string | no | Sort by: order_time (default) or last_updated |
| `order_dir` | string | no | asc or desc (default) |

**Example call the LLM would make:**
```json
{
  "destination_city": "Shanghai",
  "status": "late",
  "window_hours": 48,
  "limit": 10
}
```

---

## Tool 2: `get_recent_events`

**Description for the LLM:**
> Fetch recent status transitions across all shipments. Use this to scan
> for anomalies: spikes in late transitions, clusters of flights landing
> together, or cities where nothing has moved. Good as a first step to
> understand what is happening right now.

**Method:** `GET`

**URL:** `https://plan.supply-science.com/api/v1/monitoring/tools/recent-events`

**Headers:**

| Header | Value |
|---|---|
| `X-Api-Key` | `{{env.CT_API_KEY}}` |

**Parameters:**

| Name | Type | Required | Description |
|---|---|---|---|
| `minutes` | integer | no | Look-back window in minutes. Default 15. Use 240 for a 4-hour scan. |
| `limit` | integer | no | Max events to return (1-2000). Default 500. |

**Example call the LLM would make:**
```json
{
  "minutes": 240,
  "limit": 500
}
```

---

## Tool 3: `post_flash_report`

**Description for the LLM:**
> Post a flash report to the Control Tower dashboard after completing an
> investigation. Only call this when the user explicitly confirms they
> want a report filed. Include specific shipment references, numbers,
> and the root cause attribution you computed.

**Method:** `POST`

**URL:** `https://plan.supply-science.com/api/v1/monitoring/reports`

**Headers:**

| Header | Value |
|---|---|
| `X-Happyrobot-Token` | `{{env.CT_HAPPYROBOT_TOKEN}}` |
| `Content-Type` | `application/json` |

**Body parameters:**

| Name | Type | Required | Max length | Description |
|---|---|---|---|---|
| `title` | string | yes | 200 chars | Short headline summarizing the finding |
| `severity` | string | yes | — | `critical`, `warning`, or `info` |
| `summary` | string | yes | 2000 chars | One-paragraph summary with key numbers |
| `body` | string | yes | 20 KB | Full analysis in markdown format |
| `persona_name` | string | no | 100 chars | E.g. "Local Logistics Manager — Shanghai" |
| `metrics_json` | string | no | 20 KB | Valid JSON string with key metrics |
| `tools_called` | string | no | 10 KB | Valid JSON array of tools and args used |
| `source` | string | no | 100 chars | Always set to `"happyrobot"` |
| `trigger` | string | no | — | `manual` (user-initiated) or `scheduled` |

**Example call the LLM would make:**
```json
{
  "title": "Shanghai: Air Freight root cause, Road Transport cascade victim",
  "severity": "warning",
  "summary": "9 of 10 late Shanghai shipments caused by flight delays...",
  "body": "## Evidence\n\n...",
  "persona_name": "Local Logistics Manager — Shanghai",
  "source": "happyrobot",
  "trigger": "manual"
}
```

---

## Notes

- **Token substitution**: `{{env.CT_API_KEY}}` and
  `{{env.CT_HAPPYROBOT_TOKEN}}` reference HappyRobot environment
  variables set in Settings. Check HappyRobot's exact syntax for env
  var interpolation in tool headers — it may be `@env.CT_API_KEY` or
  similar depending on the version.

- **Rate limiting**: The Control Tower API is shared infrastructure.
  Tell the LLM to use `limit=5` to `limit=20` for analysis calls, not
  `limit=500`.

- **Write token status**: `X-Happyrobot-Token` returns 401 as of
  2026-04-16. Use `X-Openclaw-Token` as header name with the openclaw
  token as fallback if needed.
