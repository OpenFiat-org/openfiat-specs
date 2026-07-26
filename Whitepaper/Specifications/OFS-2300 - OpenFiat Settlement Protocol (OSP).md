# OFS-2300 — Settlement Protocol (OSP)

**Document ID:** OFS-2300

**Title:** OpenFiat Settlement Protocol

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Trading

**Depends On:** OFS-2000, OFS-2200

---

# Abstract

The OpenFiat Settlement Protocol (OSP) defines how reserved trades are completed after escrow has been created.

It specifies the complete settlement lifecycle, including fiat payment, payment confirmation, merchant verification, escrow release, trade completion, settlement recovery, and failure handling.

The protocol guarantees that stablecoins are released only when settlement conditions have been satisfied while preserving deterministic behavior across the decentralized OpenFiat network.

---

# 1. Introduction

Settlement is the most security-critical phase of an OpenFiat trade.

During settlement:

* Fiat changes hands.
* Stablecoins remain safely locked.
* Both participants exchange evidence.
* The merchant verifies payment.
* The smart contract releases escrow.

Settlement begins immediately after reservation succeeds.

It ends only when escrow is either:

* Released
* Refunded
* Redirected through dispute resolution

---

# 2. Scope

This specification defines:

* Settlement lifecycle
* Fiat payment
* Payment confirmation
* Payment proof
* Merchant verification
* Escrow release
* Settlement timeout
* Settlement cancellation
* Settlement recovery

This specification does not define:

* Reservations
* Disputes
* Reputation
* Notification delivery

---

# 3. Design Goals

The Settlement Protocol SHALL:

* Guarantee escrow safety.
* Prevent unauthorized releases.
* Preserve deterministic settlement state.
* Recover from failures.
* Minimize settlement time.
* Support multiple fiat payment methods.

---

# 4. Settlement Philosophy

Settlement separates:

**Movement of fiat**

from

**Movement of stablecoins.**

The blockchain secures the stablecoins.

Traditional financial systems move the fiat.

The protocol coordinates both worlds without requiring custody of either.

---

# 5. Settlement Lifecycle

```text id="settlement-lifecycle"
Reservation Accepted

↓

Escrow Locked

↓

Buyer Pays Fiat

↓

Buyer Marks "I Paid"

↓

Merchant Reviews

↓

Settlement Approved

↓

Escrow Released

↓

Trade Completed
```

Every transition produces a signed protocol event.

---

# 6. Escrow State

Before settlement begins:

* Escrow already exists.
* Funds are locked.
* Advertisement inventory has already been updated.
* Reservation is exclusive.

No additional escrow operations are required during settlement.

---

# 7. Fiat Payment

The payer submits fiat using one of the merchant's advertised payment methods.

Supported methods include:

* Bank Transfer
* Mobile Money
* ACH
* SEPA
* Faster Payments
* PIX
* Cash Deposit
* Regional Instant Payment Networks

The protocol does not process fiat.

It coordinates settlement.

---

# 8. Payment Deadline

Each settlement defines a payment deadline.

Example:

```text id="payment-deadline"
Reservation

↓

Payment Window

20 Minutes

↓

Expired

↓

Reservation Cancelled
```

Timeout values are configurable through governance.

---

# 9. "I Paid" Event

Once fiat has been sent:

The payer selects:

> **I Paid**

This creates a signed Payment Submitted event.

"I Paid" represents a declaration by the payer.

It does **not** release escrow.

---

# 10. Reversing "I Paid"

Until settlement has been approved, the payer MAY withdraw the "I Paid" declaration.

Reasons include:

* Incorrect payment amount.
* Wrong recipient.
* Duplicate submission.
* User mistake.

Every reversal generates a new signed protocol event.

Applications SHALL display the latest valid payment state.

---

# 11. Payment Proof

Payment submissions MAY include evidence.

Supported examples include:

* Bank receipt
* Mobile money confirmation
* Transaction reference
* PDF receipt
* Image attachment
* Screenshot

Evidence becomes permanently associated with the settlement record.

---

# 12. Merchant Verification

After receiving the Payment Submitted event, the merchant reviews:

* Incoming payment
* Payment proof
* Reference number
* Payment amount
* Sender information

Merchants MAY request additional information before settlement.

---

# 13. Additional Verification

Merchants MAY request:

* Updated receipt
* Bank reference
* Payment clarification
* Sender confirmation
* Additional documentation

Every request generates a synchronized protocol event.

---

# 14. Automatic Settlement

Future protocol versions MAY support trusted payment verification providers.

Examples include:

* Banking APIs
* Mobile money APIs
* Certified payment providers

When available, merchants MAY opt into automatic settlement.

Automatic settlement remains optional.

---

# 15. Settlement Approval

When payment has been successfully verified:

```text id="approval-flow"
Merchant Approves

↓

Settlement Approved Event

↓

Program Releases Escrow

↓

Trade Completed
```

The merchant never manually transfers stablecoins.

Only the OpenFiat Program releases escrow.

---

# 16. Escrow Release

Escrow release is performed exclusively by the OpenFiat Program.

The program verifies:

* Reservation validity
* Escrow state
* Settlement approval
* Trade integrity

Upon successful verification:

Stablecoins are transferred to the receiving wallet.

---

# 17. Settlement Failure

Settlement may fail due to:

* Payment timeout
* Invalid payment
* Escrow inconsistency
* User cancellation
* Smart contract failure
* Network interruption

Failed settlements generate deterministic failure events.

---

# 18. Settlement Recovery

If infrastructure fails during settlement:

```text id="settlement-recovery"
Node Failure

↓

Session Recovery

↓

Settlement State Loaded

↓

Continue Settlement
```

Settlement survives:

* Browser refreshes
* Node restarts
* Session migration
* Network interruptions

No completed work is lost.

---

# 19. Settlement Cancellation

Settlement MAY terminate before completion when:

* Reservation expires.
* Payment window expires.
* Escrow creation failed.
* User withdraws before payment.
* Merchant rejects payment.

Cancellation returns the trade to a deterministic terminal state.

---

# 20. Settlement State Machine

```text id="settlement-state-machine"
Escrow Locked

↓

Awaiting Payment

↓

Payment Submitted

↓

Merchant Reviewing

↓

Approved

↓

Escrow Released

↓

Completed

or

Rejected

or

Cancelled

or

Disputed
```

Only valid state transitions are permitted.

---

# 21. Settlement Synchronization

Settlement events include:

* PaymentSubmitted
* PaymentReversed
* VerificationRequested
* SettlementApproved
* SettlementRejected
* EscrowReleased
* SettlementCompleted

Events propagate through OFS-1200.

---

# 22. Settlement Notifications

Settlement SHOULD generate notifications.

Buyer:

* Payment received for review
* Additional information requested
* Escrow released
* Settlement completed

Merchant:

* Buyer marked "I Paid"
* Payment proof uploaded
* Payment deadline approaching
* Settlement completed

Notification delivery is specified in OFS-6000.

---

# 23. Settlement Metrics

Settlement metrics contribute to merchant reputation.

Metrics include:

* Average settlement time
* Approval rate
* Response speed
* Successful settlements
* Timeout frequency
* Settlement cancellations

These metrics feed into OFS-3000.

---

# 24. Security Considerations

Implementations MUST prevent:

* Unauthorized escrow release
* Duplicate settlement
* Replay attacks
* Forged payment confirmations
* Invalid settlement approvals
* Tampered payment evidence

All settlement events MUST be digitally signed.

---

# 25. Performance Considerations

Settlement is among the highest-priority operations within the network.

Implementations SHOULD optimize:

* Low-latency event propagation
* Fast merchant notifications
* Efficient state recovery
* Immediate escrow release

Settlement traffic SHOULD receive high-priority scheduling under OFS-1600.

---

# 26. Conformance

A compliant implementation MUST:

* Support signed settlement events.
* Support payment proof attachments.
* Allow "I Paid" reversal before approval.
* Support merchant verification requests.
* Ensure only the OpenFiat Program releases escrow.
* Support deterministic settlement state.
* Support settlement recovery.
* Support settlement cancellation.
* Synchronize settlement events through OFS-1200.

---

# 27. Relationship to Other Specifications

The Settlement Protocol governs the final execution phase of every OpenFiat trade.

```text id="settlement-architecture"
              OFS-2100
      Advertisement Protocol
                    │
                    ▼
              OFS-2200
      Reservation Protocol
                    │
                    ▼
              OFS-2300
       Settlement Protocol
                    │
        ┌───────────┼────────────┐
        ▼           ▼            ▼
 Escrow Release  OFS-2400   OFS-3000
                 Disputes   Reputation
```

The Settlement Protocol answers one critical question:

**"How does a decentralized marketplace safely exchange off-chain fiat for on-chain stablecoins?"**

By coordinating fiat payment, cryptographically authenticated settlement events, merchant verification, and automatic smart contract escrow release, the OpenFiat Settlement Protocol enables secure, deterministic, and interoperable settlement without requiring a centralized escrow agent or marketplace operator.
