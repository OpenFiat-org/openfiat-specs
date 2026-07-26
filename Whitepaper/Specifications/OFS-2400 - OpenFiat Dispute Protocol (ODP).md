# OFS-2400 — OpenFiat Dispute Protocol (ODP)

**Document ID:** OFS-2400

**Title:** OpenFiat Dispute Protocol

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Trading

**Depends On:** OFS-2000, OFS-2200, OFS-2300, OFS-3000, OFS-5000

---

# Abstract

The OpenFiat Dispute Protocol (ODP) defines how disagreements arising from OpenFiat trades are initiated, managed, investigated, resolved, and permanently recorded.

Unlike traditional peer-to-peer marketplaces that rely on a centralized customer support team, OpenFiat uses decentralized dispute resolution supported by cryptographically verifiable evidence, merchant reputation, community arbitrators (future), and deterministic protocol rules.

The protocol guarantees that disputed escrow remains secure until a final resolution has been reached.

---

# 1. Introduction

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

# 2. Scope

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

# 3. Design Goals

The protocol SHALL:

* Protect honest participants.
* Preserve escrow integrity.
* Maintain deterministic dispute state.
* Support evidence from multiple sources.
* Minimize fraud.
* Produce auditable outcomes.
* Remain decentralized.

---

# 4. Design Philosophy

A dispute is **not** evidence of fraud.

It is simply a disagreement requiring additional verification.

The protocol assumes neither party is correct until sufficient evidence has been evaluated.

---

# 5. When a Dispute May Be Opened

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

# 6. Automatic Escrow Freeze

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

# 7. Dispute Identifier

Every dispute receives a globally unique Dispute ID.

The identifier remains permanent for auditing purposes.

---

# 8. Dispute Record

Each dispute contains:

* Dispute ID
* Trade ID
* Reservation ID
* Advertisement ID
* Buyer Wallet
* Merchant Wallet
* Creation Time
* Current Status
* Assigned Arbitrator (future)
* Resolution
* Evidence References

---

# 9. Evidence Submission

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

# 10. Protocol Evidence

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

# 11. External Evidence

Future protocol versions MAY integrate external providers.

Examples:

* Banking APIs
* Mobile money verification
* Payment verification providers
* Fraud intelligence services

External evidence supplements—but never replaces—protocol evidence.

---

# 12. Wallet Risk Intelligence

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

# 13. Evidence Synchronization

Evidence submissions generate immutable protocol events.

Every node eventually receives identical evidence references.

Large files MAY be stored externally with cryptographically verifiable hashes recorded within the protocol.

---

# 14. Dispute Timeline

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

Decision

↓

Escrow Released

↓

Reputation Updated

↓

Dispute Closed
```

---

# 15. Investigation Phase

During investigation:

* Additional evidence may be requested.
* Participants may respond.
* Risk providers may contribute intelligence.
* Timeline events remain synchronized.

Every action is permanently recorded.

---

# 16. Arbitration

The initial OpenFiat network may utilize trusted protocol arbitrators appointed through governance.

Future protocol versions are expected to evolve toward decentralized arbitration mechanisms.

Potential future models include:

* Stake-weighted arbitrators
* Reputation-based arbitrators
* Jury selection
* Randomized arbitration committees

The arbitration mechanism itself is intentionally upgradeable.

---

# 17. Resolution Outcomes

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

---

# 18. Appeals

Future protocol versions MAY introduce formal appeals.

Appeals SHOULD require:

* New evidence
* Governance-defined conditions
* Additional arbitration

Repeated appeals without new evidence SHOULD be discouraged.

---

# 19. Reputation Impact

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

# 20. Abuse Prevention

The protocol discourages abusive behavior.

Examples include:

* Frivolous disputes
* Spam evidence
* Repeated false claims
* Deliberate delays
* Evidence manipulation

Repeated abuse may reduce marketplace reputation.

---

# 21. Dispute Synchronization

Dispute events include:

* DisputeOpened
* EvidenceSubmitted
* EvidenceRequested
* ArbitrationStarted
* ResolutionIssued
* EscrowReleased
* AppealSubmitted (future)
* DisputeClosed

Events propagate through OFS-1200.

---

# 22. Notifications

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

# 23. Privacy

Only authorized participants and designated arbitrators SHOULD access dispute evidence.

Sensitive financial information SHOULD remain encrypted whenever possible.

Public network synchronization SHOULD replicate hashes and metadata rather than exposing confidential documents.

---

# 24. Security Considerations

Implementations MUST protect against:

* Evidence tampering
* Forged receipts
* Replay attacks
* Unauthorized escrow release
* Duplicate disputes
* Fake arbitrators
* Identity spoofing

Every dispute action MUST be digitally signed.

---

# 25. Performance Considerations

Disputes are relatively rare but operationally significant.

Implementations SHOULD optimize:

* Efficient evidence synchronization
* Incremental evidence retrieval
* Fast state recovery
* Secure document storage
* Minimal bandwidth usage

---

# 26. Conformance

A compliant implementation MUST:

* Support dispute creation.
* Automatically freeze escrow.
* Support evidence submission.
* Preserve immutable evidence history.
* Support deterministic dispute states.
* Generate signed dispute events.
* Synchronize dispute records.
* Update reputation after resolution.
* Prevent escrow movement during active disputes.

---

# 27. Relationship to Other Specifications

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
