# Chapter 15 — Staking & Economic Security

## 15.1 Introduction

The security of OpenFiat is built upon economic incentives rather than centralized trust.

Instead of relying on administrators to determine who may participate in critical protocol roles, OpenFiat requires participants to demonstrate economic commitment by staking the protocol's native token, **OPEN**.

Staking serves several purposes simultaneously.

It demonstrates long-term commitment to the ecosystem.

It discourages malicious behavior.

It aligns participant incentives with the long-term success of the protocol.

It provides a financial mechanism through which participants become accountable for the responsibilities they assume.

Unlike traditional deposits or subscription fees, staking does not represent payment for protocol access.

Participants retain ownership of their staked OPEN.

However, participants who repeatedly violate protocol rules or fail to meet required standards may be subject to governance-defined penalties.

Staking therefore transforms economic commitment into protocol security.

---

## 15.2 Design Objectives

The staking system was designed around several guiding principles.

### Economic Accountability

Participants responsible for important protocol functions should have economic incentives to behave honestly.

### Non-Custodial

Participants always retain ownership of their stake unless protocol-defined penalties apply.

### Role-Based Security

Different protocol responsibilities require different staking requirements.

### Transparent

Every stake is visible and verifiable on-chain.

### Scalable

New protocol roles may introduce additional staking requirements without redesigning the staking system.

---

## 15.3 Why Staking?

OpenFiat operates without centralized administrators.

As a result, the protocol requires an objective method for determining which participants are trusted with additional responsibilities.

Staking fulfills this role.

By voluntarily locking OPEN, participants demonstrate confidence in the protocol and accept financial accountability for their actions.

The greater the responsibility assigned to a participant, the greater the economic commitment generally required.

---

## 15.4 Staking Lifecycle

Every staking relationship follows the same high-level lifecycle.

```text id="stakeflow1"
Acquire OPEN

↓

Stake OPEN

↓

Become Eligible

↓

Perform Protocol Role

↓

Earn Rewards

↓

Unstake Request

↓

Unlock Period

↓

Stake Released
```

The unlock period protects the protocol from participants who attempt to avoid accountability immediately after performing critical actions.

---

## 15.5 Merchant Staking

Merchants are the primary liquidity providers within OpenFiat.

Merchant staking serves multiple purposes.

It demonstrates commitment to the marketplace.

It unlocks advertisement capacity.

It determines eligibility for certain merchant features.

It provides economic accountability in cases of protocol abuse.

Merchant stake **does not** back individual trades.

Trade settlement is secured by the escrow vault system described earlier in this document.

Instead, merchant stake governs marketplace participation.

Examples of governance-controlled parameters include:

* Maximum number of active advertisements.
* Maximum aggregate advertisement capacity.
* Access to advanced merchant features.
* Eligibility for premium marketplace placement.

Merchant reputation and merchant stake work together.

High stake alone cannot compensate for poor marketplace behavior.

Likewise, excellent reputation alone does not eliminate staking requirements.

---

## 15.6 Arbitrator Staking

Arbitrators occupy one of the most sensitive roles within the protocol.

Accordingly, participation requires OPEN staking.

When joining a dispute:

* The arbitrator commits additional stake for that specific case.
* The case remains economically secured until resolution.
* Honest participation earns rewards.
* Negligent or malicious participation may result in partial slashing.

The objective is to align every arbitration decision with careful evidence review rather than financial speculation.

---

## 15.7 Node Operator Staking

Node operators maintain the decentralized OpenFiat network.

Responsibilities include:

* Peer communication.
* Advertisement propagation.
* Trade session synchronization.
* Reputation distribution.
* Marketplace state replication.
* Snapshot participation.

Node staking demonstrates long-term commitment to network reliability.

Only staked nodes become eligible for protocol rewards.

Nodes that consistently fail to satisfy minimum protocol standards may temporarily lose reward eligibility until performance improves.

---

## 15.8 Notification Provider Staking

Notification providers deliver optional communication services such as:

* Email.
* Telegram.
* Discord.
* Future messaging integrations.

Because users rely on these services during active trades, providers must maintain high reliability.

OPEN staking provides economic accountability while helping discourage low-quality or short-lived providers.

Repeated service failures may reduce reward eligibility or result in governance-defined penalties.

---

## 15.9 Oracle Provider Staking

Floating-price advertisements depend upon reliable exchange-rate data.

Oracle providers therefore stake OPEN before publishing pricing information.

Staking discourages dishonest reporting while reinforcing confidence in protocol pricing.

Oracle providers consistently publishing incorrect or unavailable pricing may lose reward eligibility and face additional penalties defined by governance.

---

## 15.10 Snapshot Provider Staking

Snapshot providers accelerate node synchronization by distributing verified RocksDB snapshots.

Their responsibilities include:

* Hosting current snapshots.
* Maintaining availability.
* Publishing integrity metadata.
* Supporting efficient network bootstrap.

Staking ensures that providers remain committed to delivering accurate and reliable infrastructure.

---

## 15.11 Future Staking Roles

The staking framework is intentionally extensible.

Future protocol extensions may introduce additional roles, including:

* Analytics providers.
* Indexing services.
* Mobile relay services.
* Institutional infrastructure.
* Future protocol extensions approved through governance.

Every new role inherits the same fundamental staking principles.

---

## 15.12 Reward Eligibility

Staking alone does not guarantee rewards.

Participants must also satisfy role-specific performance expectations.

Examples include:

Merchants:

* Maintain marketplace availability.
* Complete trades successfully.
* Respond within acceptable timeframes.

Nodes:

* Maintain uptime.
* Propagate protocol messages efficiently.
* Remain synchronized with the network.

Arbitrators:

* Join cases responsibly.
* Reveal votes before deadlines.
* Participate consistently.

Notification Providers:

* Deliver notifications reliably.
* Maintain acceptable latency.

Rewards therefore reflect both commitment and performance.

---

## 15.13 Slashing

Slashing is designed as a corrective mechanism rather than a punishment.

Its purpose is to discourage dishonest or negligent behavior that threatens the health of the protocol.

Potential causes include:

* Proven protocol abuse.
* Persistent infrastructure failures.
* Fraudulent oracle submissions.
* Failure to fulfill arbitration responsibilities.
* Repeated protocol violations.

Minor mistakes should not result in catastrophic financial penalties.

Instead, penalties should remain proportionate to the severity and frequency of misconduct.

---

## 15.14 Unlock Periods

Participants cannot immediately withdraw their stake after requesting to unstake.

An unlock period provides time for:

* Pending disputes to conclude.
* Outstanding protocol obligations to complete.
* Performance reviews where applicable.
* Slashing decisions already in progress to be finalized.

The exact duration is governed by protocol parameters and may differ between participant roles.

---

## 15.15 Restaking

Participants may increase their existing stake at any time.

Increasing stake may unlock additional protocol capabilities without requiring a new identity or participant registration.

Examples include:

* Publishing additional advertisements.
* Operating more infrastructure.
* Expanding merchant capacity.

Reductions in stake become effective only after the unlock period concludes.

---

## 15.16 Staking Transparency

Every staking position is publicly verifiable.

Participants may inspect:

* Total stake.
* Stake history.
* Active protocol roles.
* Unlock schedules.
* Historical penalties.
* Reward history.

Transparency promotes confidence and enables independent verification by every OpenFiat implementation.

---

## 15.17 Staking and Governance

Although governance determines many staking parameters, it cannot arbitrarily confiscate participant stake.

Any penalties must result from documented protocol rules and deterministic processes.

Examples include:

* Arbitration outcomes.
* Proven oracle misconduct.
* Objective node performance failures.
* Other governance-approved protocol violations.

This distinction preserves participant confidence while allowing governance to evolve operational parameters over time.

---

## 15.18 Relationship to the Three Economic Loops

Staking forms the foundation of OpenFiat's **Security Loop**.

By locking OPEN, participants provide the economic guarantees that secure merchants, arbitrators, infrastructure providers, and other protocol services.

The Utility Loop generates demand for OPEN through marketplace activity.

The Treasury Loop reinvests protocol revenue into ecosystem growth.

Together, these three loops create a self-reinforcing economic system in which honest participation strengthens both network security and long-term sustainability.

---

## 15.19 Why Staking Matters

OpenFiat replaces centralized trust with measurable economic commitment.

Rather than asking participants to trust a company, the protocol encourages trust through transparent staking, documented responsibilities, deterministic enforcement, and carefully aligned incentives.

Every major participant has something meaningful at stake, ensuring that the long-term health of the network remains in everyone's shared interest.

---

## 15.20 Looking Ahead

Staking secures the protocol by aligning incentives and accountability.

The next chapter explains how protocol fees flow through the ecosystem, introducing the Security Loop, Utility Loop, and Treasury Loop that together sustain OpenFiat's long-term economic model.
