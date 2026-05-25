# ADR 0003: Use Azure SQL as the GreenChit system of record

## Status
Accepted (date: 2026-05-22)

## Context
- Claims have structured fields and clear workflow states.
- Finance needs CSV export and audit queries.
- Privacy rules depend on relationships between claimant, manager, finance, and audit roles.

## Decision
We will use Azure SQL as the system of record for claims, claim states, receipt metadata, approvals, audit logs, and export records.

## Consequences
- Easier: relational constraints, transactions, reporting queries, and team familiarity.
- Harder: schema changes require migrations.
- Different: receipt binaries remain outside SQL in Blob Storage.

## Alternatives considered
- Cosmos DB — rejected because the data is strongly relational and workflow-based.
- Blob-only JSON files — rejected because reporting and audit queries would be difficult.
