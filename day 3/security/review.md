# BookSwap — Security Review

## Scope
This review covers the BookSwap REST API and Day 3 quality requirements. Focus areas: authentication, authorization, injection, secrets, transport, rate limiting, and PII handling.

## Summary
Highest risks:
1. Broken Object Level Authorization (BOLA) on book loan history and borrow request decision endpoints.
2. PII leakage through API responses, logs, traces, or queue messages.
3. Missing rate limits on write endpoints during the Sunday traffic spike.

## Review table

| Category | Question | Finding | Severity | Mitigation |
|----------|----------|---------|----------|------------|
| Authn | Is every non-public endpoint protected by JWT? | All endpoints except `/health` must require `Authorization: Bearer <JWT>`. Implementation may accidentally expose a route. | H | Global auth middleware with `/health` allowlist. Test every endpoint without JWT and expect `401`. |
| Authz | Does every `/{id}` endpoint check object ownership? | BOLA risk: `GET /books/{bookId}/loans` could expose another owner's borrower history if only JWT validity is checked. | H | Load resource and verify caller owns or is allowed to access it. Return `403` if not. |
| Injection | Are all DB queries parameterised? | `search`, `condition`, `availability`, `page`, and `pageSize` could be abused if concatenated into SQL. | H | Parameterised SQL, allowlisted filters, max lengths, no dynamic SQL from raw input. |
| Secrets | Where are connection strings stored? | `.env` files or source control could leak secrets. | M | Azure Key Vault + managed identity. Secret scanning in CI. No production `.env`. |
| Transport | Is TLS enforced at Front Door? | Tokens can leak if HTTP or weak TLS is allowed. | H | HTTPS-only at Azure Front Door/App Service, HSTS, TLS 1.2+. |
| Rate limit | Are auth and write endpoints rate-limited? | `POST /books` and borrow request endpoints can be spammed. | M | Azure Front Door WAF and app per-member limits. Return `429` with `Retry-After`. |
| PII | What PII appears in responses, logs, or queues? | Address/phone/JWT may leak through full request logging or queue payloads. | H | Structured redacted logging. Queue messages contain IDs only. Never log Authorization header. |

## Finding 1 — Broken Object Level Authorization scenario

A malicious authenticated member tries:

```http
GET /books/book_1001/loans
Authorization: Bearer token-for-member-who-does-not-own-book
```

If the API checks only JWT validity, the attacker could see another owner’s borrower history.

**Severity:** High  
**Mitigation:** Check the caller is the owner of `book_1001` before returning loan history. Otherwise return `403 Forbidden`.

## Finding 2 — PII leak via logs or telemetry

If the API logs full request/response bodies or Service Bus messages, address, phone, email, JWT, or private messages may appear in Azure Monitor Logs.

**Severity:** High  
**Mitigation:** Use structured logging with redaction. Approved fields: request ID, member ID, route, status, duration, book ID, loan ID. Forbidden fields: address, phone, Authorization header, JWT, raw email body, full request body.

## Finding 3 — Missing rate limit on sensitive endpoint

```http
POST /books
POST /books/book_1001/borrow-requests
```

A bot or malicious member can spam writes, increasing SQL load and notification queue depth.

**Severity:** Medium  
**Mitigation:** Front Door WAF rate limits and application per-member limits.

## Finding 4 — Injection risk in search

```http
GET /books?search=' OR 1=1 --
```

**Severity:** High  
**Mitigation:** Parameterised queries and allowlisted filters.

## Finding 5 — JWT token lifetime

API must reject tokens with lifetime greater than 1 hour.

**Severity:** Medium  
**Mitigation:** Validate issuer, audience, signature, expiry, and `exp - iat <= 3600`.

## Finding 6 — ZAP baseline report discussion

The ZAP baseline report is attached at `security/zap-baseline-report.html`. Expected findings for a local mock API often include missing security headers like CSP or `X-Content-Type-Options`. These are usually lower risk for a JSON-only mock API than BOLA or PII leaks, but should still be handled at Azure Front Door/App Service level.

## Defence in depth

BookSwap should use multiple layers of protection. Azure Front Door provides TLS, WAF, and rate limiting. The API validates JWTs, checks object-level authorization, validates input, and logs safely. Azure SQL uses least-privilege access. Azure Key Vault protects secrets. Application Insights and Azure Monitor provide detection and audit evidence.
