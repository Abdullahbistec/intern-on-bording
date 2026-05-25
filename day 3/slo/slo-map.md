# BookSwap — SLI/SLO Map

## 1. NFR inventory

| # | NFR | User-visible behaviour |
|---:|-----|------------------------|
| 1 | Catalogue search: 99% of requests under 800 ms over rolling 28 days, even at 10× normal RPS | Members can search books quickly during normal usage and the Sunday spike. |
| 2 | Listing creation: 99.9% success rate | Members can list books reliably without losing work. |
| 3 | Failed listing attempts must be retryable without duplicate listings | Members can retry after timeout/5xx safely. |
| 4 | Search results remain useful when cache is cold or unavailable | Members still get search results using SQL fallback. |
| 5 | Every endpoint except `/health` requires valid JWT | Anonymous users cannot access private APIs. |
| 6 | Tokens expire within 1 hour | Stolen tokens have limited lifetime. |
| 7 | Member must never see another member’s loan history or address | Privacy and object-level authorization are protected. |
| 8 | Complete outage of listings endpoint pages on-call within 3 minutes | Ops knows quickly when listing creation is down. |
| 9 | Authentication failures logged with request ID and member ID/anonymous marker | Security incidents can be investigated. |
| 10 | Every loan creation/return logged with request ID and member ID | Disputes have an audit trail. |
| 11 | Ops confirms system health in under 5 minutes | Ops can quickly answer “is BookSwap healthy?” |
| 12 | Photos up to 5 MB JPEG/PNG; never stored in DB | Members upload photos without bloating database. |
| 13 | In-app notifications within 2 seconds; digest best-effort | Owners receive borrow notifications quickly. |
| 14 | Addresses/phones never returned in API responses | Private contact data is not leaked. |

## 2. SLI / SLO table

| # | SLI definition | Measurement source | SLO target | Window | Error budget |
|---:|----------------|--------------------|------------|--------|--------------|
| 1 | `% of GET /books requests with success=true and duration < 800 ms` | App Insights `requests` | >= 99% | Rolling 28 days | 1% bad requests |
| 2 | `% of POST /books returning 201 or safe idempotent replay response` | App Insights `requests` | >= 99.9% | Rolling 28 days | 0.1% failed attempts |
| 3 | `% of repeated POST /books with same Idempotency-Key returning existing record, not duplicate` | Custom metric + SQL duplicate check | >= 99.99% | Rolling 28 days | 0.01% unsafe retries |
| 4 | `% of synthetic Redis-down searches returning valid SQL fallback response` | Synthetic test + dependency telemetry | >= 99% | Weekly test + rolling 28 days | 1% failed fallback |
| 5 | `% of non-/health unauthenticated requests returning 401/403` | App Insights requests + auth logs | 100% | Rolling 28 days | 0 bypasses |
| 6 | `% of accepted JWTs where exp-iat <= 3600 seconds` | Auth middleware custom metric | 100% | Rolling 28 days | 0 overlong tokens |
| 7 | Count of successful cross-member loan/address access attempts | Security audit logs + authz tests | 0 | Rolling 28 days | 0 incidents |
| 8 | Time from POST /books outage to on-call page | Azure Monitor alert timestamps | <= 3 minutes | Each outage | 0 missed pages |
| 9 | `% of auth failures logged with requestId and memberIdOrAnonymous` | Azure Monitor Logs `traces` | >= 99.5% | Rolling 28 days | 0.5% incomplete logs |
| 10 | `% of loan create/return events logged with requestId, memberId, loanId` | Azure Monitor Logs `traces` | >= 99.9% | Rolling 28 days | 0.1% incomplete audit |
| 11 | Time for dashboard to show API, SQL, Redis, Service Bus, Blob health | Ops drill + dashboard telemetry | <= 5 minutes | Monthly game day | 0 failed drills |
| 12 | Count of photo binaries stored in SQL | DB scan + Blob logs | 0 | Rolling 28 days | 0 violations |
| 13 | `% of in-app notifications created within 2 seconds after SQL commit` | Service Bus + SQL timestamps | >= 99% | Rolling 28 days | 1% late notifications |
| 14 | Count of API/log/queue payloads containing raw address or phone | DLP query + contract tests | 0 | Rolling 28 days | 0 leaks |

## 3. Example queries

### Search latency SLI

```kusto
requests
| where timestamp > ago(28d)
| where name == "GET /books"
| summarize good=countif(success == true and duration < 800), total=count()
| extend sli = 100.0 * good / total
```

### Listing creation success SLI

```kusto
requests
| where timestamp > ago(28d)
| where name == "POST /books"
| summarize good=countif(resultCode == "201" or resultCode == "200"), total=count()
| extend sli = 100.0 * good / total
```

### Auth failure logging completeness

```kusto
traces
| where timestamp > ago(28d)
| where customDimensions.event == "auth.failed"
| summarize complete=countif(isnotempty(customDimensions.requestId)
  and isnotempty(customDimensions.memberIdOrAnonymous)), total=count()
| extend sli = 100.0 * complete / total
```

## 4. Error budget policy

When any SLO consumes more than 75% of its 28-day error budget, the Engineering Lead reviews it in daily stand-up and the team avoids risky changes on that path.

When an SLO fully exhausts its error budget, the team stops non-critical feature work for that affected path and spends the next sprint capacity on reliability or security fixes. Examples: add indexes, tune autoscale, fix idempotency, reduce slow dependencies, or improve authorization tests.

The decision owner is the Engineering Lead, with Product Owner and On-call Engineer input.

## 5. Out of budget right now

The SLO I would bet we cannot meet today is **Catalogue search 99% under 800 ms at 10× traffic**, because the Day 2 design has SQL indexes and Redis cache, but no load test has proven performance across 200 buildings and cold-cache conditions.
