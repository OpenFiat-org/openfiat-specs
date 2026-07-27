# Chapter 7 — The OpenFiat Trade Protocol (OFTP)

## 7.1 Introduction

The OpenFiat Trade Protocol (OFTP) defines the complete lifecycle of every trade conducted on the OpenFiat network.

Every compliant wallet, web application, desktop client, mobile application, merchant platform, and trading tool follows the same sequence of events, ensuring that participants enjoy a consistent and predictable experience regardless of which software they choose.

Unlike centralized exchanges, where the operator controls trade execution, OpenFiat coordinates trades through an open protocol while relying on Solana smart contracts for secure escrow and settlement.

The protocol standardizes:

* Advertisement reservation.
* Escrow funding.
* Buyer and merchant communication.
* Fiat payment confirmation.
* Evidence submission.
* Dispute initiation.
* Arbitration.
* Settlement.
* Reputation updates.

Every trade follows the same deterministic state machine.

---

## 7.2 Design Objectives

The OpenFiat Trade Protocol was designed around six primary objectives.

### Familiar Experience

The trading experience should resemble existing peer-to-peer cryptocurrency exchanges, minimizing the learning curve for new users.

### Immediate Escrow Protection

Stablecoins should enter escrow before fiat payment begins, ensuring that buyers never send fiat without funds already being secured.

### Deterministic Behavior

Every client should interpret trade state identically.

Two compliant applications observing the same signed events should always display the same order status.

### Fast Coordination

Trade coordination occurs through the OpenFiat network while financial settlement occurs on Solana.

### Recoverability

Trades should survive application restarts, temporary internet outages, and device changes through signed session synchronization.

### Fairness

No merchant or client application receives special privileges beyond those explicitly defined by the protocol.

---

## 7.3 The Trade Lifecycle

Every OpenFiat trade progresses through a fixed sequence of states.

```text
Advertisement

↓

Reservation (Requested → Validated → Accepted → Escrow Locked)

↓

Awaiting Payment

↓

Payment Sent ("I Paid")

↓

Merchant Reviewing

↓

Settlement
     │
     ├──────────────┐
     │              │
Approved         Dispute
     │              │
Escrow Released  Arbitration
     │              │
     └──── Completed ────→ Reputation Update
```

This is a narrative simplification of two formally-specified state machines: the Reservation state machine (OFS-2200 §18), which governs everything up to `Escrow Locked`, and the Settlement state machine (OFS-2300 §20), which governs everything from `Escrow Locked` onward. This chapter does not define either machine independently — see those specifications for the authoritative transitions.

A trade may never skip mandatory states.

This deterministic progression ensures that all implementations interpret trade history consistently.

---

## 7.4 Step 1 — Advertisement Discovery

Every trade begins when a buyer discovers an advertisement.

Advertisements may be filtered by:

* Country.
* Currency.
* Stablecoin.
* Payment method.
* Price.
* Merchant reputation.
* Merchant specialization.
* Trade limits.
* Availability.

Because advertisements are propagated through OFNP, every compatible client accesses the same decentralized marketplace.

No application owns exclusive liquidity.

---

## 7.5 Step 2 — Reservation

Once a buyer selects an advertisement, the buyer submits a reservation request.

Reservations are processed on a **first-come, first-served** basis.

If multiple buyers attempt to reserve the same available liquidity simultaneously, the protocol resolves the conflict deterministically using the same transaction ordering guarantees provided by Solana.

Only one reservation succeeds.

All unsuccessful reservations receive immediate rejection and may retry against another advertisement.

This prevents overselling while maintaining fairness.

---

## 7.6 Step 3 — Escrow Lock

What happens immediately after a reservation is accepted depends on the trade's direction, because OpenFiat uses two different vault models (Chapter 8).

**When the merchant is selling stablecoins:** the funds are already sitting in the merchant's Liquidity Vault — deposited before the advertisement could even go live. No new transfer occurs at reservation time. The protocol simply marks the reserved portion of the existing vault balance as locked and unavailable to other buyers.

**When the merchant is buying stablecoins:** there is no pre-existing inventory to lock, because the merchant doesn't hold the asset yet. Instead, the counterparty selling stablecoins into the merchant's buy advertisement deposits the stablecoins into a newly-created Trade Escrow Vault at this point, immediately after their reservation is accepted.

In both cases, once this step completes:

* Neither party can unilaterally move the escrowed funds.
* The buyer cannot yet receive them.
* The protocol now governs their release.

Escrow guarantees that the traded liquidity genuinely exists before fiat payment begins.

This eliminates one of the most common fraud scenarios found in informal peer-to-peer trading.

---

## 7.7 Step 4 — Trade Session

Once escrow has been funded, a secure trade session is established.

The session enables participants to exchange protocol-defined messages, including:

* Payment instructions.
* Additional verification requests.
* Payment confirmations.
* Evidence attachments.
* Trade status updates.

Sessions are cryptographically signed and synchronized across the OpenFiat network, allowing participants to recover active trades after temporary disconnections.

The protocol standardizes message exchange while allowing clients flexibility in how those messages are presented to users.

---

## 7.8 Step 5 — Fiat Payment

The buyer submits payment using one of the merchant's supported payment methods.

Examples include:

* Bank transfer.
* Mobile money.
* Instant payment systems.
* Domestic payment applications.
* Cash deposit.

Because these payment systems operate outside the blockchain, OpenFiat does not attempt to verify payment directly.

Instead, the protocol coordinates communication between participants while recording signed declarations regarding payment progress.

---

## 7.9 Step 6 — "I Paid"

After submitting payment, the buyer selects **"I Paid."**

This action is a signed protocol event.

Unlike many existing platforms, OpenFiat allows this declaration to be withdrawn if a genuine mistake occurs, provided the merchant has not yet acknowledged or acted upon it.

When marking a payment as complete, the buyer may immediately attach supporting evidence, such as:

* Payment receipts.
* Transaction confirmations.
* Screenshots.
* Reference numbers.

This evidence becomes part of the trade record should arbitration later become necessary.

---

## 7.10 Step 7 — Merchant Verification

The merchant verifies that fiat payment has been received.

Depending on the selected payment method, this may involve checking:

* Bank balances.
* Mobile money accounts.
* Payment applications.
* Reference numbers.
* Additional customer verification.

Merchants may request further clarification if required.

All communication remains associated with the signed trade session.

---

## 7.11 Step 8 — Settlement

If payment is confirmed, the merchant authorizes settlement.

The OpenFiat escrow program releases the stablecoins directly to the buyer's wallet.

Settlement is performed entirely by the Solana smart contract.

Neither the merchant nor any OpenFiat node can alter the settlement rules once escrow has been created.

---

## 7.12 Disputes

If participants disagree regarding payment, either party may initiate a dispute.

The dispute process intentionally pauses escrow release.

A new arbitration case is created and published to the network.

Qualified arbitrators voluntarily join the case by staking the required amount of OPEN.

Only after joining does an arbitrator receive access to case evidence, reducing opportunities for targeted bribery before participation.

Arbitrators vote using a commit-and-reveal scheme.

Consensus determines the outcome.

The escrow program executes the final decision automatically.

The complete dispute protocol is described in a later chapter.

---

## 7.13 Trade Completion

A trade is considered complete only after:

* Escrow has been released.
* Protocol fees have been distributed.
* Reputation updates have been recorded.
* Merchant statistics have been updated.
* Buyer statistics have been updated.
* Session state has been archived.

Only then does the trade become part of each participant's permanent trading history.

---

## 7.14 Fault Recovery

Real-world networks are imperfect.

Participants lose connectivity.

Devices restart.

Applications crash.

OpenFiat is designed with these realities in mind.

Because trade sessions are signed and synchronized across multiple nodes, participants may reconnect using another compatible client without losing active trade state.

This significantly improves reliability compared to centralized platforms that depend upon a single application server maintaining session information.

---

## 7.15 Security Guarantees

The OpenFiat Trade Protocol provides several important guarantees.

### Escrow Guarantee

Stablecoins are secured before fiat payment begins.

### Deterministic State

Every compliant client interprets trade progress identically.

### Non-Custodial Design

OpenFiat never permanently controls user funds.

### Recoverable Sessions

Trades survive temporary outages.

### Cryptographic Accountability

Every significant action is digitally signed.

### Transparent Arbitration

Dispute outcomes are executed according to publicly documented protocol rules rather than discretionary administrator decisions.

---

## 7.16 Why OFTP Matters

Trading digital assets for fiat currency requires coordination between blockchain transactions and real-world payment systems.

Traditional exchanges solve this problem through centralized infrastructure.

OpenFiat solves it through an open protocol.

By standardizing every stage of the trade lifecycle, OFTP enables independent software implementations to interoperate while preserving user ownership, marketplace transparency, and decentralized governance.

The protocol transforms peer-to-peer trading from a website-specific feature into a globally accessible public standard.

---

## 7.17 Looking Ahead

Every OpenFiat trade depends upon one critical component: the escrow system.

The following chapter examines how OpenFiat secures stablecoins using Solana smart contracts, how vaults are created, how protocol-controlled escrow functions, how fees are distributed, and why users retain confidence that funds can only move according to deterministic protocol rules.
