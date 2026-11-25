# Part 2 – Functional Requirements FR-14 to FR-28 and Non-Functional Requirements

## 5. Functional Requirements – User Stories (continued)

### FR-14: Asynchronous Callbacks from TopUpNet

**Priority:** Must

**User Story:**

As a system, I want to accept asynchronous callbacks from TopUpNet and ignore duplicate callbacks, so that each mobile top-up payment is completed or failed exactly once.

**Acceptance Criteria:**

1. Given that a payment to TopUpNet is in the SentToBiller or Unknown state, when TopUpNet calls the callback URL with the result "success", then the system changes the payment state to Completed.

2. Given that a payment to TopUpNet is in the SentToBiller or Unknown state, when TopUpNet calls the callback URL with the result "failed", then the system changes the payment state to Failed and the held funds are released to the customer's wallet.

3. Given that a payment has already reached a final state (Completed, Rejected, or Refunded), when TopUpNet sends another callback for the same payment, then the system ignores the callback, does not change the payment state or charge the customer again, and records the duplicate in the log.

4. Given that a callback refers to a payment reference that does not exist in the system, when the callback is received, then the system does not change any payment and records the callback in the log.

### FR-15: Support for Multiple Billers with Different Formats

**Priority:** Must

**User Story:**

As a customer, I want to pay bills of different billers that use different communication protocols and data formats, so that I can pay all my utility bills from one wallet.

**Acceptance Criteria:**

1. Given that a customer pays an electricity bill, when the system communicates with ElectroCo, then it uses the ElectroCo REST/JSON contract and receives an immediate answer.

2. Given that a customer pays a water bill, when the system communicates with AquaCo, then it uses the AquaCo SOAP-style XML over HTTP contract, including its own field names and error codes, and handles its slower response time (2 to 4 seconds).

3. Given that a customer pays a mobile top-up, when the system communicates with TopUpNet, then it uses the TopUpNet REST/JSON contract, treats the first reply "accepted" as an acknowledgement only, and waits for the final result through the callback (FR-14).

4. Given that AquaCo rejects a payment, for example because the payment amount is above the bill amount, when the system receives the error code, then the system treats it as a biller refusal and changes the payment state to Failed.

5. Given that a customer pays an internet bill and the optional NetCo integration is implemented, when the payment is submitted, then the system includes it in the daily batch CSV file sent to NetCo and processes the result file received the next morning.

6. Given that a new biller must be added, when the biller is integrated, then only a new biller integration and its configuration are added, and the existing modules are not changed.

### FR-16: Fee Calculation

**Priority:** Must

**User Story:**

As a customer, I want the service fee to be calculated according to the defined fee rules, so that I know exactly how much I will pay in addition to my bill amount.

**Acceptance Criteria:**

1. Given that a payment is for an electricity or water bill, when the fee is calculated, then the service fee is a fixed 2.50 EGP.

2. Given that a payment is for a mobile top-up, when the fee is calculated, then the service fee is 1.5% of the payment amount, with a minimum of 1 EGP.

3. Given that a payment is for an internet bill, when the fee is calculated, then the service fee is 2 EGP for amounts up to and including 200 EGP, and 4 EGP for amounts above 200 EGP.

4. Given that a promotion is configured for one biller with a start date and an end date, when a payment for that biller is made within the promotion period, then the service fee is 0 EGP.

5. Given that a promotion period has not started or has ended, when a payment is made for that biller, then the normal fee rule applies.

6. Given that a fee is calculated, when the total payment amount is determined, then the total equals the bill amount plus the service fee, and all calculations use exact decimal types and never floating point.

### FR-17: Agent Payments and Commission

**Priority:** Should

**User Story:**

As an agent, I want to pay a bill on behalf of a customer and earn a commission, so that I am rewarded for serving customers at my PayLink terminal.

**Acceptance Criteria:**

1. Given that an agent is using a PayLink terminal and a customer provides the biller and billing information, when the agent submits the payment, then the system creates a payment that is recorded as made by that agent.

2. Given that a payment made by an agent reaches the Completed state, when the system completes the payment, then the system credits the agent's wallet with a commission equal to 40% of the service fee and creates the corresponding ledger entry.

3. Given that a payment made by an agent is Rejected, Failed, or still Unknown, when the system evaluates the commission, then no commission is credited to the agent.

4. Given that a payment is not made through an agent terminal, when the payment is completed, then no agent commission is recorded.

### FR-18: Payment Limits Enforcement

**Priority:** Must

**User Story:**

As a system, I want to reject payments that exceed the single-payment limits or the daily limit, so that customers and PayLink are protected from excessive or invalid payments.

**Acceptance Criteria:**

1. Given that a payment amount is below 5 EGP or above 10,000 EGP, when the payment is checked, then the system rejects the payment with a clear reason.

2. Given that a customer has a Basic verification level, when the total of the customer's payments for the day plus the new payment would exceed 5,000 EGP, then the system rejects the new payment with a clear reason.

3. Given that a customer has a Verified verification level, when the total of the customer's payments for the day plus the new payment would exceed 30,000 EGP, then the system rejects the new payment with a clear reason.

4. Given that a payment amount or the daily total is exactly equal to a limit, when the payment is checked, then the system accepts it, because only amounts above the limit are rejected.

5. Given that a payment is rejected because of a limit, when the rejection occurs, then the payment state changes from Initiated to Rejected and no funds are held.

6. Given that the limit values are defined in configuration, when the limits are checked, then the system reads the values from the configuration and not from the code.

### FR-19: Velocity Rule

**Priority:** Should

**User Story:**

As a system, I want to block an account that makes too many payments in a short time, so that suspicious or automated payment activity is stopped.

**Acceptance Criteria:**

1. Given that an account has made 5 payments within the last 60 seconds, when the same account submits another payment within that period, then the system rejects the payment and blocks the account for 10 minutes.

2. Given that an account is temporarily blocked by the velocity rule, when the account submits a payment during the 10-minute block, then the system rejects the payment with a clear reason.

3. Given that the 10-minute block period has ended, when the account submits a new payment, then the velocity rule does not reject it, and the payment continues through the normal checks.

4. Given that the velocity values (5 payments, 60 seconds, 10 minutes) are defined in configuration, when the rule is applied, then the system reads the values from the configuration.

### FR-20: Support Payment Search

**Priority:** Must

**User Story:**

As a support employee, I want to search for a payment by its reference and see its full history, so that I can investigate customer payment issues.

**Acceptance Criteria:**

1. Given that a support employee enters a payment reference, when the payment exists, then the system displays the payment details, including the customer, biller, amount, fee, total, and current state.

2. Given that a payment exists, when the support employee views it, then the system displays the full history of the payment, including every state transition with its time, the related ledger entries, and the biller interaction attempts.

3. Given that a support employee enters a payment reference that does not exist, when the search is performed, then the system informs the support employee that no payment was found.

4. Given that a user does not have the support employee role, when the user tries to search for payments by reference, then the system denies access.

### FR-21: Block and Unblock Account

**Priority:** Should

**User Story:**

As a support employee, I want to block and unblock a customer account, so that I can stop suspected fraud and restore access when the issue is resolved.

**Acceptance Criteria:**

1. Given that a support employee selects an active account, when the support employee blocks it, then the system marks the account as blocked.

2. Given that an account is blocked, when the account submits a new payment, then the fraud checks reject the payment with a clear reason.

3. Given that an account is blocked, when the support employee unblocks it, then the system marks the account as active and the account can submit payments again.

4. Given that a support employee blocks or unblocks an account, when the action is completed, then the system writes the action to the audit log with the support employee, the time, and the reason.

### FR-22: Payment Notifications

**Priority:** Must

**User Story:**

As a customer, I want to receive an SMS and a push notification when my payment completes or fails, so that I always know the result of my payment.

**Acceptance Criteria:**

1. Given that a payment reaches the Completed state, when the system sends the notifications, then the customer receives an SMS and a push notification confirming the payment.

2. Given that a payment reaches the Failed state, when the system sends the notifications, then the customer receives an SMS and a push notification with a clear reason for the failure.

3. Given that a payment becomes Unknown because the biller did not answer in time, when the system responds to the customer, then the customer is told that the payment is being processed.

4. Given that the SMS provider is unavailable when a payment completes, when the notification fails, then the payment remains Completed and the notification is handled by the retry rule (FR-23).

5. Given that a notification is sent, when the message is created, then the system builds it from a message template.

### FR-23: Notification Retry

**Priority:** Should

**User Story:**

As a system, I want to retry failed notifications and mark them as undelivered when all retries fail, so that temporary provider problems do not cause customers to miss important messages.

**Acceptance Criteria:**

1. Given that an SMS or email notification fails to be sent, when the failure is detected, then the system retries sending it up to 3 times.

2. Given that a retry succeeds, when the notification is delivered, then the system marks the notification as delivered and stops retrying.

3. Given that all 3 retries fail, when the last retry fails, then the system marks the notification as undelivered.

4. Given that a notification fails or is marked undelivered, when the payment result is evaluated, then the payment state and the customer's wallet are not affected.

### FR-24: Configurable Fee and Limit Values

**Priority:** Should

**User Story:**

As an administrator, I want to edit fee and limit values without redeploying the system, so that business rules can change quickly without code changes.

**Acceptance Criteria:**

1. Given that an administrator updates a fee value, when the update is saved, then payments initiated after the update use the new fee value, without redeploying or restarting the system.

2. Given that an administrator updates a limit value, when the update is saved, then payments initiated after the update are checked against the new limit, without redeploying or restarting the system.

3. Given that an administrator enters an invalid value, such as a negative number, when the update is submitted, then the system rejects the update and keeps the previous value.

4. Given that a user does not have the administrator role, when the user tries to edit a fee or limit value, then the system denies the action.

5. Given that a fee or limit value is changed, when the change is saved, then the system writes it to the audit log with the administrator, the time, the old value, and the new value.

### FR-25: Audit Log

**Priority:** Must

**User Story:**

As a system, I want to write every money movement and every administrative action to an audit log, so that all financial and administrative activity can be reviewed and traced.

**Acceptance Criteria:**

1. Given that a money movement occurs (top-up, hold, release, final charge, fee, commission, or refund), when the movement is recorded, then the system writes an audit record that states who performed it, when it happened, and why.

2. Given that an administrator performs an action, such as adding, editing, or disabling a biller or service, or changing a fee or limit value, when the action is completed, then the system writes an audit record with the administrator, the time, and the details of the change.

3. Given that a support employee blocks or unblocks an account, when the action is completed, then the system writes an audit record (see FR-21).

4. Given that an audit record has been written, when any user or process tries to edit or delete it, then the system does not allow the change.

### FR-26: Daily Reconciliation Report

**Priority:** Should

**User Story:**

As a support employee, I want the system to compare PayLink's payments with each biller's settlement file every night and produce a report of the differences, so that payment problems are found and reviewed.

**Acceptance Criteria:**

1. Given that a biller provides its daily settlement file, when the nightly reconciliation runs, then the system compares every payment in the settlement file with PayLink's own payment records.

2. Given that the settlement file lists a payment that PayLink has as Failed, when the comparison is completed, then the system flags the difference in the reconciliation report for support to review and does not change the payment automatically.

3. Given that a payment is in the Unknown state and the settlement file lists it as paid, when the comparison is completed, then the payment state changes to Completed.

4. Given that a payment is in the Unknown state and the settlement file does not list it, when the comparison is completed, then the payment state changes to Failed.

5. Given that a payment is Completed in PayLink but is missing from the settlement file, or the amount in the settlement file is different, when the comparison is completed, then the system flags the difference in the report.

6. Given that the reconciliation has finished, when the report is produced, then it lists every difference with its payment reference, biller, and the type of difference.

### FR-27: Refund of a Completed Payment

**Priority:** Could

**User Story:**

As a customer, I want to request a refund of a completed payment, so that I can recover my money when a payment was made by mistake, subject to support approval.

**Acceptance Criteria:**

1. Given that a payment is in the Completed state, when the customer requests a refund, then the system creates a refund request that waits for support approval and does not move any money yet.

2. Given that a refund request is waiting for approval, when a support employee approves it, then the system returns the approved amount to the customer's wallet, creates the corresponding ledger entry, and writes an audit record.

3. Given that a refund request is waiting for approval, when a support employee rejects it, then the system does not move any money and informs the customer of the decision.

4. Given that a payment is not in the Completed state, when the customer requests a refund through this feature, then the system rejects the request.

5. Given that a refund request is created, approved, or rejected, when the action is completed, then the Completed state of the original payment does not change, because Completed is a final state.

### FR-28: Admin Daily Totals

**Priority:** Could

**User Story:**

As an administrator, I want to view the daily totals of payments, fees earned, commissions paid, and failures per biller, so that I can monitor the business and detect problems.

**Acceptance Criteria:**

1. Given that an administrator selects a day, when the system prepares the daily totals, then it displays the total number and amount of payments, the total fees earned, and the total commissions paid for that day.

2. Given that an administrator selects a day, when the system prepares the daily totals, then it displays the number of failed payments for each biller.

3. Given that a day has no payments, when the administrator views its totals, then the system displays zero values.

4. Given that the totals are calculated, when fees earned are counted, then only payments that reached the Completed state are included, and all amounts use exact decimal types.

---

## 6. Non-Functional Requirements

The following six non-functional requirements are measurable. Each one has a numeric target and a defined way to measure it, so that automated tests or pipeline reports can show the number.

| ID | Quality | Measurable target | How it is measured | Priority |
|---|---|---|---|---|
| NFR-1 | Correctness | 0 double charges and 0 negative balances | Automated duplicate-request and concurrency tests | Must |
| NFR-2 | Performance | At least 100 payments per second for 1 minute | Load test against the mock billers | Should |
| NFR-3 | Resilience | 100% of payments to healthy billers complete while one biller is down | Integration test with a mock biller set to fail | Must |
| NFR-4 | Modifiability | 0 files changed outside the Biller Integration module and configuration when adding a biller | Review of the files changed in the pull request that adds a biller | Must |
| NFR-5 | Auditability | 100% of money movements have an audit record with who, when, and why | Automated test comparing ledger entries with audit records | Must |
| NFR-6 | Testability | At least 70% line coverage on domain logic | Coverage report produced by the CI pipeline | Must |

### NFR-1: Correctness

**Requirement:** The system must never charge a customer twice for one payment and must never let a wallet balance become negative.

**Target:** 0 double charges and 0 negative balances in all duplicate-request and concurrency tests.

**Measurement:**

- Send the same payment request twice with the same idempotency key. Expected result: exactly 1 payment and 1 charge, and the second request returns the first result (hard case 1, FR-10).
- Send 2 payments at the same moment from a wallet whose balance covers only one of them. Expected result: exactly 1 payment succeeds, the other is Rejected, and the available balance is never below 0 (hard case 4).
- After the tests, the balance of every wallet equals the sum of its ledger entries. Expected result: 0 differences (FR-5).

**Related requirements:** FR-5, FR-9, FR-10, FR-12.

### NFR-2: Performance

**Requirement:** The system must handle a high payment load, ready for the busiest day of the month.

**Target:** At least 100 payments per second, sustained for 1 minute, against the mock billers, and every submitted payment reaches a valid state of the payment life cycle (no payment is lost).

**Measurement:** A load test sends payments at 100 per second for 60 seconds. The test report shows the achieved rate and the final state of every payment.

**Related requirements:** FR-9, FR-11, FR-15.

### NFR-3: Resilience

**Requirement:** A problem with one biller must not stop payments to the other billers.

**Target:** While one mock biller is down or does not answer, 100% of test payments to the other billers complete normally, and the payments to the failing biller end in a correct state (Unknown with funds held, or Failed with funds released).

**Measurement:** An integration test configures one mock biller (for example AquaCo) to fail on purpose or answer after more than 10 seconds, then sends payments to ElectroCo and TopUpNet. The test reports the number of successful payments to the healthy billers, expected to be 100%.

**Related requirements:** FR-12, FR-13, FR-15.

### NFR-4: Modifiability

**Requirement:** Adding a new biller must not require changes to the existing modules.

**Target:** Adding a new biller changes 0 files outside the Biller Integration module and the configuration.

**Measurement:** The list of changed files in the pull request that adds a new biller is reviewed. Expected result: every changed file is inside the Biller Integration module or the configuration (hard case 8). The same measure applies to the ElectroCo API v2 change in Phase 2.

**Related requirements:** FR-6, FR-15.

### NFR-5: Auditability

**Requirement:** Every money movement must be traceable.

**Target:** 100% of money movements have an audit record that states who performed the action, when it happened, and why.

**Measurement:** An automated test creates top-ups, payments, failures, and refunds, and then checks that every ledger entry has a matching audit record with a non-empty actor, time, and reason. Expected result: 0 ledger entries without an audit record.

**Related requirements:** FR-5, FR-21, FR-24, FR-25.

### NFR-6: Testability

**Requirement:** The domain logic must be well covered by automated tests.

**Target:** At least 70% line coverage on the domain logic.

**Measurement:** The CI pipeline runs all tests on every push and publishes the line coverage percentage. The coverage report shows the number, and the pipeline fails if the number is below 70%.

**Related requirements:** All functional requirements (hard cases 1 to 8 each have at least one automated test).

---

## 7. Additional Identified Ambiguities and Proposed Decisions

The following ambiguities continue the list in Section 4 and relate to FR-14 to FR-28.

| ID | Related Requirement | Identified Ambiguity / Question | Proposed Decision |
|---|---|---|---|
| A-13 | FR-14 | How does the system know that a callback really comes from TopUpNet, and what happens if the callback has an unknown payment reference? | A callback with an unknown reference changes nothing and is logged. The authentication method of callbacks is not specified in the brief and requires clarification. |
| A-14 | FR-15 | Is NetCo required? | No. The brief marks NetCo batch as optional. The design must not prevent adding it later, and NetCo cannot use live inquiry (see A-07). |
| A-15 | FR-16 | For internet bills, which fee applies to exactly 200 EGP? | "Up to 200 EGP" is read as inclusive: 200 EGP has a 2 EGP fee, and amounts above 200 EGP have a 4 EGP fee. |
| A-16 | FR-16 | How is the 1.5% mobile fee rounded? | The result is rounded to 2 decimal places using exact decimal types (half-up proposed), and the minimum of 1 EGP applies. The team confirms the rounding mode. |
| A-17 | FR-16, FR-24 | What fee applies if an administrator changes the fee between the bill inquiry and the payment? | The fee shown to the customer in the inquiry is the fee charged for that payment. New values apply to later inquiries. |
| A-18 | FR-16 | Does a promotion apply to all services of the chosen biller, and can several promotions overlap? | A promotion applies to all services of the one configured biller during its dates. Overlapping promotions are not specified and are not supported. |
| A-19 | FR-17 | Which wallet or cash source pays the bill when an agent pays for a customer? Is commission paid on top-ups? | The brief defines commission only for bill payments made at an agent terminal (40% of the service fee). The funding source requires clarification. |
| A-20 | FR-18 | Do the single-payment and daily limits apply to the bill amount only or to the total including the fee? | The limits apply to the bill amount. The team must confirm before implementation. Note that for Basic accounts the daily limit (5,000 EGP) is lower than the single-payment maximum (10,000 EGP). |
| A-21 | FR-18 | What is "a day", and which payments count toward the daily limit? | A day is the calendar day in Egypt local time. Payments in Completed, FundsHeld, SentToBiller, or Unknown states count. Rejected and Refunded payments do not count. |
| A-22 | FR-19 | Do rejected payment attempts count toward the velocity rule, and is the 60-second window sliding or fixed? | All payment attempts count, and the window is sliding. The team confirms this decision. |
| A-23 | FR-20, FR-21, FR-24 | How do users log in, and how are the customer, agent, administrator, and support roles assigned? | Authentication and role management are not specified in the brief. Roles are assumed to exist, and the mechanism requires clarification. |
| A-24 | FR-21 | What happens to payments already in progress and to other operations (top-up, balance view) when an account is blocked? | Payments already in progress continue to a final state because their funds are held. Blocking stops new payments only. Other operations require clarification. |
| A-25 | FR-22 | Is a notification sent for Rejected payments, and through which channel is the "being processed" message for Unknown payments delivered? | The brief requires notifications for completion and failure only. Notifications for Rejected and Unknown payments, and their channel, require clarification. |
| A-26 | FR-23 | Does "retried 3 times" mean 3 retries after the first attempt, and how long is the delay between retries? | 3 retries after the first attempt (4 attempts in total). The delay is not specified and is defined in configuration. |
| A-27 | FR-24 | Which values can an administrator edit? | All business rules in Section 2.6 of the brief are stored in configuration. At minimum, fee and limit values are editable at runtime. |
| A-28 | FR-25 | How long are audit records kept, and who can read them? | Audit records are append-only and cannot be edited or deleted. Retention time and read access require clarification. |
| A-29 | FR-26 | What is the format and arrival time of settlement files, and what happens to a payment that PayLink has as Failed but the biller lists as paid? | Such a payment is only flagged for support review and is not changed automatically. The file format follows the biller contract provided by the instructors. |
| A-30 | FR-27 | Completed is a final state, so how is a refund of a completed payment represented? Is the fee refunded and the agent commission reversed? | The refund is a separate refund request and does not change the Completed state of the payment. Fee refund, commission reversal, and notifying the biller require clarification. |
| A-31 | FR-28 | What counts as a failure in "failures per biller"? | Payments in the Failed state are counted. Rejected and Unknown payments can be shown separately. Fees earned count only Completed payments. |
