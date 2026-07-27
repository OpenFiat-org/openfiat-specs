# OFS-2300 — Settlement Protocol (OSP)

**Document ID:** OFS-2300

**Title:** OpenFiat Settlement Protocol

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Trading

**Depends On:** OFS-2000, OFS-2200

---

## Abstract

The OpenFiat Settlement Protocol (OSP) defines how reserved trades are completed after escrow has been created.

It specifies the complete settlement lifecycle, including fiat payment, payment confirmation, merchant verification, escrow release, trade completion, settlement recovery, and failure handling.

The protocol guarantees that stablecoins are released only when settlement conditions have been satisfied while preserving deterministic behavior across the decentralized OpenFiat network.

---

## 1. Introduction

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

## 2. Scope

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

## 3. Design Goals

The Settlement Protocol SHALL:

* Guarantee escrow safety.
* Prevent unauthorized releases.
* Preserve deterministic settlement state.
* Recover from failures.
* Minimize settlement time.
* Support multiple fiat payment methods.

---

## 4. Settlement Philosophy

Settlement separates:

**Movement of fiat**

from

**Movement of stablecoins.**

The blockchain secures the stablecoins.

Traditional financial systems move the fiat.

The protocol coordinates both worlds without requiring custody of either.

---

## 5. Settlement Lifecycle

```text id="settlement-lifecycle"
Escrow Locked (handoff from Reservation Protocol, OFS-2200 §18)

↓

Awaiting Payment

↓

Payment Sent

↓

Merchant Reviewing

↓

Approved

↓

Escrow Released

↓

Completed
```

This is the same state machine as §20's Settlement State Machine, restated here as a lifecycle narrative. `Escrow Locked` is where authority passes from the Reservation Protocol to this specification; this specification does not re-define what happens before that point.

Every transition produces a signed protocol event.

---

## 6. Escrow State

Before settlement begins:

* Escrow already exists.
* Funds are locked.
* Advertisement inventory has already been updated.
* Reservation is exclusive.

No additional escrow operations are required during settlement.

---

## 7. Fiat Payment

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

## 8. Payment Deadline

Each settlement defines a payment deadline.

Example:

```text id="payment-deadline"
Escrow Locked

↓

Payment Window

30 Minutes

↓

Expired

↓

Settlement Cancelled
```

Timeout values are configurable through governance.

## 8a. Timeout Matrix

| Timeout | Phase | Default | Governance-Configurable | On Expiry |
|---|---|---|---|---|
| Payment window | Escrow Locked → Payment Sent | 30 minutes | Yes | Settlement cancelled (§19); escrow returned to seller/vault |
| Merchant review window | Payment Sent → Approved/Rejected | 30 minutes | Yes | Buyer may open a dispute (OFS-2400 §5) |
| Additional-verification window | Merchant requests more info (§13) | 30 minutes from request | Yes | Treated as merchant non-response; buyer may open a dispute |

See OFS-2200 §12a for the reservation-phase timeout matrix and OFS-2400 for dispute-phase timeouts (arbitrator commit/reveal windows).

---

## 9. "I Paid" Event

Once fiat has been sent:

The payer selects:

> **I Paid**

This creates a signed Payment Submitted event.

"I Paid" represents a declaration by the payer.

It does **not** release escrow.

---

## 10. Reversing "I Paid"

Until settlement has been approved, the payer MAY withdraw the "I Paid" declaration.

Reasons include:

* Incorrect payment amount.
* Wrong recipient.
* Duplicate submission.
* User mistake.

Every reversal generates a new signed protocol event.

Applications SHALL display the latest valid payment state.

---

## 11. Payment Proof

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

## 12. Merchant Verification

After receiving the Payment Submitted event, the merchant reviews:

* Incoming payment
* Payment proof
* Reference number
* Payment amount
* Sender information

Merchants MAY request additional information before settlement.

---

## 13. Additional Verification

Merchants MAY request:

* Updated receipt
* Bank reference
* Payment clarification
* Sender confirmation
* Additional documentation

Every request generates a synchronized protocol event.

---

## 14. Automatic Settlement

Future protocol versions MAY support trusted payment verification providers.

Examples include:

* Banking APIs
* Mobile money APIs
* Certified payment providers

When available, merchants MAY opt into automatic settlement.

Automatic settlement remains optional.

---

## 15. Settlement Approval

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

## 16. Escrow Release

Escrow release is performed exclusively by the OpenFiat Program.

The program verifies:

* Reservation validity
* Escrow state
* Settlement approval
* Trade integrity

Upon successful verification:

Stablecoins are transferred to the receiving wallet.

---

## 17. Settlement Failure

Settlement may fail due to:

* Payment timeout
* Invalid payment
* Escrow inconsistency
* User cancellation
* Smart contract failure
* Network interruption

Failed settlements generate deterministic failure events.

---

## 18. Settlement Recovery

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

## 19. Settlement Cancellation

The governing rule for settlement cancellation is:

**Before payment: either party may cancel, subject to protocol rules. After payment is marked sent: cancellation is restricted to prevent abuse.**

"Payment marked sent" refers to the buyer's signed "I Paid" / `PaymentSubmitted` event (§9). Before that event, cancellation is comparatively permissive, since no fiat has yet changed hands and no counterparty can be harmed by unwinding the trade. After that event, an uncontrolled cancellation would let a payer falsely claim payment, cancel, and force the counterparty to chase a fiat reversal — so cancellation authority narrows sharply.

Settlement MAY terminate before completion when:

* Reservation expires.
* Payment window expires.
* Escrow creation failed.
* User withdraws before payment.
* Merchant rejects payment.

## 19a. Cancellation Matrix

| Trigger | Before "I Paid" | After "I Paid" (Payment Sent) |
|---|---|---|
| Buyer-initiated cancellation | Allowed — escrow returned, reservation released | Not allowed unilaterally; buyer may withdraw "I Paid" per §10 if it was submitted in error, but cannot cancel the settlement outright once the merchant has begun reviewing |
| Merchant-initiated cancellation | Allowed where permitted (e.g. cannot fulfil) — escrow returned | Not allowed unilaterally; merchant must either approve, reject (with reason, §12-§13), or the case proceeds to dispute (OFS-2400) |
| Payment window timeout | Allowed (no payment ever marked sent) — automatic settlement cancellation | Not applicable — timeout only applies pre-"I Paid" |
| Merchant review timeout | Not applicable | Buyer may escalate to dispute rather than cancel (OFS-2400 §5) |
| Escrow creation failure | Allowed — automatic cancellation, no reservation was ever fully secured | Not applicable — escrow already existed by this point |
| Mutual agreement | Allowed | Allowed only as a Mutual Settlement dispute outcome (OFS-2400 §17), not a plain cancellation, to keep the record auditable |

Cancellation returns the trade to a deterministic terminal state.

---

## 20. Settlement State Machine

This is the authoritative trade state machine from `Escrow Locked` onward. The Reservation Protocol (OFS-2200 §18) owns every state before this point and its own state machine terminates at `Escrow Locked`; no other document should define settlement-phase states independently.

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

## 21. Settlement Synchronization

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

## 22. Settlement Notifications

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

## 23. Settlement Metrics

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

## 24. Security Considerations

Implementations MUST prevent:

* Unauthorized escrow release
* Duplicate settlement
* Replay attacks
* Forged payment confirmations
* Invalid settlement approvals
* Tampered payment evidence
* Duplicate processing of an already-applied settlement event

All settlement events MUST be digitally signed.

## Idempotency

Every settlement event (`PaymentSubmitted`, `PaymentReversed`, `VerificationRequested`, `SettlementApproved`, `SettlementRejected`, `EscrowReleased`, `SettlementCompleted`) carries a unique nonce. Implementations MUST detect a duplicate nonce and discard the redundant event — for example, a resent `SettlementApproved` for an already-released escrow MUST NOT trigger a second release of funds, and a duplicated `PaymentSubmitted` MUST NOT be treated as two independent payment claims.

---

## 25. Performance Considerations

Settlement is among the highest-priority operations within the network.

Implementations SHOULD optimize:

* Low-latency event propagation
* Fast merchant notifications
* Efficient state recovery
* Immediate escrow release

Settlement traffic SHOULD receive high-priority scheduling under OFS-1600.

---

## 26. Conformance

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

## 27. Relationship to Other Specifications

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
