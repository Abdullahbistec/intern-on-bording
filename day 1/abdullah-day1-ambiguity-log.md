# LearnLanka — Ambiguity Hunt Log

## Brief reference

Source paragraph:

> LearnLanka is a Colombo-based startup that connects O/L and A/L students with vetted tutors for one-to-one online sessions. They have a one-paragraph brief, no diagrams, and three contradictory expectations from the founders. Your job is to turn the brief into a clear requirements document, a C4 Context diagram, a user story set, and a list of clarification questions the team should ask before any code is written.

Ambiguous phrases highlighted:

- “connects”
- “O/L and A/L students”
- “vetted tutors”
- “one-to-one online sessions”
- “three contradictory expectations”
- “clear requirements”
- “before any code is written”

Additional ambiguity sources are also taken from the functional requirements, non-functional requirements, and technical constraints in the full challenge brief.

## Findings

| # | Quote | Why ambiguous | Clarification question | Priority |
|---:|---|---|---|:---:|
| 1 | “vetted tutors” | The brief does not define what checks are required before a tutor is trusted on the platform. This affects onboarding workflow, admin tools, legal risk, and launch timeline. | What exact tutor vetting evidence is required before publication: NIC, certificates, service letters, interview, demo class, police clearance, or manual admin approval? | H |
| 2 | “Students must be able to search for tutors by subject, grade, language, and price band” | It does not say whether all filters are mandatory, whether multiple subjects/languages can be selected, or how results should be sorted. | Which search filters are required, can students select multiple values, and what is the default sort order: rating, price, availability, relevance, or sponsored ranking? | H |
| 3 | “book a 1-hour session with a tutor” | It does not define whether booking is instantly confirmed after payment or requires tutor acceptance first. This changes booking state design and payment timing. | Does a paid booking become confirmed immediately, or must the tutor accept the booking before the student is charged or before funds are captured? | H |
| 4 | “pay via card or eZ Cash” | It does not define payment failure, refunds, partial refunds, cancellation refunds, or whether eZ Cash is supported through PayHere in the required flow. | What refund and payment failure rules apply for card and eZ Cash payments, especially for tutor decline, student cancellation, and tutor cancellation? | H |
| 5 | “accept or decline bookings” | It does not specify the time limit for tutors to respond or what happens if the tutor never responds. | How long does a tutor have to accept or decline a booking request before it expires automatically? | H |
| 6 | “cancel with at least 12 hours notice” | It names a tutor cancellation rule but does not say whether it applies to students, admins, emergencies, refunds, or time zone handling. | Does the 12-hour cancellation rule apply only to tutors, and what exceptions or penalties apply for emergencies and student cancellations? | H |
| 7 | “completed session” | Commission and payout depend on completion, but the brief does not define how completion is detected or disputed. | What makes a session completed: scheduled end time reached, both parties joined, minimum attendance minutes, or manual admin confirmation? | H |
| 8 | “pay tutors weekly via bank transfer” | It does not define payout day, cutoff time, failed payout handling, or whether Sampath Vishwa integration is manual, file-based, or API-based. | What is the weekly payout cutoff and what exact Sampath Vishwa method will be used: manual upload, CSV/SFTP, API, or admin-entered transfer? | H |
| 9 | “Both parties must be able to rate each other” | It does not define whether ratings are public, editable, anonymous, moderated, or blocked during disputes. | Are ratings/comments public immediately, can they be edited, and should reviews be hidden during disputes or moderation? | M |
| 10 | “one-line comment” | It does not specify character limit, supported languages, profanity rules, or moderation process. | What is the maximum character count for a one-line comment, and what content moderation rules should apply? | M |
| 11 | “under 800 ms at the 95th percentile from a Sri Lankan ISP” | It does not define the test location, endpoint, payload size, data volume, network type, or measurement tooling. | Which search endpoint, dataset size, test location, and monitoring tool will be used to measure the 800 ms p95 search target? | H |
| 12 | “99.5% monthly uptime, measured against the booking endpoint” | It does not define whether planned maintenance counts, what success means, or which booking endpoint/state is measured. | What exact URL and success criteria define the booking endpoint uptime SLO, and does planned maintenance count as downtime? | H |
| 13 | “support 200 simultaneous video sessions” | It does not say whether this means 200 rooms, 200 participants, or 200 one-to-one sessions with 400 users. It also affects third-party video capacity and cost. | Does “200 simultaneous video sessions” mean 200 active one-to-one rooms, 200 total participants, or another concurrency measure? | H |
| 14 | “comply with Sri Lanka Personal Data Protection Act 2022” | Compliance obligations are broad and legal-specific; the brief names consent and deletion but not retention, lawful basis, breach handling, or DPO responsibilities. | What retention, lawful basis, consent wording, breach notification, and data subject request rules must be approved by legal before launch? | H |
| 15 | “Payment data: never stored on LearnLanka servers” | It may still be necessary to store transaction IDs, gateway references, amount, status, and last payment result. The phrase could be interpreted too broadly. | Which payment-related fields may LearnLanka store: PayHere transaction ID, masked method, amount, status, invoice number, refund ID, and webhook logs? | H |
| 16 | “Mobile-first product” | It does not specify whether launch is native Android, responsive web, PWA, or both. This changes architecture, testing, and release process. | Is version 1 a responsive web app, Android native app, PWA, or a combination? | H |
| 17 | “at least 80% of usage is expected on Android devices” | It is a forecast, not a testable requirement unless converted into device support and QA coverage. | Which Android versions, screen sizes, and browsers/devices must be officially supported at launch? | M |
| 18 | “preferred cloud is Azure” | It says preferred, not mandatory, and names services but not region, disaster recovery, backup, or cost limits. | Is Azure mandatory for production, and which Azure region, backup policy, and monthly cost limit should be assumed? | M |
| 19 | “Daily.co or 100ms” | Two different vendors have different SDKs, APIs, pricing, region support, recording options, and compliance terms. | Which video provider should be selected for v1, and are recording, screen sharing, chat, or attendance logs required? | H |
| 20 | “All UI strings must support Sinhala, Tamil, and English from launch” | It does not say whether user content, emails, SMS messages, validation errors, admin screens, and legal text are included. | Does localization apply to all customer screens, tutor screens, admin screens, SMS messages, emails, payment labels, and legal documents? | M |
| 21 | “three contradictory expectations from the founders” | The actual contradictory expectations are not listed, so the team cannot resolve trade-offs. | What are the three founder expectations, and which one has priority if cost, speed, quality, and compliance conflict? | H |
| 22 | “Operations Admin” | The challenge expects an admin persona, but the brief does not define admin permissions, roles, or audit requirements. | What admin roles are needed at launch, and what actions require audit logs or approval by a second admin? | M |

## Results Summary

| Metric | Target | Achieved |
|--------|--------|----------|
| Items found | 10+ | 22 |
| High-priority items | 3+ | 15 |
| Items convertible to test cases | 5+ | 16 |

## Top 3 questions to ask the founders

1. Does a student booking become confirmed immediately after payment, or does the tutor need to accept before payment is captured and the session becomes confirmed?
2. What exactly counts as a completed session for commission, payout, refund, and dispute purposes?
3. Which v1 client platform must be built first: responsive web, Android native app, PWA, or a combination?

## Reflection

The ambiguity that caused the most risk was lifecycle ambiguity. Words such as “book”, “accept”, “completed”, “cancel”, and “pay” sound simple, but they hide the most important business rules: payment timing, refund responsibility, tutor control, student protection, and payout accuracy.

The question most likely to change the architecture is whether booking is confirmed immediately after payment or only after tutor acceptance. If tutor acceptance is required before payment capture, the system needs more booking states, payment authorization/expiry handling, notifications, and timeout rules. If payment immediately confirms the booking, the architecture is simpler but creates higher risk of tutor rejection and refund handling. Resolving this before coding prevents expensive rework in the booking, payment, payout, and notification flows.
