# OFS-1300 — Snapshot Synchronization Protocol (SSP)

**Document ID:** OFS-1300

**Title:** Snapshot Synchronization Protocol

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Network

**Depends On:** OFS-1000, OFS-1100, OFS-1200

---

## Abstract

The OpenFiat Snapshot Synchronization Protocol (SSP) defines how nodes rapidly synchronize marketplace state after joining the network or recovering from extended downtime.

Rather than replaying millions of historical events, a node downloads a recent, cryptographically verifiable snapshot of the current marketplace state and then resumes normal synchronization through the Gossip Protocol.

The protocol is inspired by large-scale distributed systems, including Solana's snapshot architecture, while remaining independent of blockchain ledger synchronization.

Snapshot Synchronization dramatically reduces bootstrap time, bandwidth consumption, and recovery time while maintaining deterministic network state.

---

## 1. Introduction

The OpenFiat network continuously produces events.

Over time, replaying every advertisement, order update, governance event, reputation change, and service registration becomes increasingly inefficient.

Instead, nodes periodically produce **Snapshots**.

A snapshot represents the complete network state at a specific point in time.

New nodes begin from the latest trusted snapshot before replaying only the events that occurred afterward.

This approach enables synchronization in minutes rather than hours or days.

---

## 2. Scope

This specification defines:

* Snapshot creation
* Snapshot format
* Snapshot providers
* Snapshot verification
* Snapshot distribution
* Snapshot selection
* Incremental synchronization
* Snapshot recovery

This specification does not define:

* Gossip propagation
* Peer discovery
* RocksDB implementation
* Marketplace logic

---

## 3. Design Goals

Snapshot Synchronization SHALL:

* Minimize bootstrap time.
* Minimize bandwidth usage.
* Preserve deterministic state.
* Allow multiple independent providers.
* Prevent malicious snapshot distribution.
* Recover rapidly after outages.

---

## 4. Snapshot Philosophy

Snapshots are **performance optimizations**.

They are **not** a source of truth.

The canonical state of OpenFiat remains the deterministic result of:

* Protocol rules
* Valid events
* Smart contract state
* Cryptographic verification

A snapshot simply represents that state at a particular instant.

---

## 5. Snapshot Providers

Any node meeting protocol requirements MAY become a Snapshot Provider.

Providers advertise this capability through the Service Registry (OFS-1500).

Snapshot providers may include:

* AllenHark
* Community node operators
* Enterprises
* Universities
* Exchanges
* Infrastructure companies

No provider possesses special authority.

Clients choose providers independently.

---

## 6. AllenHark Bootstrap Snapshots

During the network bootstrap phase, AllenHark intends to publish **minute-level snapshots** of the marketplace.

These frequent snapshots allow newly started nodes to synchronize extremely quickly without replaying large event histories.

AllenHark snapshots are provided as a convenience to accelerate ecosystem growth.

They are **not mandatory**, and nodes remain free to download snapshots from any compatible provider.

As the network matures, additional community-operated snapshot providers are expected to reduce reliance on any single organization.

---

## 7. Snapshot Contents

A snapshot represents the complete network state required to resume operation.

A snapshot MAY contain:

* Active advertisements
* Active reservations
* Pending settlements
* Merchant profiles
* Reputation database
* Service registry
* Governance state
* Oracle cache
* Risk intelligence cache
* Peer metadata
* Protocol configuration
* Synchronization checkpoint

Expired and historical data SHOULD NOT be included unless required for protocol correctness.

---

## 8. Snapshot Metadata

Every snapshot SHALL contain metadata describing itself.

Required metadata includes:

* Snapshot Version
* OFS Version
* Snapshot Slot
* Creation Timestamp
* Snapshot Identifier
* State Root
* Snapshot Size
* Compression Method
* Producing Node
* Digital Signature

Metadata allows clients to evaluate and verify snapshots before downloading them.

---

## 9. Snapshot Slot

`[PROPOSED — NEEDS SIGN-OFF]`

Every snapshot SHALL record the **Solana slot its state is current as of**,
and a node MUST NOT produce a snapshot until it has observed one.

### 9.1 Why the ordering key is borrowed rather than invented

An earlier revision of this section required "a monotonically increasing
Snapshot Height" without saying what a height counts. An implementation
resolved that as the producing node's own local count of gossip events,
and that cannot do the job the field is asked to do.

**A per-producer counter is not comparable across producers.** Two nodes
holding identical state report different numbers if they have observed
different numbers of events; a node that joined last week reports a lower
number than one running since genesis, with the same state. So ordering two
producers' snapshots by it orders nothing, and a receiving node comparing
an incoming number against its own — which is what an anti-rollback check
must do — is comparing two different quantities.

The protocol has no consensus over off-chain state, so it cannot mint a
global sequence of its own. It does not need to: participants already share
one. The Solana slot is agreed by everyone, monotonic, and produced by a
consensus this protocol already depends on for settlement (OFS-4300).

A borrowed clock also gains a property a self-reported counter can never
have: **a claimed slot is checkable.** A node MAY compare an announced slot
against its own view of the chain and refuse one from an implausible
future. Nothing equivalent is possible against a number only the announcer
can see.

### 9.2 What a slot asserts, and what it does not

A Snapshot Slot says **when** the state was captured. It does not assert
**what** the snapshot contains.

Two nodes snapshotting at the same slot MAY hold different off-chain state,
because gossip propagation is not instantaneous. A slot is a recency
anchor, not a proof of containment — the same thing a Solana snapshot means
by it. Establishing containment would require consensus over off-chain
state, which this protocol deliberately does not have, and implementations
MUST NOT present slot ordering as such a proof.

### 9.3 A node without a slot does not produce

A node that has never observed a slot MUST NOT produce a snapshot. It
cannot say when its state is current as of, and a fabricated value is worse
than no snapshot at all: every peer orders candidates by that number, so an
invented one either buries an honest producer or promotes itself above one.

This is not a requirement to hold an RPC connection. A `GossipOnly` node
learns slots over the Chain Bridge (OFS-4300), so any node connected to the
network at all has one shortly after its first peer.

---

## 10. State Root

Every snapshot SHALL include a deterministic State Root.

The State Root is a cryptographic digest representing the complete snapshot state.

Nodes MUST verify the State Root after importing a snapshot.

Any mismatch SHALL invalidate the snapshot.

---

## 11. Snapshot Generation

Snapshot generation occurs periodically.

Typical process:

```text id="snapshot-create"
Freeze State

↓

Flush Database

↓

Compute State Root

↓

Generate Metadata

↓

Compress

↓

Digitally Sign

↓

Publish
```

Snapshot generation MUST NOT interrupt normal marketplace operation.

---

## 12. Snapshot Publication

Snapshot providers advertise newly available snapshots through the Gossip Protocol.

Only metadata is gossiped.

Actual snapshot files are transferred separately.

This minimizes network bandwidth.

---

## 13. Snapshot Discovery

Nodes searching for snapshots query the Service Registry.

Providers respond with:

* Latest Snapshot Slot
* Snapshot Identifier
* Supported OFS Version
* Download Endpoints
* Compression Formats

Clients choose which provider to use.

---

## 14. Snapshot Download

Snapshots MAY be downloaded using one or more transport mechanisms.

Examples include:

* libp2p stream transfer
* HTTPS
* HTTP/3
* Peer-to-peer segmented transfer

The transport mechanism does not affect snapshot validity.

Verification always occurs after download.

---

## 15. Compression

Snapshots SHOULD be compressed.

Supported compression algorithms are negotiated independently.

Compression reduces:

* Storage
* Bandwidth
* Bootstrap time

The compression method MUST be recorded within snapshot metadata.

---

## 16. Snapshot Verification

After download, every snapshot MUST be verified.

Verification includes:

* Digital signature
* Snapshot version
* Protocol compatibility
* Compression integrity
* Metadata validity
* State Root verification

Only verified snapshots MAY be imported.

---

## 17. Snapshot Import

Import process:

```text id="snapshot-import"
Download

↓

Verify

↓

Decompress

↓

Load Into RocksDB

↓

Verify State Root

↓

Activate Snapshot

↓

Resume Gossip
```

Nodes SHALL NOT participate in the network until import completes successfully.

---

## 18. Incremental Synchronization

A snapshot is only current until new events occur.

Immediately after activation:

```text id="incremental-sync"
Snapshot Imported

↓

Request Missing Events

↓

Replay Missing Events

↓

Current Network State
```

Missing events are obtained through the Gossip Protocol.

---

## 19. Snapshot Retention

Snapshot providers SHOULD retain multiple historical snapshots.

Benefits include:

* Rollback
* Version compatibility
* Recovery
* Long-distance synchronization

Retention policies remain implementation specific.

---

## 20. Snapshot Replacement

Nodes periodically evaluate available snapshots.

If a significantly newer snapshot exists, implementations MAY recommend resynchronization.

Routine operation SHOULD continue using incremental synchronization.

---

## 21. Partial Synchronization

Future protocol versions MAY introduce partial snapshots.

Examples include:

* Governance only
* Reputation only
* Merchant advertisements only
* Oracle cache only

Partial snapshots are optional extensions.

---

## 22. Failure Recovery

If snapshot import fails:

```text id="snapshot-failure"
Verification Failure

↓

Discard Snapshot

↓

Select Alternate Provider

↓

Retry Download
```

Nodes MUST NOT activate partially imported snapshots.

---

## 23. Multiple Providers

Nodes SHOULD support multiple snapshot providers.

Selection factors MAY include:

* Latency
* Download speed
* Availability
* Reputation
* Geographic proximity
* Infrastructure diversity

Provider choice remains entirely local.

---

## 24. Security Considerations

Implementations MUST protect against:

* Corrupted snapshots
* Malicious providers
* Replay attacks
* Downgrade attacks
* Version mismatches
* Truncated downloads
* Signature forgery

Cryptographic verification is mandatory regardless of provider trust.

---

## 25. Performance Considerations

Snapshot Synchronization is expected to reduce synchronization time from hours of event replay to minutes.

Providers SHOULD optimize:

* Compression ratios
* Parallel downloads
* Incremental publication
* Efficient RocksDB serialization

The protocol is designed to support very large marketplace datasets while maintaining fast recovery.

---

## 26. Conformance

A compliant implementation MUST:

* Support snapshot discovery.
* Verify digital signatures.
* Verify State Roots.
* Validate metadata.
* Reject incompatible snapshots.
* Resume Gossip synchronization after import.
* Support multiple providers.
* Maintain deterministic imported state.

---

## 27. Relationship to Other Specifications

Snapshot Synchronization accelerates network recovery without replacing normal protocol operation.

```text id="snapshot-architecture"
             OFS-1000
        Network Protocol
                  │
                  ▼
             OFS-1100
         Peer Discovery
                  │
                  ▼
             OFS-1300
     Snapshot Synchronization
                  │
                  ▼
         Local RocksDB State
                  │
                  ▼
             OFS-1200
        Gossip Synchronization
                  │
                  ▼
        Current Marketplace State
```

Snapshot Synchronization answers a single question:

**"How can a node reach the current network state as quickly and efficiently as possible?"**

By combining cryptographically verified snapshots with incremental event replay, OpenFiat enables rapid bootstrap, fast disaster recovery, and scalable operation while preserving the decentralized and deterministic nature of the network.
