# Chapter 18 — Node Operators

## 18.1 Introduction

Node operators are the backbone of the OpenFiat network.

While Solana validators secure the blockchain and execute the OpenFiat smart contracts, OpenFiat node operators maintain the decentralized marketplace that exists above the blockchain.

Every advertisement published, reputation update distributed, trade session synchronized, notification provider announced, and marketplace discovery request flows through the OpenFiat peer-to-peer network.

Without node operators, the protocol would remain secure on-chain but the marketplace itself would become fragmented and difficult to use.

For this reason, OpenFiat incentivizes independent node operators around the world to maintain highly available infrastructure.

The objective is to create a globally distributed marketplace that does not depend upon AllenHark or any other single organization.

---

## 18.2 Design Objectives

The Node Operator Program has several goals.

### Decentralization

Encourage infrastructure ownership by the global community.

### High Availability

Maintain continuous marketplace connectivity.

### Fast Propagation

Distribute marketplace events with minimal latency.

### Geographic Diversity

Reduce regional infrastructure concentration.

### Economic Sustainability

Reward operators according to measurable contributions.

### Open Participation

Any participant meeting protocol requirements may become a node operator.

---

## 18.3 Responsibilities

Every node participates in maintaining the decentralized marketplace.

Core responsibilities include:

* Maintaining peer-to-peer connections.
* Participating in gossip propagation.
* Synchronizing marketplace state.
* Hosting advertisement indexes.
* Distributing reputation vectors.
* Distributing risk vectors.
* Broadcasting trade lifecycle events.
* Maintaining the provider directory.
* Publishing node health information.
* Serving client applications.

Nodes never custody user funds.

All financial operations remain under the control of Solana smart contracts.

---

## 18.4 Node Lifecycle

A new node follows a deterministic initialization process.

```text
Install OpenFiat Node

↓

Generate Node Identity

↓

Configure Wallet

↓

Stake OPEN

↓

Register with Network

↓

Connect to Bootstrap Nodes

↓

Download Latest Snapshot

↓

Verify Snapshot Integrity

↓

Replay Recent Gossip

↓

Begin Serving Network
```

After synchronization completes, the node becomes a full participant in the OpenFiat network.

---

## 18.5 Node Identity

Each node possesses a unique cryptographic identity.

This identity is independent from:

* Merchant accounts.
* Governance accounts.
* Arbitrator identities.
* Wallet ownership.

Node identities are used for:

* Peer authentication.
* Reputation tracking.
* Reward calculation.
* Performance measurement.
* Secure communication.

Because node identities are cryptographically generated, no central authority assigns or approves them.

---

## 18.6 Node Registration

Nodes announce themselves to the network after satisfying minimum protocol requirements.

Registration includes metadata such as:

* Node public key.
* Supported protocol version.
* Geographic region (optional).
* Available services.
* Software version.
* Public network endpoints.

The announcement is propagated through the gossip network.

Nodes may update their metadata as necessary.

---

## 18.7 Services Provided

Every OpenFiat node provides a standard collection of services.

### Marketplace Discovery

Responding to advertisement searches.

### Advertisement Distribution

Broadcasting newly created advertisements.

### Reputation Distribution

Serving reputation vectors.

### Risk Distribution

Serving marketplace risk information.

### Session Coordination

Synchronizing signed trade session messages.

### Peer Discovery

Introducing newly joined nodes to additional peers.

### Directory Replication

Maintaining copies of protocol directories.

Additional services may be introduced by future protocol upgrades.

---

## 18.8 Peer Connectivity

Nodes maintain multiple simultaneous peer connections.

Rather than depending upon a single upstream provider, every node continuously exchanges information with numerous peers.

Benefits include:

* Faster propagation.
* Greater redundancy.
* Improved resilience.
* Better load distribution.
* Resistance to infrastructure failures.

Connection management is handled automatically by the reference implementation.

---

## 18.9 Client Connections

Applications do not communicate directly with every node.

Instead, clients connect to one or more nearby OpenFiat nodes.

These nodes provide:

* Advertisement queries.
* Merchant discovery.
* Reputation lookups.
* Risk information.
* Trade session synchronization.
* Notification provider discovery.

If one node becomes unavailable, clients may automatically connect to another compatible node.

This architecture eliminates single points of failure while improving application responsiveness.

---

## 18.10 Storage Responsibilities

Every node maintains a local RocksDB database.

Examples of stored information include:

* Marketplace advertisements.
* Reputation vectors.
* Risk vectors.
* Peer metadata.
* Notification provider directory.
* Snapshot metadata.
* Local synchronization state.
* Cached protocol information.

Authoritative financial data remains on Solana.

The RocksDB database stores replicated marketplace information necessary for efficient operation.

---

## 18.11 Snapshot Participation

Nodes may optionally become Snapshot Providers.

Responsibilities include:

* Publishing compressed RocksDB snapshots.
* Maintaining download availability.
* Publishing integrity hashes.
* Supporting rapid network synchronization.

Snapshot Providers receive additional protocol rewards when selected by governance-approved reward formulas.

AllenHark will initially operate official snapshot servers during the bootstrap phase while encouraging independent providers to join the ecosystem.

---

## 18.12 Performance Metrics

Node quality is continuously evaluated using objective measurements.

Examples include:

### Availability

Percentage of time the node remains online.

### Synchronization

How closely the node tracks current marketplace state.

### Propagation Speed

How rapidly marketplace events are forwarded.

### Peer Stability

Quality of maintained peer connections.

### Protocol Compatibility

Support for the latest protocol specification.

### Snapshot Reliability

Where applicable, the success rate of distributed snapshots.

These metrics contribute to Node Reputation and reward eligibility.

---

## 18.13 SWQoS-Inspired Networking

OpenFiat adopts principles inspired by Solana's Stake-Weighted Quality of Service (SWQoS).

Nodes demonstrating consistently strong performance receive higher communication priority from peers.

Performance factors include:

* Low latency.
* Reliable propagation.
* Stable connectivity.
* High uptime.
* Successful synchronization.

Unlike Solana consensus, these priorities improve network efficiency only.

They do not grant authority over protocol decisions or financial settlement.

---

## 18.14 Node Rewards

Node operators are compensated from protocol revenue rather than token inflation.

Reward calculations consider multiple factors, including:

* Active stake.
* Node reputation.
* Marketplace availability.
* Successful synchronization.
* Peer connectivity.
* Network contribution.
* Service quality.

Governance defines the precise weighting of these factors within the Tokenomics Specification.

This model encourages long-term investment in reliable infrastructure rather than simply operating the greatest number of nodes.

---

## 18.15 Misbehavior and Penalties

OpenFiat distinguishes between ordinary failures and malicious behavior.

Examples of ordinary failures include:

* Temporary power outages.
* Internet disruptions.
* Hardware replacement.
* Scheduled maintenance.

Such events primarily affect reputation and reward eligibility.

Examples of malicious behavior include:

* Intentionally withholding marketplace information.
* Publishing corrupted snapshots.
* Protocol manipulation.
* Deliberate spam propagation.
* Persistent protocol violations.

Where appropriate, governance-approved slashing rules may apply.

Penalties are always deterministic and publicly documented.

---

## 18.16 Hardware Recommendations

The OpenFiat reference node is designed to run on commodity server hardware.

Typical recommendations include:

Minimum:

* 4 CPU cores.
* 16 GB RAM.
* 250 GB NVMe SSD.
* Stable broadband connection.

Recommended:

* 8–16 CPU cores.
* 32 GB RAM or more.
* 1 TB NVMe SSD.
* High-bandwidth, low-latency internet connection.
* Uninterruptible Power Supply (UPS).

These recommendations will evolve as the network grows.

Unlike Solana validators, OpenFiat nodes do not require specialized high-end hardware, making participation accessible to a much broader range of operators.

---

## 18.17 AllenHark's Initial Role

During the network bootstrap phase, AllenHark will operate several public infrastructure services, including:

* Bootstrap nodes.
* Snapshot servers.
* Public discovery endpoints.
* Monitoring infrastructure.
* Developer tooling.

These services are intended to accelerate early network growth.

As independent operators join the ecosystem, the network becomes progressively decentralized, reducing reliance on AllenHark infrastructure.

---

## 18.18 Open Infrastructure

The OpenFiat network welcomes independent infrastructure providers.

Organizations may operate:

* Public nodes.
* Regional nodes.
* Enterprise infrastructure.
* Academic research nodes.
* Community-hosted nodes.
* Private organizational nodes.

Provided they follow the published protocol specification, all compatible implementations participate equally within the network.

---

## 18.19 Why Node Operators Matter

Node operators transform OpenFiat from a collection of smart contracts into a living global marketplace.

They distribute information, improve resilience, reduce latency, and ensure that merchants and traders around the world can discover one another without relying upon centralized infrastructure.

The health of the OpenFiat ecosystem therefore depends not only upon secure smart contracts but also upon a vibrant, geographically distributed community of independent infrastructure providers.

---

## 18.20 Looking Ahead

Not every participant wishes to operate infrastructure.

Many simply want timely updates when trades progress, payments are received, disputes begin, or escrow is released.

The next chapter introduces Notification Providers, an optional decentralized service layer that enables participants to receive real-time alerts through email, Telegram, Discord, and future communication platforms while preserving OpenFiat's decentralized architecture.
