# OFS-2400 — OpenFiat Dispute Protocol (ODP)

**Document ID:** OFS-2400

**Title:** OpenFiat Dispute Protocol

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Trading

**Depends On:** OFS-2000, OFS-2200, OFS-2300, OFS-3000, OFS-5000

---

## Abstract

The OpenFiat Dispute Protocol (ODP) defines how disagreements arising from OpenFiat trades are initiated, managed, investigated, resolved, and permanently recorded.

Unlike traditional peer-to-peer marketplaces that rely on a centralized customer support team, OpenFiat uses decentralized dispute resolution supported by cryptographically verifiable evidence, merchant reputation, community arbitrators who stake OPEN and vote using a commit-and-reveal scheme, and deterministic protocol rules.

The protocol guarantees that disputed escrow remains secure until a final resolution has been reached.

---

## 1. Introduction

Although the vast majority of trades are expected to complete successfully, disputes are inevitable.

Examples include:

* Buyer claims payment was sent.
* Merchant claims payment was never received.
* Incorrect payment amount.
* Payment sent from an unexpected account.
* Fraudulent payment receipt.
* Chargeback concerns.
* Banking delays.
* Accidental duplicate payments.

The Dispute Protocol provides a standardized framework for resolving these situations fairly and transparently.

---

## 2. Scope

This specification defines:

* Dispute creation
* Evidence collection
* Evidence synchronization
* Escrow freezing
* Arbitration
* Resolution
* Reputation impact
* Appeals
* Dispute history

This specification does **not** define:

* Reservation
* Settlement
* Merchant reputation formulas
* Governance voting

---

## 3. Design Goals

The protocol SHALL:

* Protect honest participants.
* Preserve escrow integrity.
* Maintain deterministic dispute state.
* Support evidence from multiple sources.
* Minimize fraud.
* Produce auditable outcomes.
* Remain decentralized.

---

## 4. Design Philosophy

A dispute is **not** evidence of fraud.

It is simply a disagreement requiring additional verification.

The protocol assumes neither party is correct until sufficient evidence has been evaluated.

---

## 5. When a Dispute May Be Opened

A dispute MAY be initiated when:

* Merchant rejects payment.
* Buyer disagrees with rejection.
* Merchant reports incorrect payment.
* Buyer reports escrow issue.
* Payment delay exceeds timeout.
* Fraud is suspected.
* Both parties cannot agree.

Disputes may only be opened while a trade remains active.

---

## 6. Automatic Escrow Freeze

Immediately after a dispute begins:

```text id="dispute-freeze"
Settlement

↓

Dispute Opened

↓

Escrow Frozen

↓

No Funds Move
```

The OpenFiat Program freezes the escrow automatically.

Neither party can release or reclaim funds while the dispute is active.

---

## 7. Dispute Identifier

Every dispute receives a globally unique Dispute ID.

The identifier remains permanent for auditing purposes.

---

## 8. Dispute Record

Each dispute contains:

* Dispute ID
* Trade ID
* Reservation ID
* Advertisement ID
* Buyer Wallet
* Merchant Wallet
* Creation Time
* Current Status
* Assigned Arbitrators
* Resolution
* Evidence References

---

## 9. Evidence Submission

Either participant MAY submit evidence.

Examples include:

* Bank receipt
* Mobile money receipt
* Transaction reference
* Account statement
* PDF
* Screenshot
* Chat transcript
* Signed protocol events

Evidence remains immutable after submission.

Additional evidence creates new evidence entries.

---

## 10. Protocol Evidence

The protocol itself automatically contributes evidence.

Examples include:

* Reservation events
* Settlement events
* Session history
* Advertisement history
* Escrow state
* Oracle price history
* Notification history
* Wallet signatures

Protocol-generated evidence is generally considered highly trustworthy because it is cryptographically verifiable.

---

## 11. External Evidence

Future protocol versions MAY integrate external providers.

Examples:

* Banking APIs
* Mobile money verification
* Payment verification providers
* Fraud intelligence services

External evidence supplements—but never replaces—protocol evidence.

---

## 12. Wallet Risk Intelligence

Risk Intelligence Providers registered through OFS-1500 MAY contribute wallet intelligence.

Examples include:

* Known scam wallets
* Sanctioned addresses
* Fraud rings
* Money laundering indicators
* Previous dispute history

Examples of providers include commercial blockchain intelligence services such as CipherOwl and future compatible providers.

Risk intelligence informs investigations but does not automatically determine the outcome of a dispute.

---

## 13. Evidence Synchronization

Evidence submissions generate immutable protocol events.

Every node eventually receives identical evidence references.

Large files MAY be stored externally with cryptographically verifiable hashes recorded within the protocol.

---

## 14. Dispute Timeline

```text id="dispute-flow"
Trade

↓

Dispute Opened

↓

Escrow Frozen

↓

Evidence Submitted

↓

Investigation

↓

Arbitrators Commit Stake & Join Case

↓

Case Locked

↓

Evidence Released To Arbitrators

↓

Commit Phase

↓

Reveal Phase

↓

Decision (Consensus)

↓

Escrow Released

↓

Reputation Updated

↓

Dispute Closed
```

---

## 15. Investigation Phase

During investigation:

* Additional evidence may be requested.
* Participants may respond.
* Risk providers may contribute intelligence.
* Timeline events remain synchronized.

Every action is permanently recorded.

---

## 16. Arbitration

OpenFiat uses decentralized arbitration from the initial network launch. Disputes are resolved by independent, staked arbitrators who voluntarily join a case and vote using a commit-and-reveal scheme, as defined in Chapter 11 (The OpenFiat Dispute Resolution Protocol). This is the v1 arbitration mechanism, not a placeholder pending a future decentralization step.

In summary:

* Qualified arbitrators (minimum OPEN stake, minimum reputation, sufficient protocol age, no active penalties — Chapter 11 §11.6) may voluntarily join a published case.
* Case evidence remains hidden until an arbitrator has committed stake and been accepted into the case, reducing pre-participation bribery risk (Chapter 11 §11.7-11.8).
* The arbitrator threshold required for a case is determined internally and not publicly disclosed (Chapter 11 §11.9); once reached, the case locks and no further arbitrators may join (Chapter 11 §11.10).
* Arbitrators vote via commit-and-reveal: each arbitrator submits a cryptographic commitment during the commit phase, then reveals the vote and secret during the reveal phase (Chapter 11 §11.12). Only revealed votes matching their earlier commitment are counted.
* A round reaches a verdict only if at least three votes were counted (§16.1). Fewer is not a decision, whatever stake stands behind them.
* Consensus follows deterministic protocol rules, with no discretionary interpretation by AllenHark or any node operator (Chapter 11 §11.13).
* Arbitrators whose revealed vote falls outside consensus may incur a partial, moderate stake slash; arbitrators voting with consensus earn arbitration rewards (Chapter 11 §11.15-11.16).

Future protocol versions MAY extend this model — specialized arbitrator categories, expert witnesses, cross-protocol interoperability (Chapter 11 §11.20) — but the underlying commit-reveal, stake-secured, voluntary-participation mechanism is not itself a future upgrade; it is the current protocol.

### 16.1 Minimum Counted Votes

A dispute round MUST NOT produce a verdict unless at least **three** votes were counted.

This minimum is new in this amendment. Earlier versions of this specification stated only that qualified arbitrators "may voluntarily join a published case" and set no minimum anywhere, so consensus followed whatever happened to be revealed: one arbitrator who was the only one to reveal decided the case alone and moved the escrow, and nothing in the protocol required a second participant to exist. Three is the smallest set that produces a genuine majority and still resolves cases when one member is absent or dishonest.

**Counted, not seated.** A counted vote is one the tally actually sums: revealed within the reveal window, matching that arbitrator's earlier commitment, and carrying non-zero stake weight. The minimum is deliberately *not* a count of seats filled. A seat can be occupied while contributing nothing to the totals — an arbitrator who commits and never reveals, or who reveals with no effective stake behind them — so counting seats would let three such accounts satisfy the minimum while the decision still rested on a single real vote. That is the seat-squatting shape the stake gate on joining a case already exists to prevent, and it must not be reintroduced through the participation check.

**A floor on participants, not on weight.** No amount of stake behind fewer than three counted votes decides a case. A single large stake deciding alone is precisely the case this rule exists to exclude, so the threshold is independent of the weights, which continue to determine only *which* outcome wins once the floor is met.

**Falling short is not a verdict.** A round with fewer than three counted votes MUST NOT pay either party, and MUST NOT be resolved to any of §17's outcomes. It is handled exactly as a tie is: the case re-opens for a further arbitration round with fresh commit and reveal windows (§14), bounded by the protocol's round limit, giving arbitrators who missed the round another opportunity to participate. If the round limit is reached with still no verdict, the escrow is split evenly between the parties. Deciding nothing must never be worth more to either party than losing, which is what the even split guarantees: a party who suppresses participation forfeits half rather than winning the escrow.

**Companion to per-case sortition.** OFS-4100 §4.1 makes arbitrator eligibility a per-case draw the arbitrator cannot choose the outcome of. Drawing seats from a small pool is what makes this floor necessary rather than merely prudent: on a thin pool a draw can leave one or two eligible wallets, and without a minimum the case would be handed to whoever holds the most wallets. The two mechanisms are load-bearing together — sortition decides who may vote, this floor decides how few voters is too few for the result to count.

---

## 17. Resolution Outcomes

A dispute concludes with exactly one resolution.

Possible outcomes include:

Buyer Wins

* Escrow released to buyer.

Merchant Wins

* Escrow returned to merchant or seller.

Mutual Settlement

* Participants voluntarily resolve the dispute.

Partial Settlement (Future)

* Escrow divided according to arbitration decision.

Invalid Dispute

* Dispute rejected.

No Consensus

* Reached only when the round limit is exhausted without a verdict — including because no round ever reached the minimum counted votes of §16.1. The escrow is split evenly between the parties. This is a termination, not a finding against either of them.

---

## 18. Appeals

Future protocol versions MAY introduce formal appeals.

Appeals SHOULD require:

* New evidence
* Governance-defined conditions
* Additional arbitration

Repeated appeals without new evidence SHOULD be discouraged.

---

## 19. Reputation Impact

Every completed dispute affects reputation.

Possible impacts include:

Positive:

* Honest cooperation
* Rapid evidence submission
* Accurate reporting

Negative:

* Fraud attempts
* False evidence
* Repeated disputes
* Abuse of dispute system

Dispute statistics contribute to OFS-3000.

---

## 20. Abuse Prevention

The protocol discourages abusive behavior.

Examples include:

* Frivolous disputes
* Spam evidence
* Repeated false claims
* Deliberate delays
* Evidence manipulation

Repeated abuse may reduce marketplace reputation.

---

## 21. Dispute Synchronization

Dispute events include:

* DisputeOpened
* EvidenceSubmitted
* EvidenceRequested
* ArbitratorJoined
* CaseLocked
* VoteCommitted
* VoteRevealed
* ResolutionIssued
* EscrowReleased
* AppealSubmitted (future)
* DisputeClosed

Events propagate through OFS-1200.

---

## 22. Notifications

Participants SHOULD receive notifications for:

Buyer

* Dispute opened
* Evidence requested
* Resolution issued
* Appeal deadline

Merchant

* New dispute
* Evidence received
* Investigation updates
* Final decision

Notification delivery is defined in OFS-6000.

---

## 23. Privacy

Only authorized participants and designated arbitrators SHOULD access dispute evidence.

Sensitive financial information SHOULD remain encrypted whenever possible.

Public network synchronization SHOULD replicate hashes and metadata rather than exposing confidential documents.

---

## 24. Security Considerations

Implementations MUST protect against:

* Evidence tampering
* Forged receipts
* Replay attacks
* Unauthorized escrow release
* Duplicate disputes
* Fake arbitrators
* Identity spoofing
* Duplicate processing of the same signed dispute event

Every dispute action MUST be digitally signed.

## Idempotency

Every dispute event (evidence submission, arbitrator commitment, vote commit, vote reveal, resolution) carries a unique nonce tied to its dispute case. Implementations MUST detect and silently discard a duplicate event with an already-processed nonce rather than re-applying it — for example, a resubmitted `VoteRevealed` message for a vote already recorded MUST NOT be counted twice, and a replayed `EscrowReleased` outcome MUST NOT trigger a second transfer.

---

## 25. Performance Considerations

Disputes are relatively rare but operationally significant.

Implementations SHOULD optimize:

* Efficient evidence synchronization
* Incremental evidence retrieval
* Fast state recovery
* Secure document storage
* Minimal bandwidth usage

---

## 26. Conformance

A compliant implementation MUST:

* Support dispute creation.
* Automatically freeze escrow.
* Support evidence submission.
* Preserve immutable evidence history.
* Support deterministic dispute states.
* Refuse to decide a round on fewer than the minimum counted votes (§16.1), and re-open rather than pay out when it falls short.
* Generate signed dispute events.
* Synchronize dispute records.
* Update reputation after resolution.
* Prevent escrow movement during active disputes.

---

## 27. Relationship to Other Specifications

The Dispute Protocol protects the integrity of the OpenFiat marketplace whenever settlement cannot be completed through normal procedures.

```text id="dispute-architecture"
              OFS-2200
      Reservation Protocol
                    │
                    ▼
              OFS-2300
       Settlement Protocol
                    │
      Successful      │      Disagreement
      Settlement      ▼
                 OFS-2400
            Dispute Protocol
                    │
        ┌───────────┼─────────────┐
        ▼           ▼             ▼
  OFS-3000    OFS-5000      OFS-4000
 Reputation   Identity      Governance
```

The OpenFiat Dispute Protocol answers one essential question:

**"When two parties disagree, how can a decentralized marketplace resolve the conflict fairly without relying on a centralized customer support team?"**

By combining immutable protocol history, cryptographically verifiable evidence, automatic escrow freezing, independent risk intelligence, and an extensible arbitration framework, the OpenFiat Dispute Protocol provides a transparent and interoperable foundation for dispute resolution while preserving user sovereignty and marketplace integrity.
