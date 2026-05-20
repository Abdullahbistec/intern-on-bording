# LearnLanka — Requirements Document

## 1. Problem Statement

LearnLanka needs a trusted, mobile-first online tutoring platform that helps Sri Lankan O/L and A/L students quickly find suitable vetted tutors, book and pay for one-hour online lessons, attend sessions through an outsourced video provider, and review the learning experience afterwards. The platform must also support tutors with availability management, booking decisions, cancellations, ratings, and weekly payouts while protecting student and tutor personal data, avoiding storage of card/payment details, and meeting measurable performance, availability, localization, and compliance targets from launch.

## 2. Personas

### Persona 1 — Student: “Nethmi”, Grade 12 A/L Student
- **Profile:** Uses an Android phone and mobile data after school; wants affordable one-to-one help for difficult subjects such as Combined Maths or Chemistry.
- **Goals:** Search tutors by subject, grade, language, price band, rating, and availability; book quickly; pay safely; join the session without technical issues.
- **Frustrations:** Slow search results, unclear tutor prices, tutors who do not respond, payment failures, and lessons that start late or have poor video quality.

### Persona 2 — Tutor: “Mr. Rizwan”, Part-time Tutor
- **Profile:** Teaches O/L Mathematics and A/L Physics in Sinhala and English; wants extra income without manual coordination through WhatsApp.
- **Goals:** Publish available time slots, accept or decline bookings, avoid last-minute cancellations, receive fair ratings, and get paid weekly with transparent commission deductions.
- **Frustrations:** Double-bookings, students missing sessions, unclear payout calculations, unfair reviews, and too much admin work for schedule changes.

### Persona 3 — Operations Admin: “Chamodi”, LearnLanka Support/Admin
- **Profile:** Works for LearnLanka operations; manages tutor vetting, disputes, compliance requests, and payment exceptions.
- **Goals:** Monitor bookings, review tutor profiles, manage complaints, process deletion requests, reconcile completed sessions with PayHere and Sampath Vishwa, and ensure service reliability.
- **Frustrations:** Missing audit trails, unclear booking status, manual payout errors, unresolved student/tutor disputes, and lack of evidence for privacy or compliance requests.

## 3. Functional Requirements

### 3.1 Student Requirements

1. **Student account registration:** A student shall be able to create an account using name, mobile number, email address, preferred language, grade, and consent to the privacy policy.
2. **Student login:** A registered student shall be able to sign in and access student features after successful authentication.
3. **Tutor search:** A student shall be able to search for tutors using subject, grade, teaching language, and price band as filters.
4. **Search results display:** The system shall display each matching tutor with tutor name, subject(s), grade(s), language(s), hourly rate, average rating, and next available slot.
5. **Tutor profile view:** A student shall be able to open a tutor profile and view tutor bio, qualifications summary, subjects, languages, hourly rate, availability, and rating summary.
6. **Session booking request:** A student shall be able to select an available 1-hour slot and submit a booking request for that tutor.
7. **Payment initiation:** A student shall be able to pay for a requested booking using card or eZ Cash through PayHere.
8. **Payment confirmation:** The system shall record a booking as paid only after receiving a successful confirmation from PayHere.
9. **Session joining:** A student shall be able to join a confirmed session through the provided video session link during the allowed joining window.
10. **Student cancellation request:** A student shall be able to request cancellation of a future session according to the platform cancellation policy.
11. **Post-session rating:** After a completed session, a student shall be able to rate the tutor from 1 to 5 stars and leave one one-line comment.
12. **Privacy request:** A student shall be able to submit a personal data deletion request from the account/privacy area.

### 3.2 Tutor Requirements

13. **Tutor account registration:** A tutor shall be able to create a tutor account with name, contact details, subjects, grades, languages, hourly rate, bank details, and verification information.
14. **Tutor profile publication:** A verified tutor shall be able to publish or unpublish their public tutor profile.
15. **Availability management:** A tutor shall be able to create, edit, and remove 1-hour availability slots.
16. **Booking decision:** A tutor shall be able to accept or decline a new booking request before it becomes a confirmed session.
17. **Tutor cancellation rule:** A tutor shall be able to cancel a confirmed booking only when the session start time is at least 12 hours away, unless an admin override is applied.
18. **Tutor schedule view:** A tutor shall be able to view upcoming, completed, cancelled, and pending sessions.
19. **Post-session student rating:** After a completed session, a tutor shall be able to rate the student from 1 to 5 stars and leave one one-line comment.
20. **Payout visibility:** A tutor shall be able to view completed sessions, gross earnings, LearnLanka 15% commission, net payout amount, payout status, and payout date.

### 3.3 Operations Admin Requirements

21. **Tutor vetting:** An operations admin shall be able to review tutor registration details and mark a tutor as approved, rejected, or requiring more information.
22. **Booking monitoring:** An operations admin shall be able to search and view bookings by student, tutor, subject, status, date range, and payment status.
23. **Commission calculation:** The system shall calculate LearnLanka commission as 15% of the session fee for every completed session.
24. **Weekly payout preparation:** The system shall generate a weekly tutor payout list containing tutor bank details, completed session totals, commission deducted, and net payout amount.
25. **Payout status update:** An operations admin shall be able to mark payouts as pending, submitted, successful, or failed based on Sampath Vishwa processing.
26. **Review moderation:** An operations admin shall be able to view ratings/comments and hide comments that violate published content rules.
27. **Dispute handling:** An operations admin shall be able to record a dispute against a session and update its resolution status.
28. **Consent audit:** The system shall store the date, time, policy version, and channel for each user consent decision.
29. **Deletion request handling:** An operations admin shall be able to view, process, and mark personal data deletion requests as completed, rejected with reason, or pending verification.
30. **Localization management:** An operations admin shall be able to identify missing Sinhala, Tamil, or English UI strings before release.

## 4. Non-Functional Requirements

| Category | Metric | Target | How we'll measure it |
|---|---:|---|---|
| Search performance | Tutor search API response time at 95th percentile from a Sri Lankan ISP | < 800 ms | Azure Application Insights synthetic tests from Sri Lanka or nearest available region plus real-user monitoring |
| Booking availability | Successful response percentage for booking endpoint | 99.5% uptime per calendar month | Azure Monitor availability tests and server-side request success metrics |
| Video concurrency | Number of simultaneous active video sessions | Support at least 200 simultaneous sessions during first 6 months | Daily.co/100ms dashboard and LearnLanka active session telemetry |
| Payment security | Storage of card/eZ Cash sensitive payment data on LearnLanka servers | 0 records stored | Data model review, security audit, and gateway integration test evidence |
| Payment gateway compliance | Payment processing handled through PCI-DSS compliant provider | 100% of payment flows through PayHere-hosted/API-approved flow | Integration logs, payment architecture review, and PayHere documentation check |
| Privacy compliance | Consent captured before account use | 100% of registered users have policy version, timestamp, and channel recorded | Database audit query and registration test cases |
| Privacy deletion | Deletion request acknowledgement time | Within 48 hours of submission | Admin workflow timestamps and audit log |
| Localization | UI launch language coverage | 100% of user-facing strings available in Sinhala, Tamil, and English | Localization key coverage report in CI/CD |
| Mobile usability | Primary student and tutor flows usable on Android mobile viewport | Search, booking, payment, joining, rating, and availability flows pass mobile QA | Android device/browser test checklist |
| Reliability | Failed booking requests caused by server error | < 1% of total booking attempts monthly | Azure Application Insights request failure rate |
| Observability | Critical production incidents with trace/log evidence | 100% of P1/P2 incidents have logs and correlation IDs | Incident review checklist |
| Payout accuracy | Difference between calculated net payout and payout file total | 0 unresolved mismatches per weekly payout cycle | Weekly reconciliation report comparing completed sessions and Sampath Vishwa submission |

## 5. Assumptions

1. Students and tutors must register and log in before booking, accepting, joining, rating, or managing sessions.
2. Tutor verification is required before a tutor profile becomes searchable by students.
3. Each bookable lesson is exactly 60 minutes in version 1.
4. The system will support O/L and A/L grades only at launch, not university or professional courses.
5. The first launch will be web/mobile-web or Android-focused; a native iOS app is not required for version 1.
6. LearnLanka will use PayHere for both card and eZ Cash payment initiation/confirmation.
7. Video calling rooms will be created through Daily.co or 100ms using API integration and not built in-house.
8. Sampath Vishwa payout processing may require admin review or file/API submission rather than fully automated instant transfer.
9. The 15% commission is calculated on the gross session fee before tutor payout and after the session is completed.
10. A “completed session” means a paid and accepted session that reached its scheduled end time and was not cancelled or refunded.
11. Ratings are available only after the scheduled session has ended.
12. One-line comments require a maximum character limit, assumed as 200 characters unless founders decide otherwise.
13. SMS is required for OTPs or reminders, although the specific SMS provider is not named in the brief.
14. All user-facing text must be managed through a localization system from launch.
15. Deletion requests may retain legally required financial/audit records in anonymised or limited form if required by law.

## 6. Out of Scope

1. Building an in-house video conferencing system.
2. Storing card numbers, CVV values, or sensitive payment credentials on LearnLanka servers.
3. Native iOS application for the first release.
4. Group classes, webinars, or batch tuition sessions.
5. AI tutor matching or automated learning-path recommendations.
6. Offline/in-person tuition booking.
7. In-app chat with file sharing beyond essential booking/session notifications.
8. Subscription plans or monthly student memberships.
9. Tutor tax filing or legal employment management.
10. Full accounting system replacement for LearnLanka finance operations.
