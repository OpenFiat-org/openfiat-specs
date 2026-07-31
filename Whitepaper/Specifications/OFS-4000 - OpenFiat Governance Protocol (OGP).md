# OFS-4000 — OpenFiat Governance Protocol (OGP)

**Document ID:** OFS-4000

**Title:** OpenFiat Governance Protocol

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Governance

**Depends On:** OFS-1000, OFS-3000, OFS-5000

---

## Abstract

The OpenFiat Governance Protocol (OGP) defines how the OpenFiat protocol evolves without relying on a centralized authority.

It specifies how protocol improvement proposals are introduced, discussed, voted upon, approved, activated, and archived.

Governance enables the OpenFiat ecosystem—including token holders, infrastructure providers, merchants, developers, and the AllenHark Foundation—to collaboratively evolve the protocol while preserving stability, transparency, and decentralization.

The governance process modifies protocol rules only. It does not intervene in individual trades except through predefined emergency mechanisms approved by governance.

---

## 1. Introduction

Every decentralized protocol eventually evolves.

Examples include:

* New stablecoins
* New fiat currencies
* Improved dispute rules
* Updated reputation algorithms
* New service provider categories
* Security improvements
* Performance optimizations

Without governance, protocol evolution depends on a centralized organization.

OpenFiat instead defines governance as a protocol.

---

## 2. Scope

This specification defines:

* Governance proposals
* Proposal lifecycle
* Proposal categories
* Discussion
* Voting
* Quorum
* Execution
* Emergency governance
* Protocol upgrades

This specification does **not** define:

* Token economics
* Voting power calculations
* Treasury management
* Foundation operations

Those are defined in the Tokenomics and Foundation documents.

---

## 3. Design Goals

The Governance Protocol SHALL:

* Support decentralized evolution.
* Remain transparent.
* Prevent governance attacks.
* Encourage informed participation.
* Preserve protocol stability.
* Minimize unnecessary protocol fragmentation.

---

## 4. Governance Philosophy

Governance exists to improve the protocol.

It does **not** exist to manage users.

Governance SHALL NOT:

* Approve individual trades.
* Moderate marketplace activity.
* Override valid settlements.
* Change completed transactions.

Governance changes protocol rules—not protocol history.

---

## 5. Governance Participants

Governance may involve multiple stakeholder groups.

Examples include:

* OPEN Token Holders
* Node Operators
* Infrastructure Providers
* Merchants
* Developers
* Foundation Representatives (during bootstrap)

Each participant contributes to protocol evolution in different ways.

---

## 6. Bootstrap Governance

During the initial growth phase, the protocol is funded and developed primarily by AllenHark.

During this bootstrap period:

* AllenHark leads protocol development.
* Community proposals are encouraged.
* Governance decisions remain publicly documented.
* Progressive decentralization is planned.

Bootstrap governance is temporary.

Long-term control transitions to decentralized governance.

---

## 7. Governance Proposal (OFP)

Every governance change begins with an OpenFiat Proposal (OFP).

Each proposal receives:

* Proposal Number
* Title
* Author
* Category
* Version
* Status

Example:

```text
OFP-0042

Increase Reservation Timeout

Status:

Voting
```

---

## 8. Proposal Categories

Examples include:

Protocol

* Network improvements
* Trading rules
* Security

Economics

* Reward parameters
* Treasury policies
* Fee adjustments

Marketplace

* Reputation
* Merchant tiers
* Risk engine

Infrastructure

* Node incentives
* Service registry
* Oracle framework

Governance

* Voting mechanisms
* Proposal process
* Constitutional changes

---

## 9. Proposal Lifecycle

Every proposal follows the same lifecycle.

```text id="proposal-lifecycle"
Draft

↓

Community Discussion

↓

Formal Submission

↓

Technical Review

↓

Voting

↓

Accepted

or

Rejected

↓

Implementation

↓

Activation
```

Every proposal remains permanently archived.

---

## 10. Proposal Requirements

A proposal SHOULD include:

* Problem statement
* Motivation
* Technical specification
* Security considerations
* Backward compatibility
* Upgrade strategy
* Implementation timeline

Major protocol changes SHOULD include a reference implementation.

---

## 11. Community Discussion

Before voting:

The proposal enters public discussion.

Goals include:

* Technical review
* Economic review
* Community feedback
* Risk analysis

Open discussion generally produces stronger proposals.

---

## 12. Technical Review

Protocol maintainers review:

* Security
* Compatibility
* Complexity
* Network impact
* Economic implications

Technical review informs voters.

It does not replace voting.

---

## 13. Voting

After review:

Eligible participants vote.

Voting options include:

* Approve
* Reject
* Abstain

Voting periods are governance configurable.

---

## 14. Voting Power

The exact voting model is defined by the Tokenomics specification.

Possible inputs include:

* OPEN tokens
* Delegated voting
* Staking
* Reputation weighting (future)

The governance protocol intentionally separates voting mechanics from governance workflow.

---

## 15. Quorum

Proposals require minimum participation before becoming valid.

Quorum prevents:

* Extremely small groups changing protocol rules.
* Low-participation governance attacks.

Quorum requirements are governance configurable.

---

## 16. Proposal Approval

A proposal becomes Approved when:

* Voting period ends.
* Quorum is satisfied.
* Approval threshold is reached.

Approval alone does not immediately change protocol behavior.

### 16.1 Where the approval decision is made

The three conditions above are evaluated **on chain**, by the
`openfiat-governance` program's `tally_and_finalize` (OFS-4200 §6), over
stake-weighted votes any node can verify. A node MUST adopt that result rather
than compute its own.

This is not a stylistic preference. A node only records a vote once it has
verified the voter's weight against their on-chain stake account, so the vote set
a node holds is a function of its own connectivity: an RPC-connected node and a
gossip-only node tallying the same proposal reach different answers, both
correctly. Worse, a node holding no verifiable votes cannot distinguish "nobody
voted" from "I discarded what I could not verify", so a local tally would report
a confident rejection where the honest answer is "I do not know". A node MUST
NOT resolve a proposal from the votes it happens to hold.

### 16.2 Adopting the on-chain result

A proposal that goes on chain carries `onchain_proposal_id`, the `u64` of its
on-chain counterpart, inside its signed `ProposalCreate` event — fixed at
creation, because no event amends it. The on-chain proposal names the off-chain
one back. **Both claims are required**; the exact mechanism, the digest, and why
one claim alone must never be treated as a link are specified in OFS-4200 §6.1.

A node MUST NOT adopt an outcome from an on-chain proposal that has not named it
back, and MUST distinguish, when reporting to a client:

* no on-chain counterpart was ever claimed;
* a claim that is not reciprocated (including one this node has not yet been able
  to check);
* linked, chain still voting;
* linked, chain resolved, not yet adopted locally;
* linked and in agreement;
* linked and in disagreement.

The fourth and the sixth are different facts and collapsing them is an error:
every linked proposal passes through the fourth on its way to the fifth. A
proposal with no on-chain counterpart is not an error — informational proposals
legitimately never go on chain — and remains resolvable only by the off-chain
lifecycle events of §21 and §18.

---

## 17. Implementation

Approved proposals move into implementation.

Implementation includes:

* Development
* Testing
* Reference client updates
* Documentation
* Security review

Only completed implementations proceed to activation.

---

## 18. Protocol Activation

Activation occurs using scheduled protocol upgrades.

Example:

```text id="activation"
Proposal Approved

↓

Reference Implementation

↓

Network Upgrade

↓

Protocol Version Increased

↓

Feature Active
```

Activation schedules provide operators sufficient time to upgrade.

---

## 19. Emergency Governance

Exceptional circumstances may require accelerated governance.

Examples include:

* Critical vulnerabilities
* Consensus bugs
* Severe economic exploits
* Infrastructure attacks

Emergency procedures SHOULD require higher approval thresholds than ordinary proposals.

---

## 20. Constitutional Changes

Certain protocol principles are foundational.

Examples:

* Self-custody
* Open participation
* Cryptographic verification
* Deterministic execution

Changes affecting constitutional principles SHOULD require stronger approval requirements than ordinary proposals.

---

## 21. Proposal Withdrawal

Proposal authors MAY withdraw proposals before voting concludes.

Withdrawn proposals remain archived.

Historical governance records are never deleted.

---

## 22. Proposal Supersession

A newer proposal may replace an older one.

Example:

```text
OFP-32

Superseded by

OFP-57
```

Historical documents remain accessible.

---

## 23. Governance Transparency

Every governance action is publicly recorded.

Examples include:

* Proposal creation
* Discussion timeline
* Vote totals
* Activation status
* Implementation history

Transparency builds long-term confidence.

---

## 24. Governance Security

Implementations SHOULD defend against:

* Vote manipulation
* Replay attacks
* Proposal forgery
* Identity spoofing
* Duplicate voting
* Unauthorized activation

Every governance action MUST be digitally signed.

---

## 25. Governance Upgrades

The governance protocol itself may evolve.

Future versions MAY introduce:

* Quadratic voting
* Delegated governance
* Domain-specific committees
* Reputation-assisted governance
* Treasury governance
* Cross-chain governance

The protocol intentionally supports long-term evolution.

---

## 26. Conformance

A compliant implementation MUST:

* Support proposal creation.
* Preserve immutable proposal history.
* Support proposal discussion.
* Support deterministic proposal states.
* Support voting.
* Support quorum validation.
* Support protocol activation.
* Support digitally signed governance events.
* Adopt approval outcomes from the on-chain tally rather than computing them
  locally (§16.1), and never report a locally-derived tally as a resolution.
* Report the off-chain/on-chain link state honestly, including the case where it
  cannot corroborate a claim (§16.2).

---

## 27. Relationship to Other Specifications

The Governance Protocol coordinates the evolution of every OpenFiat specification.

```text id="governance-architecture"
          OFS Specifications
                  │
                  ▼
             Community
                  │
                  ▼
            OFS-4000
      Governance Protocol
                  │
      ┌───────────┼────────────┐
      ▼           ▼            ▼
 Protocol     Economics    Upgrades
 Changes        Changes      Activation
```

---

## 28. Summary

The OpenFiat Governance Protocol provides a transparent, structured, and decentralized process for evolving the OpenFiat ecosystem.

Recognizing that the network begins with AllenHark as its primary developer and sponsor, the protocol explicitly supports a bootstrap governance phase while committing to progressive decentralization over time.

By separating proposal creation, technical review, community discussion, voting, implementation, and activation into distinct stages, OpenFiat ensures that protocol evolution is deliberate, auditable, and technically sound.

The Governance Protocol answers one essential question:

**"How can a decentralized financial protocol continue to evolve without depending on permanent centralized control?"**
