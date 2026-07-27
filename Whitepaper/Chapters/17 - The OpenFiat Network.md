# Chapter 17 — The OpenFiat Network

## 17.1 Introduction

OpenFiat is not a blockchain.

Instead, it is a decentralized peer-to-peer protocol built on top of the Solana blockchain.

Solana provides:

* Consensus.
* Smart contract execution.
* Asset custody.
* Escrow enforcement.
* Final settlement.

OpenFiat provides:

* Merchant discovery.
* Advertisement propagation.
* Order coordination.
* Reputation distribution.
* Marketplace synchronization.
* Peer discovery.
* Notification routing.
* Infrastructure services.

Rather than storing every marketplace event directly on Solana, OpenFiat distributes operational state through a decentralized peer-to-peer network while relying upon Solana for cryptographic security and settlement.

This architecture combines the scalability of peer-to-peer networking with the security guarantees of one of the world's highest-performance blockchains.

---

# 17.2 Design Objectives

The OpenFiat Network was designed around several principles.

### Decentralized

No single server should be required for normal protocol operation.

### Fast

Marketplace events should propagate across the network within seconds.

### Fault Tolerant

Network failures should not prevent protocol operation.

### Efficient

Only critical financial state is committed to Solana.

### Open

Anyone meeting the protocol requirements may operate a node.

### Compatible

The protocol should support implementations written in multiple programming languages.

---

# 17.3 Network Architecture

Every OpenFiat node participates equally within the peer-to-peer network.

```text id="network01"
                Solana Blockchain
                       │
             OpenFiat Programs
                       │
        ┌──────────────┼──────────────┐
        │              │              │
     Node A         Node B         Node C
        │              │              │
        └────── libp2p Gossip ───────┘
                       │
        Advertisements
        Trade Sessions
        Reputation
        Discovery
        Metadata
                       │
      ┌───────────────┼────────────────┐
      │               │                │
   Web Client     Mobile Client    Desktop Client
```

Applications communicate with nearby OpenFiat nodes rather than depending upon centralized infrastructure.

---

# 17.4 Why libp2p?

OpenFiat adopts **libp2p** as its networking foundation.

libp2p is a mature peer-to-peer networking framework supporting multiple programming languages, including Rust and JavaScript.

Advantages include:

* Proven peer discovery.
* Efficient gossip protocols.
* Encrypted communication.
* Cross-platform compatibility.
* Large open-source ecosystem.
* Extensible transport architecture.

Because libp2p is widely adopted, developers can build compatible implementations without reinventing networking primitives.

---

# 17.5 Gossip Protocol

Most marketplace information propagates using gossip.

Examples include:

* Merchant advertisements.
* Advertisement updates.
* Advertisement removals.
* Reputation updates.
* Risk signals.
* Node announcements.
* Snapshot availability.
* Notification provider announcements.
* Oracle availability.

Each node forwards newly received information to connected peers.

Over time, information naturally spreads throughout the network.

This model is similar to Solana's own gossip protocol and provides efficient decentralized dissemination without centralized coordination.

---

# 17.6 Peer Discovery

New nodes must discover existing peers before joining the network.

OpenFiat follows a bootstrap model similar to Solana.

Initially, a new node connects to one or more well-known bootstrap nodes.

Example bootstrap addresses:

* entry01.openfiat.network
* entry02.openfiat.network
* openfiat.allenhark.com

These nodes serve only as initial contact points.

After connecting, the joining node learns about additional peers through the gossip protocol.

Normal operation does not depend upon continued availability of any particular bootstrap node.

Governance may approve additional community-operated bootstrap nodes over time.

---

# 17.7 Bootstrap Philosophy

Bootstrap nodes are not authorities.

They do not:

* Validate trades.
* Control advertisements.
* Decide disputes.
* Maintain exclusive databases.
* Authorize participants.

Their sole responsibility is introducing new participants to the network.

Once peer discovery has completed, nodes communicate directly with one another.

If AllenHark's bootstrap infrastructure becomes unavailable, the network continues operating provided alternative bootstrap nodes remain accessible.

---

# 17.8 Node Identity

Each node possesses a unique cryptographic identity.

The node identity is separate from:

* Merchant identity.
* Wallet identity.
* Governance identity.

This allows infrastructure operators to manage nodes independently from marketplace participation.

Node identities are used for:

* Peer authentication.
* Reputation tracking.
* Performance measurement.
* Network routing.

---

# 17.9 RocksDB Storage

Every OpenFiat node embeds **RocksDB** as its primary local storage engine.

RocksDB stores protocol state that does not require on-chain persistence.

Examples include:

* Advertisement index.
* Active trade sessions.
* Peer information.
* Reputation cache.
* Risk cache.
* Notification provider directory.
* Snapshot metadata.
* Local synchronization state.

Using RocksDB enables fast local queries while minimizing repeated network requests.

Applications can therefore respond quickly even during periods of high marketplace activity.

---

# 17.10 Marketplace State

OpenFiat distinguishes between **authoritative state** and **replicated state**.

### Authoritative State

Stored on Solana.

Examples include:

* Escrow vaults.
* Stablecoin balances.
* Staking accounts.
* Governance votes.
* Treasury balances.

### Replicated State

Distributed through the OpenFiat network.

Examples include:

* Advertisements.
* Merchant availability.
* Risk vectors.
* Reputation vectors.
* Notification providers.
* Peer directory.
* Marketplace metadata.

This separation allows the protocol to scale efficiently while preserving cryptographic security for financial operations.

---

# 17.11 Snapshot Synchronization

Synchronizing a new node entirely from gossip may require considerable time.

To accelerate synchronization, OpenFiat supports downloadable RocksDB snapshots.

The synchronization process follows this sequence.

```text id="snapshot01"
New Node

↓

Download Snapshot

↓

Verify Integrity

↓

Restore Database

↓

Connect to Network

↓

Replay Recent Gossip

↓

Fully Synchronized
```

Snapshots dramatically reduce the time required for new infrastructure providers to become operational.

---

# 17.12 Snapshot Providers

Snapshot hosting is itself a protocol service.

Any qualified participant may become a snapshot provider.

Responsibilities include:

* Publishing current snapshots.
* Maintaining availability.
* Providing integrity metadata.
* Supporting high-bandwidth downloads.

Snapshot providers receive protocol incentives according to governance-approved reward formulas.

AllenHark will initially operate official snapshot servers during the bootstrap phase.

---

# 17.13 SWQoS-Inspired Peer Scoring

OpenFiat adopts a networking philosophy inspired by Solana's **Stake-Weighted Quality of Service (SWQoS)**.

Rather than treating every node equally under all conditions, peers are evaluated using objective performance metrics.

Examples include:

* Connection stability.
* Message propagation speed.
* Synchronization quality.
* Historical uptime.
* Successful protocol participation.
* Peer responsiveness.

Higher-quality peers receive greater priority when exchanging marketplace information.

Unlike Solana consensus, OpenFiat uses these scores only to improve network efficiency—they do not determine protocol authority.

---

# 17.14 Node Reputation

The network continuously evaluates infrastructure quality.

Examples include:

* Uptime.
* Successful synchronization.
* Gossip propagation.
* Snapshot integrity.
* Protocol compatibility.
* Peer reliability.

Node reputation affects reward eligibility but never grants special protocol privileges.

---

# 17.15 Version Compatibility

Every node advertises its supported protocol version.

This enables:

* Safe rolling upgrades.
* Backward compatibility where possible.
* Identification of outdated implementations.

Governance determines protocol deprecation schedules, ensuring upgrades occur in an orderly and transparent manner.

---

# 17.16 Failure Recovery

OpenFiat is designed to tolerate infrastructure failures.

Examples include:

* Node crashes.
* Temporary network partitions.
* Snapshot provider outages.
* Bootstrap node failures.
* Internet routing disruptions.

Because marketplace information is replicated across many independent peers, no single infrastructure provider represents a single point of failure.

Applications automatically reconnect to alternative peers whenever possible.

---

# 17.17 Network Security

All communication between peers is authenticated and encrypted.

Nodes verify the identity of connected peers before exchanging protocol messages.

Additional protections include:

* Message integrity verification.
* Replay protection.
* Peer reputation.
* Rate limiting.
* Resource quotas.
* Governance-controlled protocol upgrades.

These measures help maintain network stability while preserving decentralization.

---

# 17.18 Open Source by Design

Every component of the OpenFiat networking layer is intended to be open source.

Developers may:

* Build independent node implementations.
* Create custom client applications.
* Operate private infrastructure.
* Audit networking behavior.
* Extend protocol tooling.

Open specifications encourage interoperability while reducing dependence on any single implementation.

---

# 17.19 Why This Architecture Matters

Many peer-to-peer marketplaces rely on centralized APIs even when settlement occurs on-chain.

OpenFiat instead decentralizes the marketplace itself.

Advertisement discovery, merchant availability, reputation, risk signals, and infrastructure services are distributed across a global peer-to-peer network.

Only the financial settlement layer depends on Solana.

This separation provides scalability without compromising the security guarantees of on-chain escrow.

---

# 17.20 Looking Ahead

A decentralized network is only as strong as the participants who maintain it.

The next chapter examines the responsibilities, incentives, hardware requirements, reward mechanisms, and operational expectations for OpenFiat node operators, who form the backbone of the protocol's decentralized infrastructure.
