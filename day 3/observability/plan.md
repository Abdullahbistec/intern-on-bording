# BookSwap — Observability Plan

## Setup

### Logs
- Tool: Azure Monitor Logs and Application Insights traces.
- Format: structured JSON.
- Retention: 30 days hot; export security/audit logs for long-term storage if required.
- Required fields: timestamp, level, service, route, requestId, memberIdOrAnonymous, event, statusCode, durationMs.
- Redaction: do not log address, phone, Authorization header, JWT, raw email body, or full request body.

### Metrics
- Tool: Azure Application Insights metrics and Azure Monitor metrics.
- Track request count, latency, success rate, dependency failures, cache hit/fallback rate, queue depth, auth failures, and audit completeness.

### Traces
- Tool: Application Insights distributed tracing.
- Default sample rate: 10% successful requests.
- Always sample failures, slow requests over 800 ms, `POST /books`, borrow request creation, and loan return.
- Propagate `traceparent` and request ID across API, SQL, Redis, Blob, and Service Bus.

## Signal catalogue

| # | Signal type | Source | What it answers | Sample query / metric name |
|---:|-------------|--------|-----------------|----------------------------|
| 1 | Metric | App Insights requests | Search latency p95 | `requests | where name=="GET /books" | summarize percentile(duration,95) by bin(timestamp,1m)` |
| 2 | Metric | App Insights requests | Listing creation success rate | `requests | where name=="POST /books" | summarize successRate=100.0*countif(resultCode=="201")/count()` |
| 3 | Metric | Custom metric | Idempotent retry safety | `customMetrics | where name=="idempotency.replay.success"` |
| 4 | Metric | Azure Service Bus | Queue depth/backlog | `ActiveMessages`, `DeadletteredMessages`, `OldestMessageAge` |
| 5 | Metric | Azure Cache for Redis | Cache health | `cache.hit.rate`, `cache.fallback.count` |
| 6 | Metric | Azure SQL | SQL bottleneck | SQL CPU/DTU %, deadlocks, dependency p95 |
| 7 | Log | Azure Monitor Logs | Auth failures | `traces | where customDimensions.event=="auth.failed"` |
| 8 | Log | Azure Monitor Logs | Loan audit completeness | `traces | where customDimensions.event in ("loan.created","loan.returned")` |
| 9 | Log | Azure Monitor Logs | PII leakage check | `traces | where message has_any ("phone","address","Authorization")` |
| 10 | Log | Azure Front Door/WAF logs | Blocked/rate-limited traffic | WAF logs by rule ID/client IP |
| 11 | Trace | App Insights | Slow search breakdown | Trace view for `GET /books` showing API, Redis, SQL spans |
| 12 | Trace | App Insights | Listing creation path | Trace for `POST /books` showing SQL insert and Service Bus enqueue |
| 13 | Trace | App Insights | Redis fallback behaviour | Trace with Redis failure followed by SQL dependency |
| 14 | Trace | App Insights | Blob/photo handling delay | Trace for photo metadata/upload path |

## Results Summary

| Metric | Target | Achieved |
|--------|--------|----------|
| SLOs covered by an alert | 100% | 100% planned |
| Alerts with a clear runbook link | 100% | 100% planned |
| Dashboards for ops | 1 health, 1 business | 2 planned |

## Alert proposal

| Alert | Condition | Severity | Notification | Runbook |
|-------|-----------|----------|--------------|---------|
| Search SLO burn | `GET /books` bad rate > 1% over 5 min | Sev2 | Pager + Teams | `reliability/runbook.md#failure-3-sunday-tabloid-spike--10-sustained-traffic` |
| Listing endpoint outage | `POST /books` synthetic test fails for 3 min | Sev1 | Pager + SMS + Teams | `reliability/runbook.md#failure-1-azure-sql-primary-unavailable-for-5-minutes` |
| Listing success drop | `POST /books` success rate < 99% over 5 min | Sev2 | Pager + Teams | `reliability/runbook.md#failure-1-azure-sql-primary-unavailable-for-5-minutes` |
| Redis down/fallback high | Redis failures > 10% for 2 min and search p95 > 800 ms | Sev3/Sev2 | Teams; Pager if SLO burns | `reliability/runbook.md#failure-2-azure-cache-for-redis-is-down` |
| SQL failures | SQL dependency failure rate > 5% for 2 min | Sev1 | Pager + Teams | `reliability/runbook.md#failure-1-azure-sql-primary-unavailable-for-5-minutes` |
| Auth bypass risk | Non-`/health` unauthenticated request returns non-401/403 | Sev1 | Pager + Security channel | `security/review.md` |
| Auth failure spike | Auth failures > 100/min or > 5× baseline | Sev2 | Teams + Security channel | `security/review.md` |
| Loan audit missing | Loan audit completeness < 99.9% over 15 min | Sev2 | Teams + on-call | `security/review.md` |
| Service Bus backlog | Active messages > 10,000 or oldest message age > 10 min | Sev3 | Teams | `reliability/runbook.md#failure-3-sunday-tabloid-spike--10-sustained-traffic` |
| PII leak detected | Logs contain address/phone/JWT pattern | Sev1 | Pager + Security channel | `security/review.md` |

## Sample alert queries

```kusto
requests
| where timestamp > ago(5m)
| where name == "GET /books"
| summarize bad=countif(success == false or duration >= 800), total=count()
| extend badRate = 100.0 * bad / total
| where badRate > 1
```

```kusto
traces
| where timestamp > ago(15m)
| where customDimensions.event in ("loan.created", "loan.returned")
| summarize complete=countif(isnotempty(customDimensions.requestId)
  and isnotempty(customDimensions.memberId)
  and isnotempty(customDimensions.loanId)), total=count()
| extend completeness = 100.0 * complete / total
| where completeness < 99.9
```

## What we are deliberately NOT alerting on

1. Every single 404. Track on dashboard, do not page unless spike is far above baseline.
2. Individual Redis cache misses. Alert only when fallback burns search SLO.
3. Single email digest failure. Digest is best-effort; alert on repeated failures or DLQ growth.
4. Low-traffic noisy percentages. Percentage alerts need minimum volume.
5. Normal 400/422 validation errors. Track for product quality, not on-call pages.
