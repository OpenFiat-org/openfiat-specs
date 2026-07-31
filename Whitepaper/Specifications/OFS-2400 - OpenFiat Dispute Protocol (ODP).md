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
* Current Status (§8.1)
* Assigned Arbitrators
* Resolution (set only as §16.2 allows)
* On-Chain Execution Signature
* Evidence References

### 8.1 Dispute statuses

`[PROPOSED — NEEDS SIGN-OFF]`

This specification named a Current Status without ever saying what the
statuses are. §16.2 then described `Awaiting Chain Execution` in prose, as
an addition to a list that did not exist. The enumeration:

| Status | Meaning |
|---|---|
| `Open` | Escrow frozen (§6). Evidence accepted; arbitrators may still join. |
| `CaseLocked` | The required arbitrator count is reached (§14). No further arbitrators may join; the commit phase is live. |
| `RevealPhase` | Every required arbitrator has committed. The reveal phase is live. |
| `AwaitingChainExecution` | The off-chain layer has done everything it can, and the escrow has not moved. |
| `Resolved` | The chain executed an outcome and this node has independently observed the confirmation. |

Two of these carry rules that are not obvious from the name.

**`AwaitingChainExecution` is reached two ways** — every required
arbitrator has revealed, or both parties signed a mutual settlement (§17).
It is named for the *execution* rather than for a verdict because those two
differ: the chain decides the first and merely carries out the second. What
a node knows in both cases is the same, and it is the only thing worth
recording — nothing has been paid yet.

**`Resolved` has exactly one entry path**: an observed, confirmed on-chain
execution (§16.2). No count of reveals, no pair of party signatures, and no
gossiped message may set it. A record that reaches `Resolved` by any other
route is asserting an outcome the protocol did not establish.

There is no `Rejected` or `Cancelled` status. A dispute that should not
have been opened is resolved as `Invalid Dispute` (§17) by the chain like
any other outcome, rather than disappearing from the record — the Dispute
ID is permanent (§7) and so is what happened to it.

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

Awaiting Chain Execution

↓

Chain Arbitrates & Executes Outcome

↓

Escrow Released

↓

Execution Observed Confirmed

↓

Reputation Updated

↓

Dispute Closed
```

The verdict is the chain's, not the collecting node's — see §16.2. `Awaiting Chain Execution` is the state the off-chain record holds from the moment it has collected everything it can until the moment it observes what the chain decided.

The state is named for the *execution* rather than for a verdict because it is reached two ways and only one of them is a verdict. The path drawn above arrives from a completed reveal phase, where an arbitrator ruling is pending. A case where both parties have agreed a mutual settlement (§17) arrives at the same state from a different direction, with no ruling pending at all — but with the same thing outstanding, which is the escrow actually moving.

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

### 16.2 The chain decides; the off-chain layer collects and records

`[PROPOSED — NEEDS SIGN-OFF]`

| Rule | Value |
|---|---|
| Who tallies revealed votes | The on-chain program, alone |
| What the off-chain dispute registry does | Collects, verifies and replicates signed commitments and reveals |
| State once every required reveal is in | `Awaiting Chain Execution` |
| What may set a resolution | An observed, confirmed on-chain execution. Nothing else, with no exceptions |

The off-chain registry MUST NOT tally revealed votes, and MUST NOT derive a verdict from them.

**Two tallies of the same votes are a divergence generator, not a second opinion.** The chain re-arbitrates the same case under its own rules — stake-weighted, with the counted-vote floor of §16.1, re-opening a round on a tie rather than breaking it. An off-chain tally counting heads over the same reveals can therefore reach a different answer about the same dispute, and when it does, the interface shows one outcome while the money follows the other. The chain is the authority over the escrow, so the off-chain answer is not a second opinion; it is a statement the protocol makes and then contradicts with its own funds.

**Collecting is still the off-chain layer's job**, and it is a real one: verifying each arbitrator's signature, checking a reveal against that arbitrator's earlier commitment, refusing a reveal from a wallet that never committed, discarding duplicates, and replicating the result so every node sees the same evidence. All of that is the layer doing everything it can. Deciding is the part it cannot do.

**Observing, not assuming.** A resolution is recorded only when this node has independently observed the executing transaction confirm and has read the outcome from the case account on chain. A node that saw an execution land but could not read what it decided MUST remain in `Awaiting Chain Execution` and record the transaction it observed. That state is the truth — something happened on chain and this node does not yet know what — and inventing a verdict to fill the gap is the exact failure this rule removes.

**The chain executes on its own deadlines.** A commit or reveal window can expire with seats unfilled, and the chain will decide anyway. So an implementation MUST accept an observed execution against a case its own view still considers live; refusing it would leave the node displaying a running case that has already paid out.

**A case is `Awaiting Chain Execution`, never "Resolved pending execution".** The distinction is the whole point: the first says the off-chain layer has finished its work, the second would claim an outcome it is not entitled to name.

**This rule has no exception for party agreement.** A mutual settlement is agreed off-chain and MUST be recorded as agreed — but agreeing is not paying, and until the escrow moves the case is `Awaiting Chain Execution` like any other. Two signatures do not release funds; a record reading `Resolved` while the money is still locked overstates by exactly the margin this section exists to close. Worse, the chain remains free to execute an arbitrated outcome on a case whose parties agreed but never relayed that agreement, which would put the two layers back into contradiction about a single dispute. See §17.

---

## 17. Resolution Outcomes

A dispute concludes with exactly one resolution.

Every outcome below except Mutual Settlement is an **arbitrator ruling**, and is therefore the chain's to declare (§16.2). Mutual Settlement is the exception in one respect only: it is not a ruling at all, being reached by both parties signing their agreement rather than by any arbitrator voting. An implementation MUST NOT report a mutual settlement as an arbitrator ruling, or fold it into Invalid Dispute — the two pay different parties.

**It is not an exception to §16.2.** The off-chain layer verifies both signatures directly and MUST record the agreement as soon as it has them — that is a real fact about the case and withholding it would hide the parties' own decision from them. But recording an agreement is not recording a resolution. Until the escrow has actually moved and that execution has been observed confirmed, the case is `Awaiting Chain Execution` and its resolution is unset, exactly as for a case awaiting a ruling.

The distinction is easy to lose, because unlike a ruling there is no computation here that two nodes could perform differently — the agreement simply *is* the two signatures. The reason it still must wait is that signatures do not move money. A case marked `Mutual Settlement` while the funds sit locked tells both parties the dispute is over and paid when neither is true. And because the chain arbitrates on its own deadlines (§16.2), it remains free to execute an arbitrated outcome on a case whose parties agreed privately and never relayed it — putting the two layers back into contradiction about a single dispute, which is precisely what §16.2 exists to prevent.

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

Dispute events gossiped between nodes:

* DisputeOpened
* EvidenceSubmitted
* EvidenceRequested
* ArbitratorJoined
* CaseLocked
* VoteCommitted
* VoteRevealed
* MutualSettlementAgreed
* AppealSubmitted (future)
* DisputeClosed

Events propagate through OFS-1200.

### 21.1 What a node learns from the chain instead

`ResolutionIssued` and `EscrowReleased` are **not gossip events**, and were
listed above as though they were. They name things the chain does, and a
node learns both by observing the chain rather than by accepting a peer's
message (§16.2).

The distinction is not editorial. A gossiped event is something a peer
tells you; a chain observation is something you checked. **No gossiped
event carries a verdict**, and an implementation MUST NOT adopt a
resolution because a peer sent one — a signed message claiming an outcome
is a claim, and escrow is not moved by claims. This is the rule the
Settlement Protocol already applies to escrow release: every node verifies
chain confirmation for itself.

Listing them as gossip events invited exactly the implementation this
protocol forbids: a node that accepts `ResolutionIssued` from the network
and marks a case resolved. There is no such message, and there must not be
one.

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

**A public dispute read is redacted.** A node answering a dispute read to a caller who has not proven they are party to it MUST NOT disclose the parties, the opening party's free-text reason, or which arbitrator cast which vote. Counts survive so a case can be seen to be progressing; the pairings do not. OFS-8200 §7.1 states the rule in full and the reasoning behind it, and applies it identically to reservations, settlements and trades. A dispute is the record where knowing who fell out with whom is most obviously worth misusing, and its `reason` field is free text describing a real disagreement about real money — it names people, banks and account references as a matter of course.

The mutual-settlement flags are redacted with them. "One side has agreed and the other has not" is a negotiating position, and publishing it to onlookers changes a negotiation between two people.

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
* Collect and replicate reveals without tallying them, and record a resolution only from an observed on-chain execution (§16.2).
* Redact parties, the opening reason, and the arbitrator-to-vote pairing from any dispute read the caller has not proven their standing for (§23).
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
