# Chapter 11 — The OpenFiat Dispute Resolution Protocol

## 11.1 Introduction

No marketplace can completely eliminate disputes.

Banks experience delayed transfers.

Mobile money systems occasionally fail.

Users make mistakes.

Payment references may be entered incorrectly.

Communication misunderstandings occur.

Rather than attempting to prevent every dispute, OpenFiat provides a transparent, decentralized, and deterministic process for resolving them.

Unlike traditional peer-to-peer exchanges, where platform employees review cases behind closed doors, OpenFiat distributes dispute resolution across independent arbitrators who are economically incentivized to reach honest decisions.

No single company decides the outcome.

No administrator can override the protocol.

Every decision follows publicly documented rules and is ultimately enforced by immutable smart contracts.

The objective of the OpenFiat Dispute Resolution Protocol is not merely to settle disagreements—it is to create a dispute system that is fair, resistant to manipulation, globally accessible, and economically aligned with honest participation.

---

## 11.2 Design Objectives

The dispute system was designed around several fundamental principles.

### Decentralized

No single organization should control dispute outcomes.

### Economically Secure

Dishonest arbitration should carry financial consequences.

### Transparent

Every rule governing dispute resolution should be publicly documented.

### Privacy Conscious

Only qualified arbitrators participating in a case should gain access to dispute evidence.

### Deterministic

The Escrow Program should execute every decision automatically.

### Scalable

The protocol should support thousands of simultaneous disputes without centralized coordination.

---

## 11.3 When a Dispute May Be Opened

Either participant may initiate a dispute when a trade cannot be completed normally.

Common examples include:

* Buyer claims payment was sent.
* Merchant claims payment was never received.
* Incorrect payment amount.
* Incorrect payment recipient.
* Fraudulent payment evidence.
* Payment reversal or chargeback.
* Other protocol-defined settlement disagreements.

Once a dispute has been opened, the associated funds remain locked within the vault until a final decision has been reached.

Neither participant may unilaterally release or reclaim the assets.

---

## 11.4 Filing a Dispute

Opening a dispute is intentionally not free.

The participant initiating the dispute must pay a **filing fee** denominated in OPEN.

The filing fee serves several purposes:

* Discourages frivolous disputes.
* Reduces automated abuse.
* Compensates arbitrators.
* Helps fund dispute infrastructure.

If the dispute is determined to have been filed in good faith, the filing fee may be refunded according to protocol rules.

If the dispute is determined to be frivolous or abusive, some or all of the filing fee may be forfeited.

The exact allocation of forfeited fees is defined within the token economics chapter.

---

## 11.5 Case Creation

When a dispute is submitted, the protocol creates a new dispute case.

The case contains:

* Trade identifier.
* Vault reference.
* Timeline of protocol events.
* Signed trade messages.
* Attached evidence.
* Payment confirmations.
* Relevant metadata.

The dispute case is published to the OpenFiat network.

However, the evidence itself is **not** immediately distributed to the public.

Only limited case metadata is announced.

This preserves participant privacy while enabling decentralized arbitration.

---

## 11.6 Arbitrator Eligibility

Not every participant may immediately serve as an arbitrator.

To become eligible, a participant must satisfy protocol requirements.

Examples include:

* Minimum OPEN stake.
* Minimum arbitrator reputation.
* Sufficient protocol age.
* No active arbitration penalties.
* Current protocol compatibility.

Governance may adjust eligibility requirements over time.

---

## 11.7 Voluntary Participation

Unlike systems that randomly assign jurors, OpenFiat allows qualified arbitrators to voluntarily join cases.

The process is as follows:

```text id="arbgd1"
Case Published

↓

Qualified Arbitrators Discover Case

↓

Arbitrator Commits Stake

↓

Protocol Accepts Participation

↓

Evidence Becomes Available
```

This design has several advantages.

Arbitrators choose cases they wish to review.

No participant must remain continuously online waiting for random assignment.

Because evidence is unavailable before joining, opportunities for targeted bribery are significantly reduced.

---

## 11.8 Privacy Before Participation

One of the most important security properties of the protocol is information isolation.

Before joining a case, arbitrators receive only minimal metadata.

Examples include:

* Case identifier.
* Protocol version.
* General dispute category.
* Required arbitration stake.
* Filing timestamp.

They **do not** receive:

* Wallet balances.
* Payment evidence.
* Screenshots.
* Bank references.
* Trade messages.
* Participant identities beyond protocol identifiers.

Only after successfully joining the case does the protocol provide access to the dispute evidence.

This significantly reduces opportunities for external influence before participation.

---

## 11.9 Dynamic Participation

The number of arbitrators required for a dispute is intentionally **not publicly disclosed**.

Publishing this number would allow participants to estimate the value of the assets involved.

Instead, the protocol determines the required threshold internally based on several factors, including:

* Economic value at risk.
* Current network conditions.
* Arbitrator availability.
* Governance-defined security parameters.

Once the required threshold has been reached, the case is automatically locked.

No additional arbitrators may join.

---

## 11.10 Case Locking

Case locking serves several important purposes.

It prevents:

* Late participation after evidence becomes widely known.
* Unlimited arbitrator growth.
* Last-minute manipulation attempts.
* Reward dilution.

Once locked, the case transitions to evidence review.

---

## 11.11 Evidence

Participants may submit evidence supporting their claims.

Examples include:

* Payment receipts.
* Bank confirmations.
* Mobile money confirmations.
* Screenshots.
* Reference numbers.
* Trade communication.
* Other protocol-supported attachments.

Evidence is cryptographically linked to the dispute case.

Every submission is timestamped.

Evidence cannot be silently modified after submission.

Future protocol versions may support additional evidence formats without altering the arbitration process.

---

## 11.12 Commit–Reveal Voting

To prevent arbitrators from influencing one another, OpenFiat uses a commit–reveal voting process.

### Commit Phase

Each arbitrator submits a cryptographic commitment to their decision.

The commitment reveals nothing about the vote itself.

### Reveal Phase

After the commit period ends, arbitrators reveal both:

* Their vote.
* The secret used to generate the commitment.

The protocol verifies that the revealed vote matches the earlier commitment.

Only valid votes are counted.

Because every arbitrator commits before any vote becomes visible, strategic vote manipulation becomes substantially more difficult.

---

## 11.13 Consensus

After all valid reveal votes have been received—or after the reveal deadline expires—the protocol determines the final outcome.

Consensus follows deterministic protocol rules.

There is no discretionary interpretation by AllenHark or node operators.

Every compliant implementation observing the same votes reaches the same conclusion.

The resulting decision becomes final.

---

## 11.14 Settlement

Once consensus has been reached, the Escrow Program executes the outcome automatically.

Depending on the decision, this may include:

* Releasing stablecoins to the buyer.
* Returning stablecoins to the seller.
* Refunding filing fees.
* Distributing arbitration rewards.
* Applying protocol penalties.

No human approval is required after consensus.

---

## 11.15 Arbitrator Rewards

Honest arbitration should be economically attractive.

Arbitrators who participate correctly receive rewards funded by:

* Protocol dispute fees.
* Filing fees.
* Future governance-defined incentive pools.

Rewards are distributed automatically after settlement.

No manual accounting is required.

---

## 11.16 Minority Penalties

OpenFiat introduces measured economic accountability.

Arbitrators whose revealed votes fall outside the final consensus may experience a partial reduction of their arbitration stake.

This mechanism serves several purposes.

It encourages careful evidence review.

It discourages random voting.

It increases the cost of coordinated manipulation.

Importantly, penalties are intentionally moderate.

The protocol recognizes that honest arbitrators may occasionally reach different conclusions.

The objective is to discourage negligence and collusion rather than punish good-faith disagreement.

---

## 11.17 Reputation Updates

After every completed case, arbitrator reputation is updated.

Metrics include:

* Participation reliability.
* Voting completion.
* Consensus alignment.
* Historical consistency.
* Case throughput.

Likewise, buyer and merchant reputation may also be updated to reflect dispute outcomes where appropriate.

---

## 11.18 Appeals

The initial version of OpenFiat deliberately does **not** include an appeal process.

Introducing multiple arbitration rounds would increase complexity, extend settlement times, and create uncertainty regarding finality.

Instead, every dispute follows a single deterministic arbitration process.

Governance may introduce optional appeal mechanisms in future protocol versions if the community determines that the benefits outweigh the additional complexity.

---

## 11.19 Why This Protocol Is Different

Most peer-to-peer exchanges rely on centralized support teams.

Users submit evidence.

Employees privately review the case.

The decision is largely opaque.

OpenFiat replaces institutional judgment with an open protocol.

Independent arbitrators voluntarily participate.

Economic incentives encourage careful review.

Commit–reveal voting prevents strategic coordination.

Smart contracts enforce the final decision.

Every rule governing the process is public.

This transforms dispute resolution from a customer support function into decentralized public infrastructure.

---

## 11.20 Future Evolution

The dispute framework has been designed for long-term growth.

Potential future enhancements include:

* Specialized arbitrator categories.
* AI-assisted evidence organization.
* Multi-language evidence translation.
* Expert witness mechanisms.
* Cross-protocol arbitration interoperability.

These capabilities can be introduced without changing the fundamental dispute lifecycle.

---

## 11.21 Looking Ahead

Trustworthy dispute resolution strengthens confidence in individual trades.

However, long-term stewardship of the protocol requires a different mechanism.

The next chapter introduces the OpenFiat Governance Protocol, explaining how OPEN holders propose upgrades, vote on protocol changes, manage treasury resources, and progressively decentralize the evolution of OpenFiat over time.
