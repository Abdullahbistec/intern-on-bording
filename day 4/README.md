# GreenChit Design Pack — Day 4

This repository contains Abdullah's Day 4 GreenChit architecture design pack.

## Structure

```text
abdullah-day4-greenchit-design/
├── README.md
├── abdullah-day4-greenchit-design.md
├── diagrams/
│   ├── container-diagram.png
│   ├── container-diagram.drawio
│   ├── component-diagram.png
│   ├── component-diagram.drawio
│   ├── sequence-submit-approve.md
│   └── sequence-submit-approve.png
├── adrs/
│   ├── 0001-record-architecture-decisions.md
│   ├── 0002-hosting-platform.md
│   ├── 0003-database-choice.md
│   └── 0004-receipts-storage-and-virus-scan.md
└── trade-offs/
    └── hosting-options.md
```

## Reading order

1. Read `abdullah-day4-greenchit-design.md`.
2. Open `diagrams/container-diagram.png`.
3. Open `diagrams/component-diagram.png`.
4. Read `diagrams/sequence-submit-approve.md`.
5. Read the ADRs.
6. Read `trade-offs/hosting-options.md`.

## v1 limitations

GreenChit v1 does not include OCR receipt extraction, full payroll processing, external users, or a native offline-first mobile app.
