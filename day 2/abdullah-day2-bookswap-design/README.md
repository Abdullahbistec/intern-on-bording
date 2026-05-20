# BookSwap Design — Day 2

This repository contains Abdullah's Day 2 BookSwap backend design deliverables.

## Contents

```text
bookswap-design/
├── diagrams/
│   ├── container-diagram.png
│   └── container-diagram.drawio
├── openapi/
│   └── bookswap-openapi.yaml
├── decisions/
│   └── storage-decisions.md
├── abdullah-day2-mock-report.md
└── README.md
```

## How to read the diagram

The member uses the React Native mobile app, not the API directly. The mobile app calls the Node.js Express API over HTTPS using a Microsoft Entra External ID JWT. The API stores structured data in Azure SQL, stores photos in Azure Blob Storage, and uses Redis only for hot read paths such as catalogue search. Asynchronous work such as notifications and weekly digest emails is published to Azure Service Bus. The worker then sends outbound email through Azure Communication Services Email.

## Validate the OpenAPI specification

```bash
npx @apidevtools/swagger-cli validate openapi/bookswap-openapi.yaml
```

## Run the mock API

```bash
npx @stoplight/prism mock openapi/bookswap-openapi.yaml --port 4010
```

## Example smoke test

```bash
curl -i "http://localhost:4010/books?page=1&pageSize=20"   -H "Authorization: Bearer mock-valid-jwt"
```

## Key decisions

- Azure SQL is the source of truth for structured BookSwap data.
- Azure Blob Storage stores all book photo binaries.
- Azure Cache for Redis caches hot, short-lived search/read results.
- Azure Service Bus decouples user actions from email and notification work.
- Azure Communication Services Email sends outbound digest emails.
- Microsoft Entra External ID handles authentication and JWT issuing.
