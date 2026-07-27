# Chapter 4 — System Architecture

## 4.1 Introduction

The previous chapters explained why OpenFiat exists and the principles that guide its development.

This chapter answers a different question:

> **How does OpenFiat actually work?**

Understanding OpenFiat begins with understanding one important architectural decision:

**OpenFiat is not a blockchain.**

Instead, OpenFiat is a decentralized protocol built on top of an existing blockchain.

This distinction affects every aspect of the system.

Solana remains responsible for securing digital assets, executing smart contracts, and achieving network consensus.

OpenFiat provides everything required to transform those blockchain capabilities into a decentralized peer-to-peer marketplace.

Rather than attempting to replace Solana, OpenFiat extends it.

The result is a layered architecture where each layer performs the tasks it is best suited to perform.

---

# 4.2 The Layered Architecture

OpenFiat separates responsibilities into three primary layers.

```text
                    Applications
──────────────────────────────────────────
 Web UI • Mobile App • Desktop App • SDKs

                 OpenFiat Protocol
──────────────────────────────────────────
 OFNP • OFTP • Reputation • Governance
 Service Registry • Notifications
 Discovery • Advertisements • Sessions

                 Solana Blockchain
──────────────────────────────────────────
 Escrow • Treasury • Staking
 Governance Execution
 Token Program
```

Each layer has a clearly defined responsibility.

The Application Layer provides interfaces for users.

The OpenFiat Protocol coordinates participants.

The Solana Layer provides immutable settlement and security.

No layer duplicates the responsibilities of another.

This separation keeps the protocol efficient, modular, and easier to maintain.

---

# 4.3 Why Build on Solana?

One of the earliest design decisions was whether OpenFiat should introduce its own blockchain.

After extensive evaluation, the answer was no.

Creating a new blockchain would require solving problems that have already been solved extremely well by Solana, including:

* Global consensus
* Transaction ordering
* Block production
* Validator incentives
* Cryptographic security
* Network synchronization
* Final settlement

Rebuilding these components would dramatically increase complexity without providing additional value to OpenFiat users.

Instead, OpenFiat focuses exclusively on decentralized marketplace coordination while relying on Solana for blockchain security.

This allows OpenFiat developers to spend their time improving the marketplace rather than maintaining blockchain infrastructure.

---

# 4.4 The Major Components

The OpenFiat ecosystem consists of several independent components working together.

These components communicate through open standards rather than proprietary interfaces.

The primary components include:

* Users
* Wallets
* Smart Contracts
* OpenFiat Nodes
* Service Providers
* Client Applications
* Governance
* Treasury

Each component is described below.

---

# 4.5 Users

Users are the participants who ultimately benefit from the protocol.

OpenFiat recognizes several participant roles.

These include:

* Buyers
* Merchants
* Arbitrators
* Node Operators
* Service Providers
* Developers
* Governance Participants

Importantly, these are **roles**, not account types.

A single wallet may perform multiple roles simultaneously.

For example, a merchant may also:

* Operate a node
* Participate in governance
* Serve as an arbitrator
* Provide notification services

The protocol does not restrict these combinations.

---

# 4.6 Wallets

Every interaction with OpenFiat begins with a cryptocurrency wallet.

The wallet acts as the participant's cryptographic identity.

Rather than creating usernames and passwords, users authenticate by signing messages using their wallet.

The wallet is responsible for:

* Holding OPEN tokens
* Holding supported stablecoins
* Signing protocol messages
* Funding escrow
* Receiving rewards
* Participating in governance

The wallet never shares its private keys with OpenFiat.

Private keys remain under the user's exclusive control.

---

# 4.7 Smart Contracts

Smart contracts represent the trust layer of OpenFiat.

They execute deterministic rules that cannot be altered during individual trades.

The primary smart contracts include:

* Escrow Program
* Treasury Program
* Governance Program
* Registry Program
* Staking Program

These contracts are responsible only for operations that require blockchain-level security.

For example:

Escrow custody belongs on-chain.

Advertisement discovery does not.

This distinction significantly reduces transaction costs while preserving security.

---

# 4.8 OpenFiat Nodes

OpenFiat nodes form the decentralized coordination network.

Unlike Solana validators, OpenFiat nodes do not produce blocks.

Instead, they coordinate marketplace activity.

Their responsibilities include:

* Advertisement propagation
* Merchant discovery
* Session synchronization
* Reputation indexing
* Gossip networking
* Order propagation
* Snapshot generation
* State synchronization

Nodes communicate using libp2p.

Local state is maintained using RocksDB.

Snapshots allow new nodes to synchronize quickly without rebuilding the entire network history.

---

# 4.9 Service Providers

Certain protocol functions benefit from specialization.

Instead of requiring every node to perform every service, OpenFiat introduces specialized service providers.

Examples include:

Notification Providers

Responsible for delivering:

* Email notifications
* Telegram notifications
* Discord notifications
* Future messaging integrations

Oracle Providers

Responsible for publishing reference exchange rates used by floating-price advertisements.

Snapshot Providers

Responsible for distributing verified RocksDB snapshots that accelerate node synchronization.

Future versions of the protocol may introduce additional provider categories while preserving compatibility with existing implementations.

All providers register through the protocol registry and satisfy staking requirements appropriate to their service category.

---

# 4.10 Client Applications

Client applications provide the user interface.

Examples include:

* Web applications
* Mobile applications
* Desktop applications
* Command-line interfaces
* Merchant dashboards
* Trading bots

Applications are interchangeable.

Users may freely switch between compatible clients without creating new accounts or migrating data.

This is possible because user identity resides in the wallet rather than the application.

Applications communicate with OpenFiat nodes using the public protocol rather than proprietary APIs.

---

# 4.11 OpenFiat Network (OFNP)

OpenFiat nodes communicate through the OpenFiat Network Protocol (OFNP).

OFNP is built upon libp2p and adopts many networking principles proven within the Solana ecosystem.

Its responsibilities include:

* Peer discovery
* Gossip propagation
* Session synchronization
* Advertisement distribution
* Reputation updates
* Service discovery
* Snapshot exchange

Bootstrap nodes assist new participants in locating peers but do not become permanent points of dependency.

Once connected, nodes communicate directly with one another.

---

# 4.12 Trade Protocol (OFTP)

Trading within OpenFiat follows the OpenFiat Trade Protocol (OFTP).

OFTP defines every stage of a trade, including:

* Advertisement selection
* Reservation
* Escrow funding
* Payment confirmation
* Evidence submission
* Dispute creation
* Arbitration
* Settlement

Every compliant application follows the same protocol.

This guarantees interoperability across independently developed software.

---

# 4.13 Trust Boundaries

One of the most important concepts within OpenFiat is understanding which components require trust.

The protocol deliberately minimizes trust wherever possible.

```text
User Wallet
      │
      │ Cryptographic Signatures
      ▼
OpenFiat Protocol
      │
      │ Smart Contracts
      ▼
Solana Blockchain
```

Users never trust individual nodes to safeguard funds.

Nodes never control escrow.

Applications never control wallets.

Service providers never control governance.

Each component performs only the responsibilities assigned to it.

---

# 4.14 Network Topology

OpenFiat operates as a decentralized mesh network.

```text
                Bootstrap Nodes
           entry01.openfiat.network
         entry02.openfiat.network
      openfiat.allenhark.com
      openfiat.allenhark.network

                   │
        ───────────┼───────────
                   │
        libp2p Discovery Layer
                   │
      ┌────────────┼────────────┐
      │            │            │
   Node A       Node B       Node C
      │            │            │
      ├────── Gossip Network ───┤
      │            │            │
  Merchant      Wallet      Mobile App
```

Bootstrap nodes help new participants discover the network.

After discovery, nodes communicate directly without routing traffic through AllenHark infrastructure.

This architecture eliminates single points of failure while simplifying initial network connectivity.

---

# 4.15 Data Flow

A simplified trade demonstrates how the architecture operates.

1. A merchant publishes an advertisement.

2. The advertisement propagates across the OpenFiat node network.

3. A buyer discovers the advertisement through any compatible client.

4. The buyer reserves the trade.

5. Escrow is funded on Solana.

6. Off-chain communication coordinates fiat payment.

7. Settlement instructions are submitted to the escrow program.

8. Funds are released.

Only blockchain operations require Solana transactions.

Marketplace coordination occurs efficiently through the OpenFiat network.

---

# 4.16 Security Model

OpenFiat deliberately separates security into two independent layers.

Blockchain Security

Provided by Solana.

Protects:

* Escrow
* Tokens
* Treasury
* Governance execution

Protocol Security

Provided by OpenFiat.

Protects:

* Reputation
* Gossip
* Discovery
* Session integrity
* Advertisement authenticity
* Provider authentication

Separating these responsibilities improves scalability while reducing unnecessary blockchain transactions.

---

# 4.17 Failure Tolerance

The protocol is designed to continue operating despite partial failures.

Examples include:

If a node disconnects:

Other nodes continue propagating advertisements.

If a notification provider becomes unavailable:

Users may select another registered provider.

If AllenHark shuts down its bootstrap infrastructure:

Community-operated bootstrap nodes continue serving the network.

If one client application disappears:

Users migrate to another compatible application.

This resilience results directly from the protocol's decentralized architecture.

---

# 4.18 Progressive Infrastructure Decentralization

OpenFiat begins with reference infrastructure operated by AllenHark.

This includes:

* Bootstrap nodes
* Snapshot distribution
* Notification services
* Oracle services
* Reference user interface

These components are intended to accelerate early adoption rather than establish permanent control.

As the ecosystem grows, community-operated providers gradually replace or supplement AllenHark-operated infrastructure.

Success is measured by decreasing dependence upon any single operator.

---

# 4.19 Why This Architecture Matters

The architecture presented in this chapter reflects a deliberate engineering philosophy.

Rather than placing every operation on-chain, OpenFiat carefully identifies which functions require blockchain security and which benefit from decentralized off-chain coordination.

This balance provides several advantages:

* Lower transaction costs.
* Faster user experience.
* Reduced blockchain congestion.
* Greater implementation flexibility.
* Easier scalability.
* Independent client implementations.
* Long-term protocol sustainability.

The result is a decentralized marketplace that remains secure without sacrificing usability.

---

# 4.20 Looking Ahead

The architecture described in this chapter provides a high-level view of the OpenFiat ecosystem.

The following chapters examine each major subsystem individually.

The next chapter introduces the OpenFiat Network Protocol (OFNP), explaining how nodes discover one another, exchange information, synchronize state, propagate advertisements, and maintain a resilient decentralized communication network.
