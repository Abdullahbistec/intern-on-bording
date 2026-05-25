# GreenChit — Trade-offs and Design Review

## Setup

Two architectural options are compared:

- **Option A: App Service monolith** — one GreenChit API on Azure App Service with internal components.
- **Option B: Container Apps split** — multiple smaller services on Azure Container Apps.

Scores use 1–5, where 5 is stronger.

## Results Summary

| Metric | Target | Achieved |
|--------|--------|----------|
| Quality attributes scored | 6 | 7 |
| Cells with a written justification | 12 | 14 |
| Decision-affecting attributes identified | 2-3 | 3 |

## Trade-off table

| Quality attribute | Option A: App Service monolith | Option B: Container Apps split | Why |
|---|---:|---:|---|
| Time-to-first-deploy | 5 | 2 | App Service has fewer deployment pieces. Container Apps needs container builds, registry, networking, and more IaC. |
| Cost / low spend | 5 | 3 | One App Service plan plus managed services is cheaper for v1. Split services increase baseline operational cost. |
| Operability for 10-person team | 4 | 3 | App Service has simpler logs, scaling, and deployment. Container Apps needs more operational maturity. |
| Independent deploy | 1 | 5 | App Service monolith redeploys the whole API. Container Apps can deploy services separately. |
| Future scaling | 2 | 5 | App Service scales the whole API together. Container Apps can scale receipt/export/notification components independently. |
| Authn/authz consistency | 4 | 3 | A monolith centralizes RBAC and claim visibility checks. Split services need shared policy enforcement to avoid drift. |
| Business-hours availability | 4 | 4 | Both can meet 99.9% business-hours availability if configured well. |
| **Total** | **25** | **25** | The total is tied, so the decision is based on business priorities, not arithmetic alone. |

## Decision and rationale

We choose **Option A: App Service monolith for v1**.

The decision is driven by time-to-first-deploy, operability for a small team, and authn/authz consistency. The uncomfortable trade-off is weaker independent deployment and per-component scaling. We accept this because GreenChit v1 is an internal business-hours tool.

## Design review feedback received from another pair

### 3 strengths

1. Container and component diagrams separate runtime containers from internal API responsibilities clearly.
2. ADRs are decisive and explain rejected alternatives.
3. Receipt storage design correctly avoids storing images in the database.

### 3 weaknesses or risks

1. The first design did not clearly show how failed receipt upload returns the claim to a recoverable state.
2. The audit log was described as tamper-evident, but the hashing/signing mechanism needed clearer explanation.
3. Email fallback could create duplicate manager notifications if Teams succeeds slowly and email retry also fires.

### 2 actionable improvements

1. In `diagrams/sequence-submit-approve.md`, add an error branch for receipt upload failure and show status update to Draft with `upload_failed`.
2. In `adrs/0004-receipts-storage-and-virus-scan.md`, add a rule that payroll export blocks claims until receipt scan status is clear.

## Design review feedback given to another pair

### 3 strengths

1. Their sequence diagram was easy to follow and showed claimant and manager actions.
2. Their trade-off table had honest scoring and did not blindly choose the highest total.
3. Their ADR for database choice clearly connected workflow data to relational storage.

### 3 weaknesses or risks

1. Their component diagram looked like a class diagram and had too many method names.
2. Their container diagram did not label protocols on all arrows.
3. Their ADR consequences section did not name uncomfortable trade-offs.

### 2 actionable improvements

1. Replace class-like boxes such as `ClaimController` with responsibility-level components such as Claims Component and Audit Component.
2. Add one “Harder” consequence to each ADR, especially around App Service monolith deployment and Azure SQL migrations.
