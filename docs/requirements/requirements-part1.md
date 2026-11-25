# PayLink - System Requirements Specification

## 📋 Table of Contents
1. [Problem Summary](#1-problem-summary)
2. [Stakeholders](#2-stakeholders)
3. [Functional Requirements – User Stories](#3-functional-requirements--user-stories)
4. [Identified Ambiguities and Proposed Decisions](#4-identified-ambiguities-and-proposed-decisions)

---

## 1. Problem Summary

**PayLink** is a digital payment platform in Egypt that provides customers with a convenient way to pay their utility bills, including electricity, water, mobile, and internet services, through a digital wallet.

Currently, customers may need to visit utility offices or rely on traditional payment methods to settle their bills. This can be time-consuming and inconvenient. PayLink aims to simplify this process by allowing customers to make payments through a mobile application or authorized agents using PayLink terminals.

The system provides customers with digital wallets that support:
* Money deposits (Top-ups)
* Balance inquiries
* Transaction history
* Bill payments (retrieving outstanding amounts, viewing service fees, and confirming payments)

However, handling financial transactions introduces several challenges. The system must ensure that payments are processed accurately, prevent duplicate transactions, maintain correct wallet balances, and protect customers' funds when payment operations fail or experience delays.

PayLink must also integrate with multiple billers that use different communication protocols and payment mechanisms. Therefore, the backend should be flexible enough to support different integration methods and allow new billers to be added without affecting existing components.

> **Goal:** Provide a secure, reliable, and extensible backend system that simplifies bill payments while maintaining financial accuracy, transaction consistency, and customer trust.

---

## 2. Stakeholders

Stakeholders are individuals, groups, or organizations that interact with the PayLink system or are affected by its operations.

| Stakeholder | Category | Description and Responsibilities |
| :--- | :--- | :--- |
| **Customers** | Primary | Register accounts, manage their wallets, deposit money, inquire about bills, make payments, and view transaction history. |
| **Agents** | Primary | Assist customers with cash top-ups and bill payments through PayLink terminals and receive commissions for successful transactions. |
| **Administrators** | Primary | Manage billers and their services, including adding, editing, and disabling them. |
| **Support Employees** | Internal | Investigate customer payment issues, review transaction records, and handle reconciliation differences. |
| **PayLink Management** | Internal | Define business objectives, monitor operations, and ensure that the platform meets business requirements. |
| **Development and Operations Team** | Internal | Develop, test, deploy, maintain, and monitor the PayLink backend system. |
| **Billers (Utility Companies)** | External | Provide bill information, receive payment requests, confirm or reject payments, and provide settlement information. |
| **Regulatory Authorities** | External | Oversee applicable financial regulations and compliance requirements. |
| **Notification Providers** | External | Deliver SMS, email, and push notifications to customers. |

---

## 3. Functional Requirements – User Stories

### FR-1: Customer Registration
* **Priority:** `Must`
* **User Story:** As a new customer, I want to register an account with PayLink and receive a basic verification level, so that I can access the platform's wallet and payment services.
* **Acceptance Criteria:**
  * **Given** that a new customer provides the required registration information, **when** the registration process is completed successfully, **then** the system creates a new customer account.
  * **Given** that a customer account is successfully created, **when** the account is initialized, **then** the customer's verification level is automatically set to `Basic`.

### FR-2: Upgrade Verification Level
* **Priority:** `Should`
* **User Story:** As a customer, I want my account to be upgraded from Basic to Verified, so that I can benefit from higher daily payment limits.
* **Acceptance Criteria:**
  * **Given** that a customer's account has a `Basic` verification level, **when** the account is successfully upgraded, **then** the system updates the verification level to `Verified`.
  * **Given** that a customer's verification level is `Basic`, **when** the system checks the applicable daily payment limit, **then** the limit is **5,000 EGP**.
  * **Given** that a customer's verification level is `Verified`, **when** the system checks the applicable daily payment limit, **then** the limit is **30,000 EGP**.

### FR-3: View Wallet Balance and Transaction History
* **Priority:** `Must`
* **User Story:** As a customer, I want to view my wallet balance and transaction history, so that I can monitor my available funds, held funds, and previous financial activities.
* **Acceptance Criteria:**
  * **Given** that a customer is logged into their account, **when** they request their wallet balance, **then** the system displays both the available balance and the held balance.
  * **Given** that a customer has previous wallet transactions, **when** they request their transaction history, **then** the system displays the recorded transactions associated with their wallet.
  * **Given** that a customer's wallet contains held funds, **when** they view their wallet information, **then** the held amount is displayed separately from the available balance.

### FR-4: Wallet Top-up
* **Priority:** `Must`
* **User Story:** As a customer, I want to add money to my digital wallet using a simulated card payment or through an agent, so that I can have sufficient funds to pay my bills.
* **Acceptance Criteria:**
  * **Given** that a customer chooses the simulated card top-up method, **when** the top-up operation is completed successfully, **then** the system adds the corresponding amount to the customer's wallet.
  * **Given** that a customer chooses the agent cash top-up method, **when** the agent completes the top-up operation successfully, **then** the system adds the corresponding amount to the customer's wallet.
  * **Given** that a top-up operation is completed successfully, **when** the wallet balance is updated, **then** the system creates the corresponding ledger entry.

### FR-5: Ledger Management
* **Priority:** `Must`
* **User Story:** As a system, I want to record every wallet balance change as a ledger entry, so that all financial movements are accurately tracked, and wallet balances can be calculated from recorded entries.
* **Acceptance Criteria:**
  * **Given** that a financial operation changes a customer's wallet balance, **when** the operation is completed, **then** the system creates a corresponding ledger entry.
  * **Given** that a wallet contains multiple ledger entries, **when** the system calculates its balance, **then** the balance is calculated from the recorded ledger entries.
  * **Given** that a financial operation is performed, **when** the system records the corresponding financial movement, **then** it must not directly overwrite the wallet balance without recording the movement in the ledger.

### FR-6: Manage Billers and Services
* **Priority:** `Must`
* **User Story:** As an administrator, I want to add, edit, and disable billers and their services, so that I can manage the utility companies and services available through PayLink.
* **Acceptance Criteria:**
  * **Given** that an administrator wants to add a new biller, **when** the administrator provides the required information, **then** the system adds the biller to the catalog.
  * **Given** that an existing biller or service requires modification, **when** the administrator updates its information, **then** the system saves the updated information.
  * **Given** that an administrator wants to disable a biller or service, **when** the administrator performs the disabled action, **then** the system marks the selected biller or service as disabled.

### FR-7: View Available Billers
* **Priority:** `Must`
* **User Story:** As a customer, I want to view the available billers and the required input fields for each service, so that I know which services I can use and what information I need to provide.
* **Acceptance Criteria:**
  * **Given** that a customer accesses the biller catalog, **when** the system retrieves the available billers, **then** it displays the active billers and their available services.
  * **Given** that a customer selects a particular service, **when** the system displays its information, **then** it shows the required input fields, such as meter number or phone number, according to the selected service.
  * **Given** that a biller or service is disabled, **when** the customer accesses the available services, **then** the disabled service is not offered as an available payment option.

### FR-8: Bill Inquiry
* **Priority:** `Must`
* **User Story:** As a customer, I want to inquire about my bill before making a payment, so that I can know the amount due, the applicable service fee, and the total amount I need to pay.
* **Acceptance Criteria:**
  * **Given** that a customer selects a biller that supports live inquiry and provides the required billing information, **when** the customer submits an inquiry request, **then** the system sends the inquiry to the selected biller.
  * **Given** that the biller returns a valid bill amount, **when** the inquiry is completed successfully, **then** the system displays the amount due to the customer.
  * **Given** that the bill amount is retrieved successfully, **when** the system calculates the applicable service fee, **then** it displays the fee separately.
  * **Given** that the bill amount and service fee are calculated, **when** the inquiry result is displayed, **then** the system shows the total payment amount before the customer proceeds with payment.
  * **Given** that a selected biller does not support live inquiry, **when** the customer attempts to perform a live inquiry, **then** the system follows the supported integration mechanism instead of assuming that a live inquiry is available.

### FR-9: Pay a Bill
* **Priority:** `Must`
* **User Story:** As a customer, I want to pay my utility bills using my PayLink wallet, so that I can complete my bill payments digitally without visiting utility offices.
* **Acceptance Criteria:**
  * **Given** that a customer has completed the bill inquiry process, **when** the customer confirms the payment, **then** the system initiates a payment request using the customer's wallet.
  * **Given** that a payment request is initiated, **when** the system performs the required checks, **then** it verifies the customer's available funds and applicable payment limits.
  * **Given** that the customer passes the required checks and has sufficient funds, **when** the payment proceeds, **then** the system follows the defined payment lifecycle.
  * **Given** that the customer has insufficient available funds, **when** the payment is processed, **then** the system rejects the payment.
  * **Given** that the biller successfully confirms the payment, **when** the system receives the confirmation, **then** the payment is marked as `Completed`.

### FR-10: Idempotent Payment Requests
* **Priority:** `Must`
* **User Story:** As a customer, I want repeated payment requests with the same idempotency key to return the original result, so that network problems or accidental repeated requests do not cause duplicate payments.
* **Acceptance Criteria:**
  * **Given** that a payment request has been submitted with a unique idempotency key, **when** the same request is submitted again using the same key, **then** the system returns the original payment result.
  * **Given** that the same idempotency key is received multiple times, **when** the system processes these requests, **then** only one payment operation is executed.
  * **Given** that a payment has already been processed using a specific idempotency key, **when** another identical request with that key is received, **then** the system must not charge the customer again.

### FR-11: Payment Lifecycle Management
* **Priority:** `Must`
* **User Story:** As a system, I want every payment to follow the predefined payment lifecycle and allow only valid state transitions, so that payment operations remain consistent and unauthorized state changes are prevented.
* **Acceptance Criteria:**
  * **Given** that a payment is in the `Initiated` state, **when** fraud checks pass and the customer's wallet has sufficient funds, **then** the payment state changes to `FundsHeld`.
  * **Given** that a payment is in the `Initiated` state, **when** fraud checks fail or the customer's available funds are insufficient, **then** the payment state changes to `Rejected`.
  * **Given** that a payment is in the `FundsHeld` state, **when** the payment request is sent to the biller, **then** the payment state changes to `SentToBiller`.
  * **Given** that a payment is in the `SentToBiller` state, **when** the biller confirms the payment, **then** the payment state changes to `Completed`.
  * **Given** that a payment is in the `SentToBiller` state, **when** the biller refuses the payment, **then** the payment state changes to `Failed`.
  * **Given** that a payment is in the `SentToBiller` state, **when** the biller does not respond within the specified timeout, **then** the payment state changes to `Unknown`.
  * **Given** that a payment is in the `Unknown` state, **when** the biller later confirms the payment through a callback, retry, or settlement file, **then** the payment state changes to `Completed`.
  * **Given** that a payment is in the `Unknown` state, **when** the biller later refuses the payment or reconciliation confirms that no payment was received, **then** the payment state changes to `Failed`.
  * **Given** that a payment is in the `Failed` state, **when** the held funds are released to the customer's wallet, **then** the payment state changes to `Refunded`.
  * **Given** that a payment has reached `Completed`, `Rejected`, or `Refunded`, **when** another state transition is requested, **then** the system rejects the transition because these are final states.

### FR-12: Hold and Release Funds
* **Priority:** `Must`
* **User Story:** As a system, I want to hold the required payment amount before contacting the biller and release the held funds if the payment fails, so that customers' money is protected during payment processing.
* **Acceptance Criteria:**
  * **Given** that a customer requests a payment and passes the required checks with sufficient available funds, **when** the system prepares the payment request, **then** it moves the required amount from the available balance to the held balance before contacting the biller.
  * **Given** that the payment is successfully confirmed by the biller, **when** the system completes the payment, **then** the held amount is converted into a final charge.
  * **Given** that the biller refuses the payment and the payment becomes `Failed`, **when** the held funds are released, **then** the corresponding amount is returned to the customer's available wallet balance and the payment becomes `Refunded`.
  * **Given** that a payment is in the `Unknown` state, **when** the system has not confirmed whether the biller received the payment, **then** the system keeps the funds held and does not automatically refund them.

### FR-13: Handle Payment Timeouts
* **Priority:** `Must`
* **User Story:** As a system, I want to handle biller timeouts by marking uncertain payments as Unknown and retrying the inquiry according to predefined rules, so that payment results can eventually be confirmed without incorrectly returning customers' money.
* **Acceptance Criteria:**
  * **Given** that a payment request has been sent to a live biller, **when** the biller does not respond within **10 seconds**, **then** the system changes the payment state to `Unknown`.
  * **Given** that a payment becomes `Unknown`, **when** the first retry is scheduled, **then** the system re-queries the biller after **1 minute**.
  * **Given** that the payment remains `Unknown` after the first retry, **when** the second retry is scheduled, **then** the system re-queries the biller after **5 minutes**.
  * **Given** that the payment remains `Unknown` after the second retry, **when** the third retry is scheduled, **then** the system re-queries the biller after **15 minutes**.
  * **Given** that all three retry attempts have been completed without a confirmed result, **when** the system finishes the retry process, **then** the payment waits for reconciliation.
  * **Given** that a payment is in the `Unknown` state, **when** the system is waiting for a response or reconciliation result, **then** the held funds remain reserved until the payment outcome is confirmed.

---

## 4. Identified Ambiguities and Proposed Decisions

During the analysis of the functional requirements, several points were identified that were not fully specified in the original project brief. These ambiguities were documented to avoid making unsupported assumptions during requirements definition and implementation.

| ID | Related Requirement | Identified Ambiguity / Question | Proposed Decision |
| :--- | :--- | :--- | :--- |
| **A-01** | `FR-1` | What registration information is mandatory when creating a new customer account? | The required registration fields should be explicitly defined before implementation. |
| **A-02** | `FR-2` | What is the exact process for upgrading a customer from Basic to Verified? Is approval required? | The verification and approval mechanism requires clarification. The requirements only specify the resulting verification level and payment limits. |
| **A-03** | `FR-3` | How many transactions should be displayed in the transaction history, and should filtering or pagination be supported? | The basic requirement is to display recorded wallet transactions. Pagination and filtering can be specified separately. |
| **A-04** | `FR-4` | What happens when a simulated card top-up or agent cash top-up fails? | A failed top-up must not increase the wallet balance. Detailed failure handling requires further specification. |
| **A-05** | `FR-6` | What happens to existing services when an administrator disables a biller? | Disabled billers and services should not be available for new payment operations. The handling of existing transactions requires clarification. |
| **A-06** | `FR-7` | Should disabled billers and services appear in the customer catalog? | Only active billers and services should be offered as available payment options. |
| **A-07** | `FR-8` | Do all billers support live bill inquiry? | No. Live inquiry should only be used for supported billers. NetCo uses a daily batch CSV process and does not support live inquiry. |
| **A-08** | `FR-9` | Can customers cancel a payment after it has been submitted to the biller? | The original brief does not define customer-initiated payment cancellation. No cancellation behavior should be assumed without clarification. |
| **A-09** | `FR-10` | What happens if the same idempotency key is reused with different payment information, such as a different amount or biller? | The system should not execute a second payment using the same key with modified request data. The exact response and error behavior should be defined by the team. |
| **A-10** | `FR-11` | Are there any payment state transitions other than those explicitly listed in Section 2.5? | No additional transitions should be introduced. The payment lifecycle must follow the states and transitions defined in the brief. |
| **A-11** | `FR-12` | How should the system handle a payment that remains Unknown for an extended period? | The funds must remain held until reconciliation or another reliable confirmation establishes the actual payment outcome. |
| **A-12** | `FR-13` | What happens if all three retry attempts fail to obtain a response? | The payment waits for reconciliation, and the held funds must not be automatically refunded. |