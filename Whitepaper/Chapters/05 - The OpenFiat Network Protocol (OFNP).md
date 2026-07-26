# Chapter 5 — The OpenFiat Network Protocol (OFNP)

## 5.1 Introduction

Every decentralized system requires a method by which independent computers discover one another, exchange information, synchronize state, and remain resilient despite failures.

In Bitcoin, this responsibility belongs to the Bitcoin peer-to-peer protocol.

In Ethereum, it is performed by the Ethereum networking stack.

In Solana, validators communicate using a combination of Turbine, Gossip, QUIC, TPU, TVU, Repair, and other networking protocols designed specifically for high-performance blockchain operation.

OpenFiat faces a different challenge.

Unlike a blockchain, OpenFiat does not need to reach global consensus about every event.

Instead, it must efficiently distribute marketplace information while allowing participants to communicate directly without depending on centralized servers.

This responsibility belongs to the **OpenFiat Network Protocol (OFNP).**

OFNP defines how OpenFiat nodes discover peers, exchange advertisements, synchronize reputation data, locate service providers, distribute snapshots, recover from failures, and maintain a shared view of marketplace activity.

It is the communication backbone of the OpenFiat ecosystem.

---

# 5.2 Why OFNP Exists

A natural question arises:

> "Why not simply store everything on Solana?"

While technically possible, such an approach would be prohibitively expensive and unnecessary.

Consider a typical trade.

A merchant updates availability.

A buyer searches advertisements.

The buyer opens an order.

Both participants exchange encrypted messages.

Typing indicators appear.

Notification providers send alerts.

The merchant temporarily goes offline.

Another node forwards an updated advertisement.

None of these activities involve transferring digital assets.

Recording each event on-chain would dramatically increase transaction costs while providing little additional security.

Instead, OpenFiat separates communication from settlement.

Marketplace communication occurs through OFNP.

Financial settlement occurs on Solana.

This division of responsibility is one of the defining architectural decisions of the protocol.

---

# 5.3 Design Goals

The OpenFiat Network Protocol was designed around several primary objectives.

### Decentralization

No permanent central server should be required for the network to operate.

### Performance

Marketplace information should propagate within seconds across the global network.

### Fault Tolerance

Individual node failures should not interrupt marketplace operation.

### Scalability

The network should support millions of advertisements without centralized indexing.

### Open Participation

Any compatible implementation should be able to join the network.

### Extensibility

Future protocol versions should introduce new message types without breaking compatibility.

---

# 5.4 Building on Proven Technology

Rather than inventing an entirely new networking protocol, OpenFiat intentionally builds upon technologies that have already demonstrated reliability at internet scale.

The reference implementation uses **libp2p** as its networking foundation.

libp2p provides:

* Peer discovery
* Encrypted communication
* Multiplexed streams
* NAT traversal
* Transport abstraction
* Secure identity
* Publish-subscribe messaging
* Request-response protocols

By adopting libp2p, OpenFiat benefits from years of engineering work contributed by multiple blockchain ecosystems.

OpenFiat defines the application protocol that operates above libp2p rather than replacing it.

---

# 5.5 Network Topology

OpenFiat forms a decentralized mesh network.

```text
                 Bootstrap Nodes
           entry01.openfiat.org
           entry02.openfiat.org
       openfiat.allenhark.com
       openfiat.allenhark.network

                   │
           Initial Discovery
                   │
        ┌──────────────┴──────────────┐
        │                             │
   Community Node                Community Node
        │                             │
   Gossip • Sync                Gossip • Sync
        │                             │
        └──────────────┬──────────────┘
                       │
                Additional Nodes
```

Bootstrap nodes only assist with initial peer discovery.

Once connected, nodes communicate directly with one another.

The network continues functioning even if every AllenHark-operated bootstrap node disappears, provided community bootstrap nodes remain available.

---

# 5.6 Bootstrap Nodes

Every decentralized network faces the same initial challenge.

A brand new node knows nothing about the network.

It has no peers.

No advertisements.

No service providers.

No reputation database.

No routing information.

Bootstrap nodes solve this initial discovery problem.

A newly installed OpenFiat node contacts one or more well-known bootstrap addresses.

Examples include:

* entry01.openfiat.org
* entry02.openfiat.org
* bootstrap.openfiat.org

These nodes return a list of active peers currently participating in the network.

After obtaining peer information, the new node disconnects from the bootstrap server if desired and begins communicating directly with other participants.

Bootstrap nodes are therefore **directories**, not centralized coordinators.

---

# 5.7 Peer Discovery

Once connected, nodes continuously discover additional peers.

Discovery occurs through several mechanisms:

* Bootstrap directories.
* Peer exchange.
* Gossip announcements.
* Previously trusted peers.
* Snapshot metadata.

Each node maintains a dynamic routing table that evolves as the network changes.

Nodes periodically evaluate peer quality using measurable characteristics such as:

* Availability.
* Response latency.
* Successful synchronization.
* Historical reliability.
* Protocol compatibility.

Poor-quality peers gradually disappear from routing tables.

Reliable peers become preferred communication partners.

---

# 5.8 Gossip Protocol

Most OpenFiat information propagates using a gossip protocol.

Gossip resembles the spread of information through human conversation.

One participant informs several others.

Each recipient informs additional participants.

Information rapidly spreads throughout the network.

Examples of gossiped information include:

* New advertisements.
* Advertisement updates.
* Merchant availability.
* Service provider registration.
* Node announcements.
* Governance proposals.
* Snapshot availability.
* Reputation updates.
* Protocol upgrades.

Because each message is cryptographically signed, receiving nodes can independently verify authenticity before forwarding information.

---

# 5.9 Message Types

OFNP defines several categories of network messages.

Examples include:

Discovery Messages

Used to locate peers.

Advertisement Messages

Publish, modify, or remove merchant advertisements.

Trade Messages

Coordinate active trades.

Service Registry Messages

Advertise available notification providers, oracle providers, and snapshot providers.

Governance Messages

Distribute governance proposals and voting announcements.

Synchronization Messages

Assist new nodes in catching up with current network state.

Heartbeat Messages

Indicate continued node availability.

Future protocol versions may introduce additional message categories without modifying the underlying networking architecture.

---

# 5.10 Local State

Unlike blockchains, OpenFiat nodes maintain local databases containing frequently accessed marketplace information.

The reference implementation uses RocksDB.

Local state includes:

* Advertisements.
* Merchant profiles.
* Reputation indexes.
* Active service providers.
* Snapshot metadata.
* Governance metadata.
* Peer information.

Because this information can be reconstructed from network synchronization, it does not require blockchain storage.

This dramatically improves performance while reducing operating costs.

---

# 5.11 Snapshots

Synchronizing years of marketplace history from individual gossip messages would become increasingly inefficient.

To address this, OpenFiat introduces state snapshots.

A snapshot contains a verified copy of current marketplace state.

Examples include:

* Active advertisements.
* Merchant reputation.
* Provider registry.
* Governance metadata.
* Peer directory.

AllenHark initially operates trusted snapshot servers.

Community members may later register independent snapshot providers.

Snapshots are cryptographically verified before use.

They accelerate synchronization without introducing trusted intermediaries.

---

# 5.12 Session Synchronization

Trades often span several minutes or longer.

Participants may temporarily lose internet connectivity.

Applications may restart.

Devices may change.

OpenFiat therefore maintains signed session state across the network.

When a participant reconnects, compatible nodes assist in reconstructing active sessions using previously synchronized signed state.

This mechanism allows trades to continue without depending on a single server maintaining session information.

---

# 5.13 Network Resilience

OFNP assumes that failures are normal.

Nodes disconnect.

Internet connections fail.

Applications crash.

Service providers disappear.

Rather than preventing failures, the protocol is designed to tolerate them.

Every important piece of information exists redundantly across multiple independent nodes.

No individual node is indispensable.

---

# 5.14 Security

Every OFNP message is digitally signed.

Nodes verify:

* Sender identity.
* Message integrity.
* Protocol version.
* Timestamp validity.
* Signature authenticity.

Unsigned or malformed messages are discarded before entering local state.

This prevents unauthorized participants from impersonating merchants or modifying protocol data.

---

# 5.15 Evolution

OFNP is designed as a living protocol.

Future versions may introduce:

* Improved compression.
* Faster synchronization.
* Additional service categories.
* New advertisement formats.
* Improved routing algorithms.
* Enhanced privacy features.

Backward compatibility remains a primary design objective whenever practical.

---

# 5.16 Looking Ahead

The OpenFiat Network Protocol enables decentralized communication.

Communication alone, however, does not create a marketplace.

The next chapter examines how merchants publish advertisements, how buyers discover offers, how pricing works, how merchant availability is managed, and how the decentralized marketplace itself operates.
