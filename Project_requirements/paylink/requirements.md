1- Problem Summary:

PayLink is a digital wallet and bill-payment backend built for the Egyptian market (modeled on services like Fawry and Vodafone Cash) where customers and shop agents pay utility bills such as electricity, water, mobile top-up, and internet in EGP. Because the platform holds customer funds, charges service fees, and pays commissions to agents, financial accuracy is critical: balances must be derived from ledger entries rather than overwritten directly, funds must be placed on hold before calling a biller, and duplicate requests or race conditions must never double-charge a wallet or push a balance below zero.   The system also has to handle real-world unreliability and integration differences cleanly. Different billers use different protocols, speeds, and formats—such as fast REST/JSON for electricity (ElectroCo), slower SOAP-style XML for water (AquaCo), asynchronous callbacks for mobile top-ups (TopUpNet), and daily batch files for internet (NetCo)—and ElectroCo later introduces a breaking v2 API that must be absorbed inside its adapter without changing the rest of the system. Payments must follow a strict state machine (Initiated, FundsHeld, Rejected, SentToBiller, Completed, Failed, Unknown, and Refunded) where timeouts move payments to Unknown with scheduled retries and never auto-refund until biller non-receipt is confirmed, alongside fraud/velocity checks, retries, notifications, audit logs, and daily settlement reconciliation.
-------------------------------------------------------

2- List of Stakeholders

1-Customers: End users who top up EGP wallets, check bills, pay utility bills, receive notifications, and request refunds.   
2-Agents: Terminal operators who process cash top-ups and customer bill payments to earn commission.   Billers: Utility and telecom companies that provide bill data, receive payments or settlement files, and follow their integration contracts.   
3-Support Employees: 
Staff who look up payment histories, block or unblock accounts, review reconciliation mismatches, and approve refunds.   
4-Administrators / Management: 
Operators and leaders who manage billers, update fees and limits without redeploying, review daily totals, and protect revenue and trust.   
Finance & Compliance / Auditors: 
Reviewers who rely on exact ledger accounting, audit logs, and daily settlement reconciliation to verify financial accuracy.   
5-Engineering Team: 
Developers and leads who build, test, deploy, and maintain the PayLink system across project phases.
-------------------------------------------------------

3- User stories:

1-Accounts and Wallet:
    FR-1: Customer Registration (Priority: Must)
    User Story: As a new customer, I want to register for a PayLink account so that I can use a digital wallet to pay my utility bills.
    Acceptance Criteria:
    Given a visitor provides valid registration details, When they submit the registration request, Then an account and EGP wallet are created with a basic verification level, an available balance of 0.00 EGP, and a held balance of 0.00 EGP.

    FR-2: Account Verification Upgrade (Priority: Should)
    User Story: As a basic-tier customer, I want to upgrade my account to a verified level so that I can access higher daily payment limits.
    Acceptance Criteria:
    Given an authenticated customer with a basic verification level, When their verification upgrade is approved, Then their account verification level updates to verified and their daily payment limit increases from 5,000 EGP to 30,000 EGP

    FR-3: View Wallet Balance and Transaction History (Priority: Must)
    User Story: As a customer, I want to view my wallet's available balance, held balance, and transaction history so that I can track my money and pending payments.
    Acceptance Criteria:
    Given an authenticated customer with a wallet, When they request their wallet details, Then the system returns their exact available balance, held balance, and chronological list of wallet transactions.

    FR-4: Wallet Top-Up (Priority: Must)
    User Story: As a customer, I want to top up my wallet via a simulated payment card or agent cash deposit so that I have funds available to pay bills.
    Acceptance Criteria:
    Given an active customer wallet and a valid positive top-up amount in EGP, When the customer completes a simulated card top-up or an agent deposits cash on their behalf, Then the customer's available balance increases by the exact top-up amount.

    FR-5: Ledger-Computed Wallet Balances (Priority: Must)
    User Story: As a system auditor, I want every wallet balance change to be recorded as an immutable ledger entry from which balances are computed so that money is never overwritten or lost.
    Acceptance Criteria:
    Given any money movement occurs on a wallet (top-up, hold, capture, release, refund, or commission), When the transaction is recorded, Then a corresponding ledger entry is appended and the wallet's available and held balances equal the exact sum of their respective ledger entries.

2-Biller Catalog:
    FR-6: Manage Billers and Services (Priority: Must)
    User Story: As an admin, I want to add, edit, and disable billers and their services so that the catalog reflects active utility providers and required inputs.
    Acceptance Criteria:
    Given an authenticated admin, When they create, update, or disable a biller or service (including required input fields and min/max limits), Then the catalog reflects the changes and disabled billers/services are unavailable for new bill inquiries or payments.

    FR-7: Browse Active Billers and Required Input Fields (Priority: Must)
    User Story: As a customer, I want to view the list of available billers and the input fields each requires so that I know what information to enter to pay my bill.
    Acceptance Criteria:
    Given an authenticated customer, When they request the biller catalog, Then the system returns all active billers and services along with their required input fields (such as meter number or phone number) and allowed payment limits.

3-Payment Processing & Biller Integration:
    FR-8: Bill Inquiry with Fee Breakdown (Priority: Must)
    User Story: As a customer, I want to run a bill inquiry before paying so that I can see the bill amount, PayLink service fee, and total cost.
    Acceptance Criteria:
    Given an active biller and valid customer identifier (such as a meter number), When the customer submits a bill inquiry, Then PayLink queries the biller (for live-inquiry billers) and returns the bill amount due, applicable service fee, and exact total (bill amount + service fee).

    FR-9: Pay a Bill from Wallet (Priority: Must)
    User Story: As a customer, I want to pay a utility bill from my PayLink wallet so that I can settle my bill without visiting a payment office.
    Acceptance Criteria:
    Given an unblocked customer with sufficient available wallet balance, When they confirm a bill payment within single and daily limits, Then PayLink places a hold on the total amount, submits the payment to the biller with a unique reference, finalizes the charge upon confirmation, and credits PayLink's fee income.

    FR-10: Idempotent Payment Requests (Priority: Must)
    User Story: As a mobile app client, I want repeated payment requests carrying the same idempotency key to return the original result so that network retries never double-charge a customer.
    Acceptance Criteria:
    Given a payment request has already been processed with a specific idempotency key, When a duplicate request arrives with the same idempotency key, Then the system does not create a new payment or hold additional funds, and returns the exact response/state of the original payment.

    FR-11: Strict Payment State Machine (Priority: Must)
    User Story: As a backend engineer, I want every payment to follow only the defined state life cycle so that invalid state transitions or premature refunds cannot occur.
    Acceptance Criteria:
    Given a payment in a valid state (Initiated, FundsHeld, Rejected, SentToBiller, Completed, Failed, Unknown, or Refunded), When an event occurs, Then the payment transitions strictly according to the permitted transition table (rejecting any illegal transition and never moving directly from Unknown to Refunded).

    FR-12: Fund Reservation (Hold) and Release on Failure (Priority: Must)
    User Story: As a customer, I want my funds to be held before calling the biller and released back to my available balance if the payment fails so that I never lose money on failed transactions.
    Acceptance Criteria:
    Given a payment passes fraud and balance checks, When the payment moves to FundsHeld and subsequently Failed (due to biller refusal or confirmed non-receipt), Then the held funds are released back to the customer's available balance and the payment transitions to Refunded.

    FR-13: Biller Timeout Handling and Unknown Payment Retries (Priority: Must)
    User Story: As a customer, I want a payment that times out at the biller to be marked Unknown and automatically re-queried so that delayed responses are resolved accurately without double-charging me
    Acceptance Criteria:
    Given a live biller does not respond within the configured 10-second timeout, When the timeout triggers, Then the payment transitions to Unknown with funds remaining held, and the system re-queries the biller up to 3 times (after 1, 5, and 15 minutes) before deferring to nightly reconciliation.

    FR-14: Asynchronous Biller Callbacks with Deduplication (Priority: Must)
    User Story: As a payment service, I want to accept asynchronous callbacks from billers like TopUpNet and ignore duplicates so that delayed payment outcomes settle cleanly once.
    Acceptance Criteria:
    Given an asynchronous payment in SentToBiller or Unknown state, When TopUpNet sends a "success" or "failed" callback, Then the payment transitions to Completed (capturing held funds) or Failed (releasing held funds), and any subsequent duplicate callback for the same payment is logged and ignored without re-applying ledger changes.

    FR-15: Multi-Biller Protocol Support (Priority: Must)
    User Story: As a platform architect, I want to integrate with multiple billers using distinct protocols and formats behind uniform adapters so that adding or updating a biller isolates protocol changes.
    Given PayLink is configured with ElectroCo (REST/JSON), AquaCo (SOAP-style XML over HTTP), and TopUpNet (async REST/JSON with callbacks) — plus optional NetCo (daily batch CSV), When payments are routed to each biller, Then each adapter translates requests, responses, and errors cleanly without leaking biller-specific formats into core payment processing.

4-Fees and Commissions:
    FR-16: Configurable Fee Calculation and Promotions (Priority: Must)
    User Story: As a customer, I want service fees to follow PayLink's pricing rules and active promotions so that I am charged accurately for every utility type.
    Acceptance Criteria:
    Given a bill payment for a specific utility type and date, When the fee is calculated, Then the system applies 2.50 EGP fixed for electricity/water, 1.5% (minimum 1.00 EGP) for mobile top-up, 2.00 EGP up to 200 EGP or 4.00 EGP above 200 EGP for internet, or 0.00 EGP if the biller is within its configured promotion window.

    FR-17: Agent-Assisted Payments and Commission (Priority: Should)
    User Story: As a PayLink agent, I want to pay a bill on behalf of a customer and earn a commission on completed payments so that I am compensated for operating a terminal.
    Acceptance Criteria:
    Given an authenticated agent initiates a bill payment for a customer, When the payment reaches Completed, Then a commission equal to 40% of the collected service fee is credited to the agent's wallet via a ledger entry.

5-Fraud Checks & Support Operations:
    FR-18: Single-Payment and Daily Limit Enforcement (Priority: Must)
    User Story: As a risk manager, I want payments exceeding single-transaction or daily limits to be rejected with a clear reason so that account exposure is controlled.
    Acceptance Criteria:
    Given a payment amount below 5 EGP, above 10,000 EGP, or causing the customer's daily total to exceed 5,000 EGP (basic) or 30,000 EGP (verified), When fraud checks execute, Then the payment transitions from Initiated to Rejected with a descriptive error reason and no funds are held.

    FR-19: Payment Velocity Block (Priority: Should)
    User Story: As a fraud prevention system, I want to block an account for 10 minutes if more than 5 payments are attempted within 60 seconds so that automated abuse is stopped.
    Acceptance Criteria:
    Given a customer account has already made 5 payment attempts in the trailing 60 seconds, When a 6th payment is submitted within that 60-second window, Then the payment is Rejected and the account is automatically blocked for 10 minutes.

    FR-20: Support Payment Search and History Lookup (Priority: Must)
    User Story: As a support employee, I want to search for a payment by its unique reference and view its full state and audit history so that I can assist customers with inquiries.
    Acceptance Criteria:
    Given an authenticated support employee enters a payment reference, When they execute the search, Then the system returns the payment details, current state, state transition timestamps, biller interaction logs, and related ledger entries.

    FR-21: Support Account Block and Unblock (Priority: Should)
    User Story: As a support employee, I want to manually block and unblock customer accounts so that I can respond to suspicious activity or resolve customer lockouts.
    Acceptance Criteria:
    Given an authenticated support employee, When they block an active account (or unblock a blocked account), Then the account status updates immediately, blocked accounts cannot initiate payments, and the action is recorded in the audit log.

6-Notifications:
    FR-22: Payment Outcome SMS and Push Notifications (Priority: Must)
    User Story: As a customer, I want to receive an SMS and a push notification when my payment completes or fails so that I have immediate confirmation of my transaction status.
    Acceptance Criteria:
    Given a payment transitions to Completed or Failed, When the outcome event is processed, Then the notification service sends both an SMS and a push notification containing the payment reference, amount, and status (including failure reason if failed).

    FR-23: Notification Retry and Undelivered Status (Priority: Should)
    User Story: As a platform operator, I want failed SMS or email notifications to be retried 3 times and then marked undelivered without affecting payment completion so that transient provider outages do not break payments.
    Acceptance Criteria:
    Given an SMS or email notification fails to send due to a provider error, When the notification service handles the failure, Then it retries delivery up to 3 times, marks the notification as undelivered if all retries fail, and leaves the associated payment's Completed state intact.

7-Administration, Audit & Reconciliation
    FR-24: Runtime Configuration of Fees and Limits (Priority: Should)
    User Story: As an admin, I want to update service fees, promotion dates, and transaction limits without redeploying the application so that business rules can adapt quickly.
    Acceptance Criteria:
    Given an authenticated admin, When they update fee values, promotion windows, or daily/single payment limits in configuration/settings, Then subsequent bill inquiries and payments immediately apply the new values without requiring an application restart or redeployment.

    FR-25: Comprehensive Audit Logging (Priority: Must)
    User Story: As a compliance officer, I want every money movement and administrative action recorded in an immutable audit log so that all financial and configuration changes are traceable.
    Acceptance Criteria:
    Given any ledger entry is created or any admin/support action is performed (such as editing billers, updating fees/limits, blocking/unblocking accounts, or approving refunds), When the operation completes, Then an audit log entry is persisted capturing who performed the action, when it occurred, the affected entity, and the reason/details.

    FR-26: Daily Settlement Reconciliation Report (Priority: Should)
    User Story: As a finance operator, I want the system to compare PayLink's daily payment records against a biller's settlement file and generate a difference report so that discrepancies are flagged for review.
    Acceptance Criteria:
    Given a biller's daily settlement file is ingested, When the reconciliation job compares it against PayLink's recorded payments for that period, Then matching Unknown payments are resolved and any mismatches (such as a payment marked Failed in PayLink but listed as paid in the settlement file, or missing records) are flagged in the reconciliation report.

    FR-27: Customer Refund Request and Support Approval (Priority: Could)
    User Story: As a customer, I want to request a refund for a completed payment and have a support employee approve it so that disputed or erroneous charges can be returned to my wallet.
    Acceptance Criteria:
    Given a payment in Completed state, When the customer submits a refund request and a support employee approves it, Then the refunded amount is credited back to the customer's available wallet balance via a ledger entry, the payment status updates to Refunded, and an audit record is created.
    
    FR-28: Admin Daily Totals Dashboard (Priority: Could)
    User Story: As an admin, I want to view daily aggregated totals for payments, fees earned, commissions paid, and failures per biller so that I can monitor business performance and biller health.
    Acceptance Criteria:
    Given an authenticated admin queries daily reporting metrics for a target date, When the report is generated, Then the system returns total payment volume/count, total service fees earned, total agent commissions paid, and failure counts grouped by biller.
-------------------------------------------------------

4- Non-Functional Requirements:

1. Correctness (Concurrency & Idempotency)
Target: Zero double charges and zero negative balances in concurrency and duplicate tests.

2. Performance (Throughput)Target: 100 payments per second sustained for 1 minute against mock billers. 

3. Resilience (Fault Isolation)
Target: A biller outage does not stop payments to other billers.

4. Modifiability (Extensibility)
Target: Adding a biller touches only the Biller Integration module and configuration.

5. Auditability (Traceability)Target: 100% of money movements have an audit record with who, when, and why.   

6. Testability (Code Coverage)Target: At least 70% line coverage on domain logic, measured in CI. 
-------------------------------------------------------

5- Technical Requirements:

1-Language and framework: Node Nestjs

2-Database: a relational database with at least 10 tables, schema migrations stored in
the repository.

3-API: at least 20 REST endpoints documented with OpenAPI, plus one asynchronous
flow (a queue, or events inside the application first, then through a broker in Phase 2).

4-Design patterns: at least 5 different patterns applied for a real reason. Suggested:
Adapter and Factory (billers), Strategy (fees), State (payment), Chain of
Responsibility (fraud checks), Observer (notifications), Command (retries).

5-Architecture: design a layered or hexagonal modular monolith in Phase 1 and build it
first in Phase 2; then evolve at least Notifications and Reconciliation to an eventdriven or service-based style, with an architecture decision record explaining the
choice and the alternatives.

6-Docker: a Dockerfile for each runnable component and one docker compose file that
starts the application, the database and the mock billers.

7-CI/CD: a GitHub Actions workflow that builds, lints, tests, reports coverage, builds
images and deploys the stack locally.
-------------------------------------------------------