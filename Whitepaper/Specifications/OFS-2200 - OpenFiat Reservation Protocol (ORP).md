# OFS-2200 — Reservation Protocol (ORP)

**Document ID:** OFS-2200

**Title:** OpenFiat Reservation Protocol

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Trading

**Depends On:** OFS-1000, OFS-1200, OFS-1400, OFS-2000, OFS-2100

---

# Abstract

The OpenFiat Reservation Protocol (ORP) defines how liquidity is reserved within the OpenFiat marketplace.

Its primary objectives are to eliminate double-selling, ensure deterministic allocation of liquidity, immediately create on-chain escrow, and provide every participant in the decentralized network with an identical view of reservation state.

Reservations are processed using deterministic first-come, first-served ordering. Once accepted, liquidity becomes unavailable to every other participant until the reservation is completed, cancelled, or expires.

---

# 1. Introduction

Every trade begins with a reservation.

Without reservations:

* Two buyers could purchase the same liquidity.
* Merchants would need manual inventory management.
* Race conditions would become common.
* Different nodes could disagree about marketplace state.

The Reservation Protocol solves these problems by introducing a deterministic reservation process synchronized across the OpenFiat network.

---

# 2. Scope

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

# 3. Design Goals

The protocol SHALL:

* Prevent double-selling.
* Prevent double-buying.
* Lock liquidity immediately.
* Produce deterministic results.
* Scale to millions of reservations.
* Recover automatically after failures.

---

# 4. Reservation Philosophy

Reservations represent a temporary exclusive claim on advertised liquidity.

A reservation does **not** complete a trade.

Instead, it guarantees that:

* Liquidity cannot be sold elsewhere.
* Escrow has been created.
* Both parties may safely proceed with fiat settlement.

---

# 5. Reservation Lifecycle

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

Escrow Created

↓

Fiat Settlement

↓

Completed

or

Cancelled

or

Expired
```

Every transition generates a signed protocol event.

---

# 6. Reservation Request

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

# 7. Reservation Validation

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

# 8. First-Come, First-Served

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

# 9. Simultaneous Requests

Two nodes may receive competing reservation requests at nearly the same time.

To guarantee identical marketplace state across the network, OpenFiat resolves conflicts using deterministic ordering principles inspired by Solana's duplicate transaction handling.

Inputs MAY include:

* Reservation Timestamp
* Session Sequence Number
* Network Arrival Order
* Reservation Hash

Every compliant implementation SHALL reach the same result.

---

# 10. Immediate Escrow Creation

Immediately after reservation succeeds:

The OpenFiat Program automatically creates escrow.

Users never manually initiate escrow creation.

---

### Selling Stablecoins

```text id="escrow-seller"
Merchant Wallet

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

# 11. Reservation Ownership

A reservation belongs exclusively to one buyer (or seller, depending on trade direction).

Reservations cannot be transferred between wallets.

---

# 12. Reservation Timeout

Reservations expire automatically if settlement does not progress.

Timeout values are configurable by governance.

Example:

```text id="reservation-timeout"
Reservation

↓

20 Minutes

↓

No Progress

↓

Reservation Expired

↓

Liquidity Returned
```

---

# 13. Reservation Extension

Merchants MAY approve reservation extensions.

Typical reasons include:

* Banking delays
* Mobile money delays
* Regional payment outages

Extensions generate signed protocol events.

---

# 14. Reservation Cancellation

Reservations may be cancelled due to:

* User cancellation
* Merchant cancellation (where permitted)
* Payment timeout
* Escrow failure
* Protocol failure
* Governance intervention (future emergency mechanisms)

Cancellation immediately releases reserved liquidity.

---

# 15. Automatic Inventory Updates

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

# 16. Reservation Recovery

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

# 17. Reservation Synchronization

Reservation events include:

* ReservationCreated
* ReservationConfirmed
* ReservationExtended
* ReservationCancelled
* ReservationExpired

All events propagate using OFS-1200.

---

# 18. Reservation State Machine

Each reservation exists in exactly one state.

```text id="reservation-state-machine"
Requested

↓

Validated

↓

Accepted

↓

Escrow Locked

↓

Payment Pending

↓

Payment Sent

↓

Settlement Pending

↓

Completed

or

Cancelled

or

Expired
```

Illegal state transitions MUST be rejected.

---

# 19. Reservation Conflicts

Conflict examples include:

* Insufficient liquidity
* Duplicate reservations
* Expired advertisements
* Conflicting updates

Conflict resolution is deterministic.

Every compliant node SHALL reach identical outcomes.

---

# 20. Reservation Notifications

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

# 21. Security Considerations

Implementations MUST reject:

* Duplicate Reservation IDs
* Invalid signatures
* Expired sessions
* Invalid advertisement references
* Negative reservation amounts
* Replay attacks
* Unauthorized reservation updates

Escrow creation MUST remain fully deterministic.

---

# 22. Performance Considerations

Reservation processing is latency-sensitive.

Implementations SHOULD optimize:

* Fast validation
* Immediate escrow creation
* Efficient inventory updates
* Minimal propagation latency

Reservation events SHOULD receive high network priority under OFS-1600 (SWQoS).

---

# 23. Conformance

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

# 24. Relationship to Other Specifications

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
