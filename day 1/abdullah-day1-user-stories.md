# LearnLanka — User Story Set v0.1

## Story 1: Search for suitable tutors

**As a** Student

**I want** to search for tutors by subject, grade, teaching language, and price band

**So that** I can quickly find a tutor who matches my learning needs and budget.

### Acceptance Criteria

- **Given** a logged-in student is on the tutor search screen **when** they select subject “Combined Maths”, grade “A/L”, language “English”, and a price band **then** the results show only tutors matching all selected filters.
- **Given** no tutor matches the selected filters **when** the student taps “Search” **then** the system shows a clear “No tutors found” message and allows the student to change filters.
- **Given** matching tutors exist **when** search results are displayed **then** each result card shows tutor name, subject, grade, language, hourly rate, average rating, and next available slot.
- **Given** the search service is operating normally **when** the student submits a search **then** the first page of results is returned within 800 ms at the 95th percentile target.

### INVEST self-check

- [x] Independent
- [x] Negotiable
- [x] Valuable
- [x] Estimable
- [ ] Small — includes both functional search and a performance concern, so it may be split later.
- [x] Testable

---

## Story 2: Book and pay for a one-hour session

**As a** Student

**I want** to book a one-hour available tutor slot and pay safely

**So that** I can reserve a confirmed online lesson.

### Acceptance Criteria

- **Given** a student is viewing a tutor profile with available slots **when** they choose a 1-hour slot and submit a booking request **then** the system creates a pending booking for that selected slot.
- **Given** a booking requires payment **when** the student chooses card or eZ Cash **then** the student is redirected or connected to PayHere for payment processing.
- **Given** PayHere confirms successful payment **when** the confirmation is received by LearnLanka **then** the booking payment status changes to paid.
- **Given** PayHere reports a failed or cancelled payment **when** LearnLanka receives the result **then** the booking remains unpaid and the student is shown a retry option.

### INVEST self-check

- [x] Independent
- [x] Negotiable
- [x] Valuable
- [x] Estimable
- [ ] Small — booking and payment may be separated into two implementation stories.
- [x] Testable

---

## Story 3: Join a confirmed video session

**As a** Student

**I want** to join my confirmed online session from the platform

**So that** I can attend the lesson at the scheduled time.

### Acceptance Criteria

- **Given** a student has a paid and accepted booking **when** they open the session details during the allowed join window **then** the system shows a join session action.
- **Given** the student taps the join action **when** the video room is available **then** the student is taken to the third-party video session.
- **Given** the session is not yet within the allowed join window **when** the student views the session **then** the join action is disabled or hidden with the scheduled start time displayed.
- **Given** the third-party video provider cannot create or open the room **when** the student tries to join **then** the system shows an error and records the failure for support review.

### INVEST self-check

- [x] Independent
- [x] Negotiable
- [x] Valuable
- [x] Estimable
- [x] Small
- [x] Testable

---

## Story 4: Publish tutor availability

**As a** Tutor

**I want** to publish 1-hour availability slots

**So that** students can request sessions only during times I am available.

### Acceptance Criteria

- **Given** a verified tutor is on the availability screen **when** they add a start time and date **then** the system creates a 1-hour available slot.
- **Given** a tutor already has a slot at the same time **when** they attempt to create another overlapping slot **then** the system prevents the duplicate or overlap.
- **Given** a slot has not been booked **when** the tutor removes it **then** it no longer appears in student search or tutor profile availability.
- **Given** a slot is already attached to a pending or confirmed booking **when** the tutor tries to remove it **then** the system follows the booking cancellation or decision rules instead of silently deleting it.

### INVEST self-check

- [x] Independent
- [x] Negotiable
- [x] Valuable
- [x] Estimable
- [x] Small
- [x] Testable

---

## Story 5: Accept or decline booking requests

**As a** Tutor

**I want** to accept or decline student booking requests

**So that** I can control which sessions become confirmed lessons.

### Acceptance Criteria

- **Given** a tutor has a pending booking request **when** they accept it **then** the booking status changes to accepted and the student is notified.
- **Given** a tutor has a pending booking request **when** they decline it **then** the booking status changes to declined and the student is notified.
- **Given** a booking request has already expired, been cancelled, or been accepted/declined **when** the tutor tries to change it **then** the system prevents the action and shows the current status.
- **Given** the booking is accepted **when** session setup is required **then** the system prepares or references a third-party video room for the scheduled session.

### INVEST self-check

- [x] Independent
- [x] Negotiable
- [x] Valuable
- [x] Estimable
- [x] Small
- [x] Testable

---

## Story 6: Cancel a tutor session with notice rule

**As a** Tutor

**I want** to cancel a confirmed booking only when enough notice is given

**So that** students are protected from last-minute tutor cancellations.

### Acceptance Criteria

- **Given** a confirmed session starts more than 12 hours from now **when** the tutor cancels it **then** the system allows the cancellation and notifies the student.
- **Given** a confirmed session starts in 12 hours or less **when** the tutor attempts to cancel it **then** the system blocks the cancellation and explains the 12-hour rule.
- **Given** an admin override is available **when** an operations admin cancels the session for an exceptional reason **then** the system records the admin, reason, timestamp, and affected booking.
- **Given** a tutor cancellation is completed **when** the session status changes **then** the cancelled slot is not shown as bookable unless the tutor republishes availability.

### INVEST self-check

- [x] Independent
- [x] Negotiable
- [x] Valuable
- [x] Estimable
- [x] Small
- [x] Testable

---

## Story 7: Rate the other party after a session

**As a** Student or Tutor

**I want** to rate the other party and leave a short comment after a completed session

**So that** LearnLanka can build trust and improve future matching.

### Acceptance Criteria

- **Given** a session is completed **when** a student opens the session review screen **then** they can submit a 1 to 5 star tutor rating and one one-line comment.
- **Given** a session is completed **when** a tutor opens the session review screen **then** they can submit a 1 to 5 star student rating and one one-line comment.
- **Given** a user has already submitted a review for that session **when** they return to the review screen **then** the system prevents duplicate review submission.
- **Given** a comment exceeds the allowed one-line character limit **when** the user submits it **then** the system rejects the comment and shows the limit.

### INVEST self-check

- [x] Independent
- [x] Negotiable
- [x] Valuable
- [x] Estimable
- [ ] Small — combines student and tutor review behaviour; can be split if needed.
- [x] Testable

---

## Story 8: Prepare weekly tutor payouts

**As an** Operations Admin

**I want** to generate weekly tutor payouts after deducting LearnLanka commission

**So that** tutors are paid accurately and LearnLanka revenue is recorded.

### Acceptance Criteria

- **Given** completed sessions exist for a payout week **when** the admin generates the payout list **then** the system calculates gross session total, 15% commission, and net tutor payout for each tutor.
- **Given** a session is cancelled, refunded, disputed, or incomplete **when** payout totals are calculated **then** that session is excluded or flagged according to the payout policy.
- **Given** a payout list is generated **when** the admin reviews it **then** each row shows tutor name, bank details reference, completed session count, gross amount, commission, net amount, and payout status.
- **Given** a payout is submitted through Sampath Vishwa **when** the admin records the result **then** the payout status is updated to successful, failed, or pending.

### INVEST self-check

- [x] Independent
- [x] Negotiable
- [x] Valuable
- [ ] Estimable — depends on the unanswered Sampath Vishwa integration method.
- [ ] Small — payout generation and payout submission may become separate stories.
- [x] Testable

---

## Story 9: Handle personal data deletion requests

**As an** Operations Admin

**I want** to process student or tutor deletion requests

**So that** LearnLanka can comply with Sri Lanka Personal Data Protection Act expectations.

### Acceptance Criteria

- **Given** a student or tutor submits a deletion request **when** the request is received **then** the system records requester, timestamp, request type, and current status.
- **Given** a deletion request is pending **when** an admin opens it **then** the admin can mark it as pending verification, completed, or rejected with a reason.
- **Given** data must be retained for payment, dispute, or legal audit reasons **when** the request is processed **then** the retained data is identified and the reason is recorded.
- **Given** a deletion request status changes **when** the admin saves the update **then** an audit record is created.

### INVEST self-check

- [x] Independent
- [x] Negotiable
- [x] Valuable
- [ ] Estimable — exact retention rules need legal clarification.
- [x] Small
- [x] Testable
