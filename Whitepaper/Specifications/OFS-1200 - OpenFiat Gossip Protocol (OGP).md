# OFS-1200 — Gossip Protocol (OGP)

**Document ID:** OFS-1200

**Title:** OpenFiat Gossip Protocol

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Network

**Depends On:** OFS-1000, OFS-1100

---

## Abstract

The OpenFiat Gossip Protocol (OGP) defines how information propagates throughout the OpenFiat network.

OGP enables decentralized dissemination of advertisements, orders, trade state changes, governance events, reputation updates, service registrations, oracle updates, and other protocol messages without requiring centralized infrastructure.

The protocol is optimized for:

* Low latency
* High reliability
* Duplicate suppression
* Horizontal scalability
* Eventual consistency
* Network efficiency

OGP does **not** define message contents.

Individual OFS specifications define their respective message schemas.

---

## 1. Introduction

The OpenFiat network is fundamentally event-driven.

Every significant action performed anywhere in the network eventually becomes visible everywhere else.

Examples include:

* Merchant creates an advertisement.
* User reserves an order.
* Merchant confirms settlement.
* Oracle publishes an updated FX price.
* Governance proposal is created.
* Notification provider registers.
* Risk provider flags a wallet.

These events are propagated through the Gossip Protocol.

Rather than every node querying every other node continuously, information spreads naturally across the peer-to-peer network.

---

## 2. Scope

This specification defines:

* Event propagation
* Gossip message lifecycle
* Duplicate suppression
* Event validation
* Peer forwarding
* Time-to-live (TTL)
* Event identifiers
* Propagation priorities
* Reliability

This specification does **not** define:

* Event schemas
* Snapshot synchronization
* Service registration
* Trade logic
* Reputation algorithms

---

## 3. Design Goals

The Gossip Protocol SHALL:

* Deliver events quickly.
* Avoid broadcast storms.
* Prevent duplicate propagation.
* Scale to millions of events.
* Recover from temporary partitions.
* Operate without centralized coordination.

---

## 4. Event Model

Everything propagated by the network is an **Event**.

Examples:

* AdvertisementCreated
* AdvertisementUpdated
* AdvertisementRemoved
* OrderReserved
* EscrowLocked
* PaymentMarked
* SettlementCompleted
* DisputeOpened
* GovernanceVote
* ReputationUpdated
* OraclePriceUpdated
* WalletFlagged

Events are immutable.

If state changes, a **new event** is generated.

---

## 5. Event Identity

Every event MUST possess a globally unique Event ID.

The Event ID SHALL be deterministically generated from:

* Event Type
* Payload
* Timestamp
* Sender Identity
* Digital Signature

Duplicate Event IDs MUST represent identical events.

---

## 6. Event Envelope

Every gossip event SHALL be transported inside a common envelope.

Conceptually:

```text id="gossip-envelope"
Event Envelope

├── Event ID
├── Event Type
├── OFS Specification
├── Version
├── Origin Node
├── Timestamp
├── TTL
├── Priority
├── Signature
└── Payload
```

The payload format is defined by the originating OFS specification.

---

## 7. Event Origination

Any authenticated node MAY originate events for services it legitimately provides.

Examples:

Merchant Gateway:

* AdvertisementCreated
* AdvertisementUpdated

Oracle Provider:

* FXPriceUpdated

Notification Gateway:

* NotificationStatus

Risk Intelligence Provider:

* WalletFlagged

Node implementations MUST reject unauthorized event types.

---

## 8. Gossip Lifecycle

Every event follows the same lifecycle.

```text id="gossip-lifecycle"
Event Created

↓

Local Validation

↓

Signed

↓

Stored

↓

Broadcast

↓

Peer Validation

↓

Forwarded

↓

Eventually Expires
```

No event is forwarded before successful validation.

---

## 9. Local Validation

Before broadcasting an event, the originating node SHALL verify:

* Schema correctness
* Required fields
* Signature validity
* Protocol version
* Event authorization

Invalid events MUST NOT enter the gossip network.

---

## 10. Event Storage

Every received event SHALL be temporarily stored.

The reference implementation stores events in RocksDB.

Storage serves several purposes:

* Duplicate detection
* Recovery
* Replay prevention
* Late peer synchronization

Retention policies are implementation configurable.

---

## 11. Duplicate Suppression

Duplicate suppression is mandatory.

When a node receives an event whose Event ID already exists locally:

```text id="duplicate-flow"
Receive Event

↓

Event Exists?

↓

YES

↓

Discard

↓

NO

↓

Validate

↓

Store

↓

Forward
```

Duplicate suppression prevents exponential network growth.

---

## 12. Time-To-Live (TTL)

Every event includes a TTL.

Each forwarding operation decrements the TTL.

Example:

```text id="ttl-example"
TTL = 8

↓

7

↓

6

↓

5

↓

...

↓

0

↓

Discard
```

Events reaching zero SHALL NOT be forwarded further.

---

## 13. Forwarding Strategy

Nodes SHOULD forward events to all eligible peers except:

* The peer that sent the event.
* Peers lacking required protocol support.
* Peers already known to possess the event.

Implementations MAY batch multiple events into a single transmission.

---

## 14. Event Priorities

Not all events are equally important.

Recommended priority levels:

Priority 1

* Session Control

Priority 2

* Reservation Events
* Escrow Events
* Settlement Events

Priority 3

* Advertisement Updates
* Merchant Availability

Priority 4

* Reputation Updates
* Oracle Updates

Priority 5

* Governance

Priority 6

* Background Synchronization

Higher-priority events SHOULD be transmitted first.

---

## 15. Event Ordering

The Gossip Protocol does not guarantee global ordering.

Applications MUST tolerate:

* Out-of-order delivery
* Duplicate delivery
* Delayed delivery

Higher-level specifications determine how conflicting events are resolved.

---

## 16. Eventual Consistency

OGP guarantees eventual consistency rather than immediate consistency.

Given stable connectivity:

Every valid event SHALL eventually reach every interested node.

Temporary delays are expected.

Permanent divergence is not.

---

## 17. Network Partitions

If the network partitions:

```text id="partition-gossip"
Partition

↓

Events Continue Locally

↓

Connectivity Restored

↓

Missing Events Exchanged

↓

State Converges
```

No manual intervention is required.

---

## 18. Selective Gossip

Nodes MAY selectively subscribe to event categories.

Examples:

Merchant Gateway:

* Advertisement Events
* Order Events

Oracle Provider:

* Oracle Events

Notification Gateway:

* Notification Events

This reduces unnecessary bandwidth.

---

## 19. Gossip Channels

Events are logically separated into channels.

Examples:

```text id="channels"
Marketplace

Governance

Reputation

Notifications

Oracle

Risk Intelligence

Infrastructure
```

Nodes subscribe only to channels relevant to their services.

---

## 20. Bandwidth Optimization

Implementations SHOULD minimize bandwidth by:

* Compressing batches
* Aggregating events
* Avoiding duplicate forwarding
* Prioritizing recent events
* Removing expired events

Bandwidth efficiency becomes increasingly important as the network grows.

---

## 21. Reliability

A forwarded event is considered delivered only after local transmission succeeds.

Implementations MAY retry failed transmissions according to local policy.

Persistent delivery failures SHOULD reduce peer quality scores.

---

## 22. Recovery

Nodes recovering after downtime SHALL request missing events.

If the missing window exceeds local retention capacity, Snapshot Synchronization (OFS-1300) SHALL be used instead.

This minimizes unnecessary snapshot downloads.

---

## 23. Security Considerations

Nodes MUST reject:

* Invalid signatures
* Corrupted payloads
* Unauthorized event types
* Replay attacks
* Expired events
* Protocol version mismatches

Rate limiting SHOULD prevent malicious event flooding.

---

## 24. Performance Considerations

The Gossip Protocol is expected to distribute:

* Millions of advertisements
* Millions of order updates
* Continuous oracle updates
* Reputation changes
* Governance activity

Implementations SHOULD prioritize:

* Low latency
* High throughput
* Low memory overhead
* Efficient RocksDB indexing

---

## 25. Relationship to SWQoS

Certain OpenFiat event categories—particularly reservation, escrow, settlement, and dispute events—are latency-sensitive.

Node operators MAY implement **Stake-Weighted Quality of Service (SWQoS)**, where participating nodes allocate higher network priority to traffic originating from trusted or staked infrastructure providers.

SWQoS affects only **network transport priority**.

It MUST NOT:

* Change protocol rules.
* Modify event contents.
* Prevent delivery of valid events.
* Grant consensus authority.
* Allow censorship of compliant nodes.

Every valid event remains eligible for propagation.

SWQoS simply improves delivery quality under network congestion, similar to its role within the Solana ecosystem.

The rules governing SWQoS participation, staking requirements, penalties, and node reputation are specified separately in **OFS-1600 — Node Reputation & QoS**.

---

## 26. Conformance

A compliant implementation MUST:

* Generate globally unique Event IDs.
* Validate all received events.
* Suppress duplicates.
* Respect TTL.
* Verify signatures.
* Store recent events.
* Support selective subscriptions.
* Forward valid events.
* Reject malformed events.
* Support eventual consistency.

---

## 27. Relationship to Other Specifications

The Gossip Protocol is the event distribution layer of OpenFiat.

```text id="gossip-architecture"
               OFS-1000
          Network Protocol
                    │
                    ▼
               OFS-1100
            Peer Discovery
                    │
                    ▼
               OFS-1200
            Gossip Protocol
                    │
     ┌──────────────┼──────────────┐
     ▼              ▼              ▼
OFS-2000       OFS-3000      OFS-7000
Trade          Reputation     Oracle
                    │
                    ▼
             All Protocol Events
```

Every OpenFiat protocol that produces state-changing events relies on OGP to distribute those events across the decentralized network.

By separating event propagation from application logic, the Gossip Protocol provides a scalable, resilient, and extensible communication layer capable of supporting millions of participants while preserving OpenFiat's decentralized architecture.
