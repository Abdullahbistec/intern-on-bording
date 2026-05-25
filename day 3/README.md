# BookSwap Quality Attribute Plan — Day 3

This repository contains Abdullah's Day 3 BookSwap quality attribute deliverables.

## Structure

```text
abdullah-day3-bookswap-quality/
├── slo/
│   └── slo-map.md
├── reliability/
│   └── runbook.md
├── security/
│   ├── review.md
│   ├── threats.csv
│   └── zap-baseline-report.html
├── observability/
│   ├── plan.md
│   └── alerts.yaml
└── README.md
```

## How to read

Start with `slo/slo-map.md` to see how NFRs become measurable SLIs and SLOs. Then read `reliability/runbook.md` for SQL failure, Redis failure, and Sunday 10× traffic spike response. Next, review `security/review.md` and `security/threats.csv` for OWASP findings including BOLA, injection, secrets, rate limits, and PII. Finally, read `observability/plan.md` and `observability/alerts.yaml` for logs, metrics, traces, dashboards, and alert rules.

## Note about ZAP

`security/zap-baseline-report.html` is included as a ready-to-replace report template. Run OWASP ZAP against the Prism mock and replace it with the generated report before a final production-quality submission.
