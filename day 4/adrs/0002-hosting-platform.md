# ADR 0002: Host GreenChit on Azure App Service

## Status
Accepted (date: 2026-05-22)

## Context
- GreenChit is an internal BISTEC tool with 99.9% business-hours availability target.
- The team needs fast delivery and simple operations.
- Azure hosting is required, and the main choices are App Service or Container Apps.

## Decision
We will host the GreenChit API on Azure App Service as a single deployable backend application. We will use deployment slots, autoscale rules, managed identity for Key Vault, and Application Insights.

## Consequences
- Easier: faster deployment, fewer moving parts, simpler operations.
- Harder: independent scaling per module is not available.
- Different: if usage grows, background workers can later be split into Container Apps.

## Alternatives considered
- Azure Container Apps split services — rejected for v1 because it adds container and IaC complexity before the team needs it.
- Azure Functions only — rejected because claim workflows and receipt handling fit better in an API service.
