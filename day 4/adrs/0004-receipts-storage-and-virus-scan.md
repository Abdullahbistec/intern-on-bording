# ADR 0004: Store receipts in Blob Storage using signed URLs and scan after upload

## Status
Accepted (date: 2026-05-22)

## Context
- Staff can attach up to 5 receipt images per claim, up to 10 MB per file.
- Receipt upload must work with intermittent connectivity.
- Receipt images are binary files and should not bloat the database.
- User-uploaded files create malware risk.

## Decision
We will store receipt binaries in Azure Blob Storage. The API will issue short-lived signed upload URLs after validating metadata. SQL stores metadata and blob references. Uploaded files will be scanned asynchronously, and payroll export is blocked until receipts are clear.

## Consequences
- Easier: large files are stored cheaply and upload retries do not tie up API threads.
- Harder: the system must handle `receipt_upload_pending`, `scan_pending`, and `scan_failed`.
- Different: claim confirmation can happen before every downstream scan completes, but export waits for safe receipts.

## Alternatives considered
- Store binaries in Azure SQL — rejected because it increases database size and backup cost.
- Upload through the API only — rejected because large files increase timeout risk.
- Skip scanning in v1 — rejected because finance users will open uploaded files.
