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
* Snapshot Height
* Creation Timestamp
* Snapshot Identifier
* State Root
* Snapshot Size
* Compression Method
* Producing Node
* Digital Signature

Metadata allows clients to evaluate and verify snapshots before downloading them.

---

## 9. Snapshot Height

Every snapshot is assigned a monotonically increasing Snapshot Height.

Example:

```text id="snapshot-height"
Snapshot 4,215

↓

Snapshot 4,216

↓

Snapshot 4,217
```

Snapshot Height enables nodes to determine whether newer snapshots are available.

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

* Latest Snapshot Height
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
