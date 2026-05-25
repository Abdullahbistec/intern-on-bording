# BookSwap — Reliability Runbook v0.1

## Failure 1: Azure SQL primary unavailable for 5 minutes

### What the user sees
Search may become slow or fail. Book listing, borrow request, and loan update actions may return `503 Service Unavailable`. Warm cached search results may still work for a short time, but new writes cannot be safely completed without SQL.

### Detection
- Azure SQL availability/resource health alert.
- App Insights dependency target `Azure SQL` failure rate > 5% for 2 minutes.
- `POST /books` 5xx rate > 2% for 3 minutes.
- Synthetic `POST /books` availability test fails.

```kusto
dependencies
| where timestamp > ago(5m)
| where target has "sql" or type == "SQL"
| summarize failures=countif(success == false), total=count()
| extend failureRate = 100.0 * failures / total
| where failureRate > 5
```

### Mitigation in design
- SQL read timeout: 2 seconds.
- SQL write transaction timeout: 3 seconds.
- Retry transient SQL errors only: 3 tries with exponential backoff 200 ms, 500 ms, 1000 ms plus jitter.
- Circuit breaker opens when 50% of SQL calls fail over 30 seconds with at least 20 calls.
- Circuit remains open 60 seconds, then half-open allows 5 trial calls.
- `POST /books` requires `Idempotency-Key`, so retries do not create duplicate listings.
- Cached `GET /books` responses may be served up to 5 minutes stale with `stale: true`.
- Writes are not queued as source of truth; they fail safely when SQL is unavailable.

### Manual response
- Paged: backend on-call engineer.
- Notify: Teams `#bookswap-incidents`.
- Actions:
  1. Check Azure SQL resource health.
  2. Check App Insights SQL dependency failures and slow queries.
  3. Check whether failure is Azure outage, connection pool exhaustion, or bad deploy.
  4. Roll back latest deploy if dependency failures started immediately after release.
  5. Avoid scaling App Service blindly if SQL is already overloaded.

### Post-incident actions
- Review slow queries and add indexes.
- Review SQL tier and connection pool settings.
- Confirm idempotency prevented duplicate listings.
- Add SQL-down game-day test.
- Add dashboard panel for SQL dependency failure rate and p95.

---

## Failure 2: Azure Cache for Redis is down

### What the user sees
Search still works through Azure SQL fallback, but may be slower. Recently added books may take longer. No data is lost because Redis is not the source of truth.

### Detection
- Redis dependency failure rate > 10% for 2 minutes.
- `GET /books` p95 > 800 ms for 5 minutes.
- Custom metric `cache.fallback.count` increases sharply.
- Azure Redis resource health alert.

```kusto
dependencies
| where timestamp > ago(5m)
| where target has "redis" or type == "Redis"
| summarize failures=countif(success == false), total=count()
| extend failureRate = 100.0 * failures / total
| where failureRate > 10
```

### Mitigation in design
- Redis timeout: 150 ms.
- Redis retry: 1 retry after 100 ms only.
- If Redis read fails, API immediately falls back to Azure SQL.
- If Redis write fails, API returns SQL result and logs `cache.write.failed`.
- Redis circuit breaker opens after 25 failures in 60 seconds.
- Circuit breaker open duration: 2 minutes.
- TTL: search cache 60 seconds, recent books 5 minutes, book detail 60 seconds.

### Manual response
- Sev3 Teams alert unless search SLO burn happens; then page on-call.
- Check Redis resource health.
- Watch SQL CPU because SQL may receive more read traffic.
- Temporarily scale Azure SQL or App Service if fallback load is high.
- Disable cache writes by feature flag if they are noisy.
- Warm common search cache after recovery.

### Post-incident actions
- Confirm no request failed only because Redis failed.
- Tune cache timeout/circuit breaker.
- Add dashboard panel for cache hit rate, miss rate, and fallback count.
- Consider SQL read replica if Redis-down fallback overloads SQL.

---

## Failure 3: Sunday tabloid spike — 10× sustained traffic

### What the user sees
Most members should still search and list books. Some users may experience slower searches. Non-critical emails may be delayed. Abusive or excessive traffic may receive `429 Too Many Requests`.

### Detection
- Azure Front Door request count > 10× baseline for 5 minutes.
- App Service CPU > 70% for 10 minutes or HTTP queue length rising.
- `GET /books` p95 > 800 ms for 5 minutes.
- Azure SQL CPU/DTU > 80% for 10 minutes.
- Service Bus active messages > 10,000 or oldest message age > 10 minutes.
- 429 rate > 5% for 5 minutes.

### Mitigation in design
- Azure Front Door with WAF and rate limiting.
- App Service autoscale:
  - Minimum 3 instances.
  - Scale out +2 instances when CPU > 60% for 5 minutes or request queue length > 100.
  - Maximum 12 instances initially.
- API is stateless, so any instance can serve any request.
- Redis cache used for common searches; cache warm job for top searches by building.
- SQL indexes on building_id + title/author/isbn/condition/availability.
- Service Bus absorbs notification and digest work.
- Rate limits:
  - `GET /books`: 120 requests/min/member.
  - `POST /books`: 20 requests/hour/member.
  - Borrow requests: 30 requests/hour/member.
- Retry policy:
  - Reads retry 2 times with 200 ms and 500 ms backoff.
  - Writes retry only with `Idempotency-Key`.

### Manual response
- Page on-call if search p95 > 800 ms for 5 minutes or listing success < 99% for 5 minutes.
- Open health dashboard.
- Identify bottleneck: App Service, SQL, Redis, Service Bus, or Front Door.
- Temporarily raise max App Service instances from 12 to 20 if API is bottleneck.
- If SQL is bottleneck, reduce expensive search filters using feature flag and increase SQL tier if approved.
- Pause weekly digest processing so user-facing notifications have priority.
- Increase Front Door rate-limit strictness if abusive traffic is detected.

### Post-incident actions
- Compare actual RPS to forecast.
- Review autoscale timing.
- Add cache warmup before known media events.
- Run 10× and 20× load test.
- Review throttled users and false positives.
