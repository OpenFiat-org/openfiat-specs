# OFS-1900 — OpenFiat Persistent State Storage Specification (PSSS)

**Document ID:** OFS-1900

**Title:** OpenFiat Persistent State Storage Specification

**Version:** 1.0.0 (Draft)

**Status:** Draft Standard

**Category:** Network Layer

**Depends On:** OFS-1000, OFS-1700

**Referenced By:** All OpenFiat Protocol Specifications

---

# Abstract

The OpenFiat Persistent State Storage Specification (PSSS) defines the canonical persistent data model for OpenFiat nodes.

Unlike traditional blockchain systems that persist ordered blocks, OpenFiat persists **deterministic protocol state**. The storage layer is responsible for maintaining the complete marketplace state, protocol objects, synchronization metadata, indexes, snapshots, and audit history required for correct node operation.

This specification defines **what** information must be persisted and **how** it is logically organized. It deliberately avoids mandating a specific database implementation.

The OpenFiat Reference Node uses **RocksDB** as its storage engine due to its performance, maturity, deterministic behavior, and embeddability. Alternative implementations MAY use different storage engines provided they satisfy all requirements defined by this specification.

---

# 1. Introduction

Every OpenFiat node maintains a persistent local copy of protocol state.

Persistent storage enables a node to:

* Recover after restart.
* Resume synchronization.
* Verify protocol history.
* Execute deterministic state transitions.
* Maintain marketplace integrity.
* Serve RPC queries efficiently.

Storage is local to each node.

It is never shared directly between nodes.

Network synchronization occurs exclusively through the OpenFiat Network Protocol.

---

# 2. Scope

This specification defines:

* Persistent state model
* Logical storage layout
* Object persistence
* Indexing
* State snapshots
* Audit history
* Versioning
* Integrity validation
* Storage recovery
* Pruning
* Backup considerations

This specification does **not** define:

* Database implementation
* File formats
* Snapshot transport
* Compression algorithms
* Filesystem selection

---

# 3. Design Goals

The storage layer SHALL:

* Preserve deterministic state.
* Support fast lookups.
* Support atomic updates.
* Recover safely after crashes.
* Scale to millions of objects.
* Support efficient synchronization.
* Remain implementation independent.

---

# 4. Design Philosophy

Persistent storage represents **current protocol state**, not an append-only blockchain.

Objects are updated as protocol events occur.

Historical events remain available through the audit log, while active state reflects the latest valid version of every protocol object.

This significantly reduces storage requirements and improves query performance while preserving determinism.

---

# 5. Reference Storage Engine

The OpenFiat Reference Implementation uses RocksDB.

Reasons include:

* Embedded deployment
* High write throughput
* Ordered key/value storage
* Atomic write batches
* Column families
* Snapshots
* Incremental compaction
* Excellent Rust ecosystem support
* No external database server required

Implementations MAY use alternative storage engines provided protocol behavior remains identical.

---

# 6. Persistent State Categories

The following categories SHALL be persisted.

### Network

* Peer metadata
* Known services
* Synchronization checkpoints
* Session metadata

### Marketplace

* Advertisements
* Reservations
* Liquidity vaults
* Settlements
* Disputes

### Identity

* Identity claims
* Verification records

### Reputation

* Reputation scores
* Historical updates

### Governance

* Proposals
* Votes
* Treasury records

### Oracle

* Latest oracle values
* Provider metadata

### Risk

* Risk intelligence
* Wallet screening records

### Notifications

* Delivery state
* Subscription records

### Configuration

* Node configuration
* Feature flags
* Version metadata

---

# 7. Logical Storage Model

The protocol organizes persistent state into logical collections.

Example:

```text
State
│
├── Network
├── Marketplace
├── Identity
├── Reputation
├── Governance
├── Oracle
├── Risk
├── Notifications
├── Metadata
└── Audit
```

The physical implementation is left to the storage engine.

---

# 8. Object Identity

Every persisted object SHALL possess a globally unique identifier.

Objects SHOULD include:

* Object ID
* Object Type
* Version
* Creation Timestamp
* Last Modified Timestamp

Object identifiers MUST remain immutable.

---

# 9. Object Versioning

Updates create new object versions.

Each version SHALL contain:

* Version Number
* Previous Version Reference
* Update Timestamp
* State Hash

Version history enables deterministic replay and audit.

---

# 10. Atomic State Transitions

A protocol event MAY update multiple objects.

Example:

SettlementCompleted

Updates:

* Reservation
* Vault balance
* Merchant statistics
* Reputation
* Audit log

These updates MUST be committed atomically.

Partial writes MUST never become visible.

---

# 11. Indexes

Implementations SHOULD maintain indexes for frequently queried objects.

Examples include:

Marketplace

* Advertisement ID
* Merchant ID
* Currency Pair
* Payment Method
* Status

Settlement

* Settlement ID
* User
* Merchant
* Status

Identity

* Claim ID
* Subject

Reputation

* Node ID
* Merchant ID

Indexes are implementation-specific but MUST remain deterministic.

---

# 12. Audit Log

The audit log records immutable protocol events.

Examples:

* AdvertisementCreated
* ReservationOpened
* SettlementCompleted
* GovernanceVoteCast
* IdentityClaimPublished
* RiskRecordAdded

The audit log SHALL support replay for verification and recovery.

---

# 13. Snapshots

Snapshots capture the complete protocol state at a specific synchronization point.

Snapshots SHALL include:

* State Hash
* Snapshot Version
* Timestamp
* Object Counts
* Checkpoint Reference

Snapshot transfer is specified by OFS-1300.

---

# 14. State Integrity

Nodes SHALL continuously verify storage integrity.

Verification MAY include:

* Object hashes
* Snapshot hashes
* Merkle-style state digests (future)
* Record counts
* Version consistency

Integrity failures SHALL trigger recovery.

---

# 15. Crash Recovery

Nodes MUST recover safely after:

* Power failure
* Process termination
* Disk interruption
* Operating system crash

Recovery SHALL restore the last successfully committed state.

Incomplete writes MUST be discarded.

---

# 16. Pruning

Historical protocol data may become unnecessary for normal operation.

Implementations MAY prune:

* Expired advertisements
* Completed reservations
* Obsolete notifications
* Old synchronization metadata

Pruning MUST NOT affect protocol correctness or audit integrity.

Governance MAY define minimum retention periods.

---

# 17. Backup and Restore

Nodes SHOULD support offline backups.

Restoration SHALL preserve:

* State consistency
* Version metadata
* Snapshot information
* Audit history

Backups SHOULD be verifiable before restoration.

---

# 18. Storage Performance

Implementations SHOULD optimize:

* Sequential writes
* Batched commits
* Read caching
* Prefix scans
* Compression
* Background compaction

Performance optimizations MUST NOT alter deterministic behavior.

---

# 19. Storage Migration

Future protocol versions may evolve the storage schema.

Implementations SHALL support:

* Schema versioning
* Automatic migrations
* Rollback protection
* Integrity validation after migration

Migration MUST preserve protocol semantics.

---

# 20. Security Considerations

Persistent storage MUST protect against:

* Corruption
* Unauthorized modification
* Replay of stale state
* Incomplete writes
* Version rollback
* Malicious local tampering

Nodes SHOULD verify critical state during startup.

Future protocol versions MAY define encrypted local storage profiles.

---

# 21. Conformance

A compliant implementation MUST:

* Persist all required protocol objects.
* Support atomic state transitions.
* Preserve deterministic state.
* Support snapshots.
* Maintain audit history.
* Recover safely after crashes.
* Verify storage integrity.
* Support schema versioning.

---

# 22. Relationship to Other Specifications

The Persistent State Storage Specification underpins every OpenFiat protocol.

```text
             Protocol Events
                    │
                    ▼
        Deterministic State Machine
                    │
                    ▼
      OFS-1900 Persistent Storage
                    │
     ┌──────────────┼──────────────┐
     ▼              ▼              ▼
 Active State   Audit History   Snapshots
     │              │              │
     └──────────────┼──────────────┘
                    ▼
          Node Synchronization
```

---

# 23. Future Extensions

Future versions may introduce:

* Content-addressable storage
* Merkleized state verification
* Incremental snapshots
* Differential backups
* Transparent compression
* Zero-copy object serialization
* Tiered storage
* Distributed archival storage
* Hardware-accelerated integrity verification

---

# 24. Summary

The OpenFiat Persistent State Storage Specification defines the canonical persistent data model required by every OpenFiat node.

By standardizing logical state organization, object lifecycle, indexing, snapshots, audit history, and integrity guarantees—while remaining independent of any particular database implementation—it ensures that every compliant node can store, recover, synchronize, and verify protocol state consistently.

The reference implementation adopts RocksDB as the recommended storage engine, but interoperability is achieved through adherence to this specification rather than dependence on a specific database technology.

The Persistent State Storage Specification answers one fundamental question:

**"How does an OpenFiat node persist deterministic marketplace state in a manner that is efficient, recoverable, verifiable, and interoperable across implementations?"**
