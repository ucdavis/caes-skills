---
name: elasticcloud-logs
description: Query and interpret logs from the CAES Elastic Cloud deployment
---

# Elastic Cloud Logs

Query the CAES Elastic Cloud deployment and interpret the results in plain language.

## Prerequisites

- 1Password CLI (`op`) installed: `brew install 1password-cli`
- 1Password desktop app integration enabled: **1Password → Settings → Developer → Integrate with 1Password CLI**
- Local agent shells may need `OP_BIOMETRIC_UNLOCK_ENABLED=true` so `op run` uses desktop-app approval. Do not require `OP_SERVICE_ACCOUNT_TOKEN` for local agent use.
- `elasticcloud-env` at the repo root (committed — `op://` references only, no real secrets)

## Index Patterns

| Apps | Index pattern | Severity field | App/service field |
|------|--------------|----------------|-------------------|
| ASP.NET / Serilog (Sloth, Anlab, Purchasing, Payments, etc.) | `*` | `level.keyword` | `fields.Application.keyword` |
| OpenTelemetry / APM (Walter, FacultyDirectory, Harvest, Readable, etc.) | `logs-apm*` | `log.level` | `service.name` |
| OpenTelemetry HTTP traces (Walter 5xx response checks) | `traces-apm*,logs-apm*` | `http.response.status_code` | `service.name` |

**Walter specifically:** filter `service.name: walter`. Distinguish environments with `service.environment: production` or `service.environment: test`. For HTTP 500/5xx questions, query `traces-apm*,logs-apm*` with `http.response.status_code` first. If that returns no hits, query `logs-apm*` for `log.level: Error` and clearly describe those as app-level errors rather than confirmed HTTP 5xx responses.

## Steps

### 1. Clarify the query

If the request is ambiguous (no app, no time range, no severity), ask one focused question before proceeding. If clear, proceed immediately.

### 2. Determine the time range

Default to the last 24 hours unless the user specifies otherwise:

```json
{ "range": { "@timestamp": { "gte": "now-24h", "lte": "now" } } }
```

### 3. Construct the query

Build a `POST /<index>/_search` JSON body with a `bool` query and `filter` clauses. Start with `size: 20`.

**HTTP 5xx query for Walter production:**

```json
{
  "size": 20,
  "track_total_hits": true,
  "query": {
    "bool": {
      "filter": [
        { "range": { "@timestamp": { "gte": "now-24h", "lte": "now" } } },
        { "term": { "service.name": "walter" } },
        { "term": { "service.environment": "production" } },
        { "range": { "http.response.status_code": { "gte": 500, "lt": 600 } } }
      ]
    }
  },
  "sort": [{ "@timestamp": { "order": "desc" } }],
  "aggs": {
    "by_status": { "terms": { "field": "http.response.status_code", "size": 10 } },
    "by_transaction": { "terms": { "field": "transaction.name", "size": 20 } },
    "by_url_path": { "terms": { "field": "url.path", "size": 20 } }
  }
}
```

Run this against `traces-apm*,logs-apm*`.

**Error query for Walter production:**

Use this fallback when HTTP 5xx status queries return no hits or when the user asks for application errors:

```json
{
  "size": 20,
  "track_total_hits": true,
  "query": {
    "bool": {
      "filter": [
        { "range": { "@timestamp": { "gte": "now-24h", "lte": "now" } } },
        { "term": { "service.name": "walter" } },
        { "term": { "service.environment": "production" } },
        { "term": { "log.level": "Error" } }
      ]
    }
  },
  "sort": [{ "@timestamp": { "order": "desc" } }],
  "aggs": {
    "by_dataset": { "terms": { "field": "event.dataset", "size": 10 } },
    "by_exception_type": { "terms": { "field": "error.exception.type", "size": 20 } },
    "over_time": { "date_histogram": { "field": "@timestamp", "fixed_interval": "1h" } }
  }
}
```

Run this against `logs-apm*`.

**Error query for Serilog apps:**

```json
{
  "size": 20,
  "query": {
    "bool": {
      "filter": [
        { "range": { "@timestamp": { "gte": "now-24h", "lte": "now" } } },
        { "term": { "fields.Application.keyword": "Sloth.Api" } },
        { "term": { "level.keyword": "Error" } }
      ]
    }
  },
  "sort": [{ "@timestamp": { "order": "desc" } }]
}
```

### 4. Execute

Use `op run` with the committed env file. 1Password will prompt for desktop-app approval to resolve the credentials:

```bash
OP_BIOMETRIC_UNLOCK_ENABLED=true op run --env-file=elasticcloud-env -- sh -c \
  'curl -s -u "$ELASTIC_USER:$ELASTIC_PASS" "$ELASTIC_URL/<index>/_search" \
    -H "Content-Type: application/json" -d @-' <<'EOF'
<query JSON>
EOF
```

### 5. Interpret the results

Summarize in plain language — counts, top patterns, notable messages, anomalies. Never dump raw JSON at the user.

If raw hits won't give a useful answer (too many results, or user wants a breakdown), switch to an aggregation query. Use your judgment on when this is appropriate:

- **`terms` on `log.level` / `service.name`** — categorical breakdown
- **`date_histogram` on `@timestamp`** — volume over time

Example aggregation for Walter error breakdown:

```json
{
  "size": 0,
  "query": {
    "bool": {
      "filter": [
        { "range": { "@timestamp": { "gte": "now-24h", "lte": "now" } } },
        { "term": { "service.name": "walter" } },
        { "term": { "service.environment": "production" } }
      ]
    }
  },
  "aggs": {
    "by_level": { "terms": { "field": "log.level", "size": 10 } }
  }
}
```

## Error Handling

| Condition | What to do |
|-----------|------------|
| `op` not found | Tell the user to run `brew install 1password-cli` and enable the desktop app integration in 1Password Settings → Developer |
| `op run` says `No accounts configured` despite desktop integration being enabled | Retry with `OP_BIOMETRIC_UNLOCK_ENABLED=true` before `op run`; this forces the desktop-app approval path for shells that do not inherit the integration setting |
| `op` auth error / Touch ID prompt fails | Tell the user to ensure they're signed in to the 1Password desktop app and the CLI integration is enabled |
| curl returns non-200 | Show the status code and `error` field from the response; suggest checking credentials or index pattern |
| Empty results | Say so clearly; suggest broadening the time range, checking the index pattern, or verifying the app/service name |
