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

## 5a. The Reservation During Settlement

`Escrow Locked` is a handoff, and a handoff has two sides. OFS-2200 §18 terminates its state machine there and this specification picks it up — but the reservation record does not cease to exist, and while nothing said otherwise it remained in `Escrow Locked` for the whole life of the trade: cancellable by its owner under OFS-2200 §14, and expirable by OFS-2200 §12's validation-window sweep, both of which return the reserved liquidity to the merchant's advertisement.

Either one, performed while a settlement is running, unwinds one half of a trade whose other half continues. The merchant's advertisement is credited liquidity committed to a live settlement; the settlement proceeds to `Approved` and the escrow releases; and a client joining the two records reports a cancelled trade whose funds have moved. The timing is not exotic: §8a's payment window and merchant-review window are thirty minutes each against OFS-2200 §12a's thirty-minute validation window, so an ordinary trade that reaches a merchant for review outlives its reservation's deadline as a matter of course.

A settlement SHALL therefore record its hold in the reservation. Two reservation states exist for this. They are defined here rather than in OFS-2200 because they describe what *settlement* has done to a reservation it has taken authority over; OFS-2200 §18's own machine still terminates at `Escrow Locked`.

| Reservation state | Entered when | Cancellable (OFS-2200 §14) | Expiry sweep (OFS-2200 §12) | Reserved liquidity |
|---|---|---|---|---|
| Escrow Locked | The reservation validated (OFS-2200 §18) | Yes | Expires it, returning the liquidity | Held, returnable |
| Settling | A settlement was initiated against it | **No** — refused | **Skipped** | Committed to the settlement |
| Settled | That settlement concluded with the escrow moving | No — terminal | Skipped | Spent; never returned |

```text id="reservation-under-settlement"
Escrow Locked

↓  settlement initiated

Settling ──────── settlement approved, or arbitration released the escrow ────────→ Settled

    │
    └──── settlement cancelled, rejected, or arbitration returned the escrow ───────→ Escrow Locked
```

A settlement that ends **without** a transfer returns the reservation to `Escrow Locked` rather than terminating it. The reservation is genuinely live again for whatever remains of its own validation window: its owner may cancel it, a further settlement against it is legitimate, and if nobody does anything the ordinary sweep expires it and returns the liquidity on its next pass. This keeps the liquidity returned by exactly the two paths that already return it, exactly once.

A settlement that ends **with** a transfer terminates the reservation at `Settled`, and the liquidity is not returned. The asset was sold. Returning it would credit the merchant's advertisement with inventory they no longer hold — which is what happened to every completed trade for as long as a completed reservation sat in `Escrow Locked` waiting to go stale.

### 5a.1 Where the refusal lives

The refusal SHALL be part of the deterministic function every node applies to a replicated `ReservationCancelled` event, not a check performed by the node that received the request. A cancellation reaches a peer as gossip (OFS-1200) without passing through any node-local API guard, so a guard placed at the API boundary would refuse the originator and admit every replica — nodes would disagree about one reservation, which is worse than the condition it was meant to fix.

### 5a.2 Where the two records may still disagree

Marking the reservation is best-effort and MUST NOT gate acceptance of the settlement. A node that does not hold the reservation, or holds it in some other state, SHALL still accept the settlement.

This is not a loose end but the only safe rule available. OFS-2200 §12 computes expiry against each node's own clock, so around the deadline a node that has swept holds an `Expired` reservation for a reservation its neighbours still hold as `Escrow Locked`. A settlement initiated in that window would be refused by the node that swept and accepted everywhere else, stranding a live trade on that node permanently — a strictly worse failure than the reservation record being briefly out of step.

Because the two can disagree, a client presenting one aggregate status for a trade SHALL let the settlement decide whenever a settlement exists, and consult the reservation only where none does. That ordering is what makes two honest nodes answer the same question the same way; the reverse ordering reports `Cancelled` for a trade whose escrow is about to release.

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

## 20a. Disputed

`Disputed` is a state of *this* machine, not a label the Dispute Protocol keeps to itself. Opening a dispute (OFS-2400 §5) freezes the escrow (OFS-2400 §6), and a settlement whose escrow is frozen is not awaiting payment, not under merchant review, and not concluded. A settlement left in its previous state while arbitrators examine it is indistinguishable from one whose merchant simply has not replied yet, which is what every client reading it would show.

It SHALL have both an entry and an exit. An entry alone would be worse than neither: dispute resolution terminates on the dispute record, escrow release requires `Approved`, and a settlement parked in `Disputed` could therefore never record the release of an escrow arbitration awarded to the buyer. Every arbitrated trade would strand there permanently.

**Entry.** Opening a dispute moves the settlement to `Disputed` and records that it was escalated. It is legal from every state except:

* `Cancelled` — no escrow was ever at stake, so there is nothing to freeze.
* `Disputed` — already frozen; OFS-2400 §5 permits one dispute per settlement, and this is what enforces it.

`Approved` and `Completed` are deliberately not distinguished. `Completed` is each node's own observation of an on-chain confirmation (OFS-4300 §7-8), so two honest nodes hold the two states for the same settlement at the same instant; a rule that admitted one and refused the other would accept a dispute on one node and refuse it on its neighbour.

**Exit.** The settlement leaves `Disputed` when — and only when — the chain has executed the case and the node has independently observed that confirmation (OFS-4200). The verdict decides where it lands, because what the settlement layer needs to know is not who won but what happened to the escrow:

| Dispute outcome (OFS-2400 §17) | What the program did to the escrow | Settlement state |
|---|---|---|
| Buyer Wins | Released to the buyer, identically to an uncontested approval | `Completed` |
| Mutual Settlement | Split; part reaches the buyer | `Completed` |
| Merchant Wins | Returned to the merchant's liquidity vault | `Cancelled` |
| Invalid | Returned to the merchant's liquidity vault | `Cancelled` |

A node that observed an execution but could not read its outcome SHALL leave the settlement in `Disputed`. That is the truth — something happened on chain and this node does not yet know what — and it matches what the same node records about the dispute itself.

The settlement does **not** return to the state it was escalated from. A restored `Payment Submitted` would be a live settlement awaiting a merchant decision that has already been made for them and will never come.

The reservation follows the same fork: an escrow released ends it at `Settled` (§5a), an escrow returned puts it back to `Escrow Locked`, and while the case is open it stays `Settling` — a frozen escrow is the strongest possible statement that the liquidity behind it is committed.

**After the case.** A settlement that has been arbitrated retains a record of the escalation once it resolves, because its state no longer says so. "Was this trade arbitrated?" is a question about a trade's history that reputation (OFS-3000) and counterparty history both ask, and it is answerable from the settlement alone — which is what keeps a reader of one party's own settlements from having to consult, and be able to see, dispute records at large.

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

### 24.1 A settlement read does not name the parties

A node answering a settlement read to a caller who has not proven they are party to it MUST NOT disclose the buyer, the seller, their keys, or the payment reference. The state, the amount, the timings and the on-chain release signature remain public — they describe the trade rather than the people, and the release signature names a transaction anyone can already read on chain, which is what makes a settlement independently checkable.

The payment reference is the sharpest of these. It is free text a buyer fills in with their own bank or mobile-money reference, so it routinely carries a real name or an account number, and nothing outside the trade has any business reading it.

OFS-8200 §7.1 states the rule across every trade read and gives the reasoning, which is about a graph rather than about any single field: a settlement names both parties, so an unauthenticated enumerating read reconstructs who trades with whom across the whole network. A party reads their own settlements in full through the wallet-proof reads of OFS-8200 §7.2.

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
