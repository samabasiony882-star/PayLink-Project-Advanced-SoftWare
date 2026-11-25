# مخططات نظام PayLink (UML Diagrams in Mermaid)

يحتوي هذا الملف على جميع مخططات النظام بصيغة **Mermaid** الجاهزة للعرض المباشر على GitHub أو Notion أو Mermaid Live Editor:
1. **Use Case Diagram:** مخطط حالات الاستخدام لجميع الأدوار.
2. **Sequence Diagram (Pay a Bill):** مخطط التسلسل لسيناريو دفع الفاتورة.
3. **Sequence Diagram (Failed Payment Release):** مخطط التسلسل لسيناريو استرجاع مبلغ عملية فاشلة.
4. **State Diagram:** مخطط الحالات لدورة حياة الدفع والانتقالات المسموحة فقط.

---

## 1. Use Case Diagram (مخطط حالات الاستخدام لجميع الأدوار)

```mermaid
flowchart LR
    %% Primary Actors (Left)
    Customer(["Customer"])
    Agent(["Agent"])
    Admin(["Administrator"])
    Support(["Support Employee"])

    %% Secondary Actors (Right)
    Biller(["Biller (Utility Company)"])
    Notif(["Notification Provider"])

    subgraph PayLink_System["PayLink System"]
        %% Customer Use Cases
        UC1("Register Account")
        UC2("Upgrade Verification Level")
        UC3("View Wallet & History")
        UC4("Top-up Wallet")
        UC5("View Available Billers")
        UC6("Bill Inquiry")
        UC7("Pay a Bill")
        UC8("Request Refund (Completed Payment)")

        %% Included / Supporting Steps
        UC_Check("Verify Limits & Velocity")
        UC_Hold("Hold Funds")
        UC_Fee("Calculate Fee")

        %% Agent Use Cases
        UC9("Agent Cash Top-up")
        UC10("Pay Bill on Behalf of Customer")

        %% Admin Use Cases
        UC11("Manage Billers & Services")
        UC12("Configure Fees & Limits")
        UC13("View Admin Daily Totals")

        %% Support Use Cases
        UC14("Search Payment & History")
        UC15("Block / Unblock Account")
        UC16("Approve / Reject Refund Request")
        UC17("Review Daily Reconciliation Report")
    end

    %% Primary Actor Associations
    Customer --> UC1
    Customer --> UC2
    Customer --> UC3
    Customer --> UC4
    Customer --> UC5
    Customer --> UC6
    Customer --> UC7
    Customer --> UC8

    Agent --> UC9
    Agent --> UC10

    Admin --> UC11
    Admin --> UC12
    Admin --> UC13

    Support --> UC14
    Support --> UC15
    Support --> UC16
    Support --> UC17

    %% Include & Extend Relationships
    UC7 -. "«include»" .-> UC_Check
    UC7 -. "«include»" .-> UC_Hold
    UC7 -. "«include»" .-> UC_Fee
    UC10 -. "«include»" .-> UC7
    UC9 -. "«extend»" .-> UC4

    %% Secondary Actor Associations
    UC6 --> Biller
    UC7 --> Biller
    UC10 --> Biller
    UC7 --> Notif
    UC16 --> Notif
```

---

## 2. Sequence Diagram — سيناريو 1: دفع الفاتورة (Pay a Bill)

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant PaymentService
    participant FraudLimitEngine
    participant WalletLedger
    participant BillerAdapter
    participant Biller
    participant NotificationService

    Customer->>PaymentService: payBill(billerId, billInfo, idempotencyKey)
    PaymentService->>PaymentService: Check idempotencyKey (FR-10)

    alt Duplicate Key
        PaymentService-->>Customer: Return original payment result
    else New Payment
        PaymentService->>PaymentService: Create payment [Initiated]
        PaymentService->>FraudLimitEngine: Check status, velocity & limits (FR-18, FR-19)

        alt Checks Fail (Blocked / Limit / Velocity / Low Funds)
            PaymentService->>PaymentService: Transition state -> Rejected
            PaymentService-->>Customer: Reject payment with reason
        else Checks Pass
            PaymentService->>WalletLedger: Hold funds (Available -> Held) + Ledger & Audit (FR-5, FR-12, FR-25)
            PaymentService->>PaymentService: Transition state -> FundsHeld

            PaymentService->>BillerAdapter: Forward payment request
            PaymentService->>PaymentService: Transition state -> SentToBiller
            BillerAdapter->>Biller: Send payment request

            alt Biller Confirms
                Biller-->>BillerAdapter: Confirmed
                BillerAdapter-->>PaymentService: Success
                PaymentService->>WalletLedger: Convert Held to Final Charge (+ Agent commission if applicable)
                PaymentService->>PaymentService: Transition state -> Completed
                PaymentService->>NotificationService: Send SMS & Push confirmation (FR-22)
                PaymentService-->>Customer: Payment Completed
            else Biller Refuses
                Biller-->>BillerAdapter: Refused / Error code
                BillerAdapter-->>PaymentService: Refused
                PaymentService->>PaymentService: Transition state -> Failed
                PaymentService->>WalletLedger: Release Held funds -> Available + Ledger & Audit
                PaymentService->>PaymentService: Transition state -> Refunded
                PaymentService->>NotificationService: Send SMS & Push failure notice (FR-22)
                PaymentService-->>Customer: Payment Failed & Funds Released
            else Timeout (> 10s) or Async Ack
                BillerAdapter-->>PaymentService: Timeout / Pending
                PaymentService->>PaymentService: Transition state -> Unknown (Keep funds Held, FR-13)
                PaymentService->>PaymentService: Schedule retries (1m, 5m, 15m) & reconciliation
                PaymentService-->>Customer: Payment is being processed
            end
        end
    end
```

---

## 3. Sequence Diagram — سيناريو 2: استرجاع مبلغ عملية فاشلة (Failed Payment Release)

```mermaid
sequenceDiagram
    autonumber
    participant BillerOrReconciler
    participant PaymentService
    participant WalletLedger
    participant AuditLog
    participant NotificationService
    actor Customer

    BillerOrReconciler->>PaymentService: Report payment failure (from SentToBiller or Unknown)
    PaymentService->>PaymentService: Transition state -> Failed (FR-11)

    PaymentService->>WalletLedger: Release held amount back to Available balance (FR-12)
    WalletLedger->>WalletLedger: Record RELEASE ledger entry (exact decimal, FR-5)
    WalletLedger-->>PaymentService: Funds released

    PaymentService->>AuditLog: Write immutable audit entry (who: System, when, why: Failed release, FR-25)
    PaymentService->>PaymentService: Transition state -> Refunded (Final State)

    PaymentService->>NotificationService: Send SMS & Push notification with failure reason (FR-22)
    alt Notification Fails
        NotificationService->>NotificationService: Retry up to 3 times -> mark Undelivered if all fail (FR-23)
    end
    NotificationService-->>Customer: Deliver failure & release notification
```

---

## 4. State Diagram — حالات الدفع والانتقالات المسموحة فقط (Payment Lifecycle)

```mermaid
stateDiagram-v2
    [*] --> Initiated : Payment request created

    Initiated --> FundsHeld : Fraud & limit checks pass + sufficient balance
    Initiated --> Rejected : Fraud/limit check fails or insufficient funds

    FundsHeld --> SentToBiller : Payment request sent to biller

    SentToBiller --> Completed : Biller confirms payment
    SentToBiller --> Failed : Biller refuses payment
    SentToBiller --> Unknown : Timeout (>10s) or async ack waiting callback

    Unknown --> Completed : Confirmed via callback, retry, or reconciliation
    Unknown --> Failed : Refused or reconciliation confirms not received

    Failed --> Refunded : Held funds released to customer wallet

    Completed --> [*]
    Rejected --> [*]
    Refunded --> [*]
```