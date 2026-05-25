# ADR 0001: Record architecture decisions

## Status
Accepted (date: 2026-05-22)

## Context
- GreenChit has design choices that affect cost, delivery speed, security, and maintenance.
- Finance, engineering, and operations need a clear record of why choices were made.
- Future engineers should understand the reasoning without relying on memory.

## Decision
We will record important architecture decisions using Nygard-style ADRs in the `adrs/` folder. Each ADR must include status, context, decision, consequences, and alternatives considered.

## Consequences
- Easier: reviewers can understand the reasoning behind each major design choice.
- Harder: engineers must spend time writing ADRs before or during implementation.
- Different: future changes should supersede old ADRs rather than silently replacing them.

## Alternatives considered
- Keep decisions only in meeting notes — rejected because notes are hard to find.
- Keep decisions only in diagrams — rejected because diagrams show structure, not trade-offs.
