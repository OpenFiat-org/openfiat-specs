# OFS-2000 — OpenFiat Trade Protocol (OFTP)

**Document ID:** OFS-2000

**Title:** OpenFiat Trade Protocol

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Trading

**Depends On:** OFS-1000, OFS-1100, OFS-1200, OFS-1300, OFS-1400, OFS-1500

---

## Abstract

The OpenFiat Trade Protocol (OFTP) defines the complete decentralized trading lifecycle within the OpenFiat Network.

It specifies how buyers and merchants discover each other, reserve liquidity, automatically create escrow, exchange fiat and stablecoins, resolve disputes, update reputation, and complete settlement without relying on a centralized marketplace operator.

OFTP serves as the foundation upon which all higher-level marketplace protocols are built.

Individual protocol specifications define each stage in greater detail:

* OFS-2100 Advertisement Protocol
* OFS-2200 Reservation Protocol
* OFS-2300 Settlement Protocol
* OFS-2400 Dispute Protocol

---

## 1. Introduction

OpenFiat is a decentralized peer-to-peer stablecoin marketplace.

Unlike centralized exchanges, OpenFiat never takes custody of user accounts.

Users retain full control of their wallets.

The protocol coordinates trades by combining:

* On-chain escrow
* Off-chain fiat settlement
* Decentralized networking
* Cryptographic authentication
* Reputation systems
* Deterministic protocol rules

Every compliant implementation follows exactly the same trade lifecycle.

---

## 2. Scope

This specification defines:

* Marketplace participants
* Trade lifecycle
* Escrow creation
* Fiat settlement flow
* Trade state machine
* Trade synchronization
* Failure recovery
* Trade completion

This specification does **not** define:

* Advertisement structure (OFS-2100)
* Reservation rules (OFS-2200)
* Settlement messages (OFS-2300)
* Dispute procedures (OFS-2400)
* Reputation calculations (OFS-3000)

---

## 3. Design Goals

The protocol SHALL:

* Eliminate centralized custody.
* Preserve user sovereignty.
* Provide deterministic settlement.
* Prevent double-selling.
* Support millions of simultaneous trades.
* Remain blockchain independent where practical.
* Recover automatically from failures.

---

## 4. Marketplace Participants

A trade may involve several independent participants.

### Buyer

Purchases stablecoins.

### Merchant

Provides liquidity.

A merchant may:

* Sell stablecoins.
* Buy stablecoins.
* Perform both roles simultaneously.

### Network Nodes

Synchronize marketplace state.

### Notification Providers

Deliver trade notifications.

### Oracle Providers

Publish exchange rates.

### Risk Intelligence Providers

Evaluate wallet and fraud risk.

### Smart Contract

Manages escrow autonomously.

No participant can unilaterally complete or alter a trade outside the protocol rules.

---

## 5. Trading Models

OpenFiat supports two trading models.

### Merchant Selling Stablecoins

Merchant advertises:

> "I have stablecoins available."

Buyer pays fiat.

Merchant receives fiat.

Buyer receives escrowed stablecoins.

---

### Merchant Buying Stablecoins

Merchant advertises:

> "I want to purchase stablecoins."

Seller transfers stablecoins into escrow.

Merchant pays fiat.

Merchant receives escrowed stablecoins.

This model allows professional liquidity providers to continuously purchase inventory from the market.

---

## 6. Core Principle: Automatic Escrow

Escrow is always created automatically by the protocol.

Neither party manually locks funds.

The protocol determines which participant is providing stablecoin liquidity and locks only the required amount.

This prevents over-locking capital while guaranteeing settlement security.

---

## 7. Merchant Selling Flow

When selling stablecoins, the merchant must first deposit the advertised inventory into a **Liquidity Vault** — a persistent, program-owned escrow account defined in Chapter 8 and OFS-2300. A merchant can never advertise or sell more than they hold deposited in that vault; there is no path that sells directly from a personal, non-custodial wallet.

```text
Liquidity Vault (program-owned)

Available

10,000 USDC

↓

Advertisement

10,000 USDC

↓

Buyer Reserves

2,000 USDC

↓

Program Automatically Locks

2,000 USDC

↓

Remaining Available

8,000 USDC
```

Only the reserved amount is locked.

No transfer occurs at reservation time — the deposit already happened before the advertisement went live. The reservation only marks a portion of the already-deposited vault balance as unavailable to other buyers.

The remaining balance remains available for additional advertisements and trades.

---

## 8. Merchant Buying Flow

When buying stablecoins:

```text
Merchant Advertisement

Buying

KES 150,000

↓

Seller Accepts

↓

Seller Deposits

Equivalent USDC

↓

Program Locks Deposit

↓

Merchant Pays Fiat

↓

Escrow Released
```

The merchant does **not** pre-lock USDC because the merchant is purchasing it.

Instead, the seller deposits the stablecoins into escrow after accepting the advertisement.

---

## 9. Trade Lifecycle

Every trade is the composition of two sub-protocols, each with its own canonical state machine:

```text
Advertisement (OFS-2100)

↓

Reservation Protocol (OFS-2200)
  Requested → Validated → Accepted → Escrow Locked

↓

Settlement Protocol (OFS-2300)
  Awaiting Payment → Payment Sent → Merchant Reviewing → Approved → Escrow Released → Completed

↓

Reputation Update (OFS-3000)

↓

Trade Closed
```

OFS-2200 owns every state up to and including `Escrow Locked`; OFS-2300 owns every state from `Escrow Locked` onward. This specification does not redefine either state machine — see OFS-2200 §18 and OFS-2300 §20 for the authoritative definitions.

Every state transition generates a signed protocol event.

---

## 10. Reservation

Reservations are handled on a strict **first-come, first-served** basis.

If multiple reservation requests arrive simultaneously, duplicate resolution follows deterministic ordering principles similar to Solana transaction processing.

Only one reservation may succeed for the same liquidity.

All unsuccessful reservations receive immediate rejection.

---

## 11. Escrow Funding

Escrow funding occurs immediately after a successful reservation.

The protocol—not the merchant UI—creates the escrow.

This guarantees:

* Reserved liquidity cannot be double-sold.
* Buyers cannot reserve unavailable inventory.
* Marketplace state remains deterministic across every node.

---

## 12. Fiat Settlement

Fiat transfers occur outside the blockchain.

Supported payment methods include:

* Bank Transfer
* Mobile Money
* Instant Payment Networks
* Cash Deposit
* Other merchant-supported methods

Payment methods are advertised before reservation.

---

## 13. Payment Confirmation

Once fiat has been sent:

Buyer selects:

> **"I Paid."**

This action:

* Creates a signed protocol event.
* May include payment proof.
* May later be reversed before settlement if submitted accidentally.
* Notifies the merchant.

---

## 14. Payment Proof

Payment confirmation MAY include:

* Bank receipt
* Mobile money receipt
* Transaction reference
* Screenshot
* PDF receipt
* Additional documentation

Evidence becomes part of the dispute record if required.

---

## 15. Merchant Verification

Merchants MAY request additional verification before confirming settlement.

Examples include:

* Identity confirmation
* Payment clarification
* Additional receipt
* Bank reference number

These requests occur within the protocol.

---

## 16. Settlement

Once payment is verified:

```text
Merchant Approves

↓

Smart Contract Releases Escrow

↓

Stablecoins Delivered

↓

Trade Complete
```

No intermediary holds custody.

---

## 17. Automatic Completion

Future protocol versions MAY support fully automated settlement where payment verification can be performed through trusted providers.

Current protocol assumes merchant verification unless automated verification is available.

---

## 18. Failure Recovery

Network failures do not invalidate trades.

Because trade state is synchronized across nodes:

* Clients may reconnect.
* Sessions may migrate.
* Notifications may be re-sent.
* Settlement resumes.

No trade should be lost due to infrastructure failure.

---

## 19. Trade Cancellation

Trades may terminate before settlement for reasons including:

* Reservation timeout.
* Buyer abandonment.
* Seller abandonment.
* Payment timeout.
* Escrow expiration.

Cancellation rules are defined in OFS-2200.

---

## 20. Trade Completion

A trade completes only when:

* Escrow released successfully.
* Final state propagated.
* Reputation updated.
* Advertisement inventory adjusted.

Only then is liquidity considered available again.

---

## 21. Security Model

The protocol prevents:

* Double-selling.
* Double-reservation.
* Manual escrow manipulation.
* Duplicate settlement.
* Unauthorized releases.
* Replay attacks.
* Duplicate processing of an already-applied signed protocol event.

All state transitions are cryptographically authenticated. Every signed event MUST carry a unique identifier so that a duplicated or replayed event is detected and discarded rather than reapplied.

---

## 22. Scalability

The protocol is designed to support:

* Millions of advertisements.
* Millions of reservations.
* Concurrent settlements.
* Independent regional markets.
* Multiple stablecoins.
* Multiple fiat currencies.

No centralized matching engine is required.

---

## 23. Deterministic State

Every compliant node processing the same sequence of protocol events SHALL reach the same marketplace state.

Deterministic execution enables:

* Decentralized synchronization.
* Fast recovery.
* Snapshot generation.
* Reliable dispute reconstruction.

---

## 24. Relationship to Other Specifications

OFTP defines the complete trading lifecycle.

Individual protocol specifications expand each stage.

```text
              OFS-2000
      OpenFiat Trade Protocol
                    │
   ┌────────────────┼────────────────┐
   ▼                ▼                ▼
OFS-2100       OFS-2200        OFS-2300
Advertisement  Reservation     Settlement
                    │
                    ▼
              OFS-2400
           Dispute Protocol
                    │
                    ▼
              OFS-3000
          Reputation Engine
```

---

## 25. Conformance

A compliant implementation MUST:

* Support both buying and selling merchants.
* Automatically create escrow through the smart contract.
* Enforce first-come, first-served reservations.
* Synchronize deterministic trade state.
* Support signed payment confirmation.
* Support payment proof attachments.
* Support merchant verification requests.
* Prevent double-selling.
* Prevent duplicate settlement.
* Generate protocol events for every state transition.

---

## 26. Summary

The OpenFiat Trade Protocol is the operational heart of the OpenFiat ecosystem.

Rather than relying on a centralized exchange operator, OFTP coordinates decentralized trading through deterministic protocol rules, automatic smart contract escrow, peer-to-peer networking, and cryptographically authenticated state transitions.

Every compliant implementation behaves identically, allowing independent applications, merchants, and infrastructure providers to interoperate seamlessly while maintaining a secure, scalable, and censorship-resistant marketplace for stablecoin-to-fiat trading.
