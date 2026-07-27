# OFS-2200 — Reservation Protocol (ORP)

**Document ID:** OFS-2200

**Title:** OpenFiat Reservation Protocol

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Trading

**Depends On:** OFS-1000, OFS-1200, OFS-1400, OFS-2000, OFS-2100

---

## Abstract

The OpenFiat Reservation Protocol (ORP) defines how liquidity is reserved within the OpenFiat marketplace.

Its primary objectives are to eliminate double-selling, ensure deterministic allocation of liquidity, immediately create on-chain escrow, and provide every participant in the decentralized network with an identical view of reservation state.

Reservations are processed using deterministic first-come, first-served ordering. Once accepted, liquidity becomes unavailable to every other participant until the reservation is completed, cancelled, or expires.

---

## 1. Introduction

Every trade begins with a reservation.

Without reservations:

* Two buyers could purchase the same liquidity.
* Merchants would need manual inventory management.
* Race conditions would become common.
* Different nodes could disagree about marketplace state.

The Reservation Protocol solves these problems by introducing a deterministic reservation process synchronized across the OpenFiat network.

---

## 2. Scope

This specification defines:

* Reservation creation
* Reservation validation
* Reservation ordering
* Escrow creation
* Reservation expiration
* Reservation cancellation
* Reservation synchronization
* Reservation recovery
* Reservation conflicts

This specification does **not** define:

* Advertisements
* Settlement
* Disputes
* Reputation

---

## 3. Design Goals

The protocol SHALL:

* Prevent double-selling.
* Prevent double-buying.
* Lock liquidity immediately.
* Produce deterministic results.
* Scale to millions of reservations.
* Recover automatically after failures.

---

## 4. Reservation Philosophy

Reservations represent a temporary exclusive claim on advertised liquidity.

A reservation does **not** complete a trade.

Instead, it guarantees that:

* Liquidity cannot be sold elsewhere.
* Escrow has been created.
* Both parties may safely proceed with fiat settlement.

---

## 5. Reservation Lifecycle

Every reservation follows the same lifecycle.

```text id="reservation-lifecycle"
Advertisement

↓

Reservation Requested

↓

Validation

↓

Accepted

↓

Escrow Locked

↓

(handoff to Settlement Protocol, OFS-2300)

or

Cancelled

or

Expired
```

This specification's authority over a reservation ends at `Escrow Locked`. Everything from that point forward — fiat payment, payment confirmation, merchant review, escrow release — belongs to the Settlement Protocol (OFS-2300 §20), not to this reservation state machine. See §18 for the full Reservation State Machine.

Every transition generates a signed protocol event.

---

## 6. Reservation Request

A reservation request contains:

* Reservation ID
* Advertisement ID
* Trade Amount
* Wallet Address
* Session ID
* Timestamp
* Digital Signature

Every request MUST be signed by the requesting wallet.

---

## 7. Reservation Validation

Nodes SHALL verify:

* Advertisement exists.
* Advertisement is active.
* Requested amount is within limits.
* Liquidity is available.
* Wallet signature is valid.
* Session is authenticated.
* Protocol versions are compatible.

Only valid reservations proceed.

---

## 8. First-Come, First-Served

Reservations are allocated strictly in arrival order.

Example:

```text id="fcfs"
Advertisement

10,000 USDC

↓

Buyer A

Requests 6,000

↓

Buyer B

Requests 6,000

↓

Buyer A Accepted

↓

Buyer B Rejected

Remaining Liquidity

4,000 USDC
```

Arrival order is determined using deterministic network ordering rules.

---

## 9. Simultaneous Requests

Two nodes may receive competing reservation requests at nearly the same time.

To guarantee identical marketplace state across the network, OpenFiat resolves conflicts using deterministic ordering principles inspired by Solana's duplicate transaction handling.

Inputs MAY include:

* Reservation Timestamp
* Session Sequence Number
* Network Arrival Order
* Reservation Hash

Every compliant implementation SHALL reach the same result.

---

## 10. Immediate Escrow Creation

Immediately after reservation succeeds:

The OpenFiat Program automatically creates escrow.

Users never manually initiate escrow creation.

---

### Selling Stablecoins

```text id="escrow-seller"
Liquidity Vault (program-owned)

10,000 USDC

↓

Buyer Reserves

2,500 USDC

↓

Program Locks

2,500 USDC

↓

Available

7,500 USDC
```

The 10,000 USDC was already deposited into the merchant's Liquidity Vault before the advertisement could go live (OFS-2100 §10, OFS-2300 §6). "Program Locks" here means marking a portion of that existing vault balance as reserved — no new transfer occurs. A merchant can never have more reserved against an advertisement than they hold deposited in the vault.

---

### Buying Stablecoins

```text id="escrow-buyer"
Merchant Advertisement

Buying Stablecoins

↓

Seller Accepts

↓

Seller Deposits

2,500 USDC

↓

Program Locks

2,500 USDC
```

Only the required liquidity is locked.

---

## 11. Reservation Ownership

A reservation belongs exclusively to one buyer (or seller, depending on trade direction).

Reservations cannot be transferred between wallets.

---

## 12. Reservation Timeout

Reservations expire automatically if settlement does not progress.

Timeout values are configurable by governance.

Example:

```text id="reservation-timeout"
Reservation

↓

30 Minutes

↓

No Progress

↓

Reservation Expired

↓

Liquidity Returned
```

The 30-minute default applies to the reservation-validation window specifically (time to reach `Escrow Locked`). It is a protocol default, not a hard-coded constant — see §12a for the full timeout matrix, and OFS-2300 §8 for the separate payment-window timeout that begins once escrow is locked.

## 12a. Timeout Matrix

| Timeout | Phase | Default | Governance-Configurable | On Expiry |
|---|---|---|---|---|
| Reservation validation window | Requested → Escrow Locked | 30 minutes | Yes | Reservation expires; liquidity returned to available |
| Reservation extension | Accepted, awaiting escrow/payment delay | Merchant-approved, no fixed cap | Yes (max extension length) | Extension request denied or original deadline applies |
| Payment window | Escrow Locked → Payment Sent (OFS-2300 §8) | 30 minutes | Yes | Settlement cancelled; escrow returned |
| Merchant review window | Payment Sent → Approved/Rejected (OFS-2300 §12) | 30 minutes | Yes | Buyer may escalate to dispute (OFS-2400 §5) |

All defaults above are protocol parameters, not constants, and may be changed by a governance parameter-category proposal (OFS-4000).

---

## 13. Reservation Extension

Merchants MAY approve reservation extensions.

Typical reasons include:

* Banking delays
* Mobile money delays
* Regional payment outages

Extensions generate signed protocol events.

---

## 14. Reservation Cancellation

Before escrow is locked, cancellation is governed entirely by this specification. Reservations may be cancelled due to:

* User cancellation
* Merchant cancellation (where permitted)
* Payment timeout
* Escrow failure
* Protocol failure
* Governance intervention (future emergency mechanisms)

Cancellation immediately releases reserved liquidity.

**Before payment: either party may cancel, subject to protocol rules.** This applies for the whole reservation phase, up to and including the moment escrow is locked. Once escrow is locked and the trade moves into the Settlement Protocol (OFS-2300), cancellation rules change — after payment is marked sent, cancellation is restricted to prevent abuse. See OFS-2300 §19 for the full post-escrow cancellation rules and matrix.

---

## 15. Automatic Inventory Updates

Reservations automatically update advertisements.

Example:

```text id="inventory-update"
Advertisement

15,000 USDC

↓

Reservation

3,000 USDC

↓

Advertisement

12,000 USDC
```

Merchants never manually update inventory.

---

## 16. Reservation Recovery

If a node fails:

```text id="reservation-recovery"
Node Failure

↓

Session Migration

↓

Reservation Restored

↓

Settlement Continues
```

Reservations survive infrastructure failures because reservation state is synchronized across the network.

---

## 17. Reservation Synchronization

Reservation events include:

* ReservationCreated
* ReservationConfirmed
* ReservationExtended
* ReservationCancelled
* ReservationExpired

All events propagate using OFS-1200.

---

## 18. Reservation State Machine

Each reservation exists in exactly one state.

```text id="reservation-state-machine"
Requested

↓

Validated

↓

Accepted

↓

Escrow Locked

or

Cancelled

or

Expired
```

`Escrow Locked` is a terminal state **for this specification** — it is the handoff point to the Settlement Protocol (OFS-2300 §20), which owns every subsequent state (Awaiting Payment, Payment Sent, Merchant Reviewing, Approved, Escrow Released, Completed). This specification's scope (§2) explicitly excludes settlement, so its own state machine must not include settlement-phase states; earlier drafts of this document incorrectly extended the reservation state machine through payment and settlement states, which duplicated and could drift from OFS-2300's authoritative definitions. That has been corrected here.

Illegal state transitions MUST be rejected.

---

## 19. Reservation Conflicts

Conflict examples include:

* Insufficient liquidity
* Duplicate reservations
* Expired advertisements
* Conflicting updates

Conflict resolution is deterministic.

Every compliant node SHALL reach identical outcomes.

---

## 20. Reservation Notifications

Reservation events SHOULD trigger notifications.

Examples:

Buyer:

* Reservation Accepted
* Reservation Expiring
* Reservation Cancelled

Merchant:

* New Reservation
* Payment Pending
* Reservation Timeout

Notification delivery is defined separately in OFS-6000.

---

## 21. Security Considerations

Implementations MUST reject:

* Duplicate Reservation IDs
* Invalid signatures
* Expired sessions
* Invalid advertisement references
* Negative reservation amounts
* Replay attacks
* Unauthorized reservation updates
* Duplicate processing of an already-applied reservation event

Escrow creation MUST remain fully deterministic.

## Idempotency

Every reservation event (`ReservationCreated`, `ReservationExtended`, `ReservationCancelled`, `ReservationExpired`) carries a unique nonce. Implementations MUST detect a duplicate nonce and discard the redundant event rather than reapplying it — for example, a resent `ReservationCreated` for an already-accepted reservation MUST NOT create a second escrow lock against the same liquidity.

---

## 22. Performance Considerations

Reservation processing is latency-sensitive.

Implementations SHOULD optimize:

* Fast validation
* Immediate escrow creation
* Efficient inventory updates
* Minimal propagation latency

Reservation events SHOULD receive high network priority under OFS-1600 (SWQoS).

---

## 23. Conformance

A compliant implementation MUST:

* Enforce first-come, first-served reservations.
* Automatically create escrow.
* Prevent double-selling.
* Prevent duplicate reservations.
* Synchronize reservation state.
* Support reservation expiration.
* Support reservation cancellation.
* Support deterministic conflict resolution.
* Generate signed reservation events.

---

## 24. Relationship to Other Specifications

The Reservation Protocol bridges advertisements and settlement.

```text id="reservation-architecture"
             OFS-2100
      Advertisement Protocol
                  │
                  ▼
             OFS-2200
      Reservation Protocol
                  │
      ┌───────────┼────────────┐
      ▼           ▼            ▼
 Escrow      OFS-2300     OFS-2400
 Creation    Settlement    Disputes
                  │
                  ▼
           Reputation Update
```

The Reservation Protocol answers one critical question:

**"How can liquidity be allocated fairly, deterministically, and securely before fiat settlement begins?"**

By combining deterministic first-come, first-served ordering with automatic smart contract escrow, synchronized reservation state, and immediate inventory updates, the Reservation Protocol guarantees that every unit of liquidity can belong to only one active trade at a time, enabling a decentralized marketplace to operate safely at global scale.
