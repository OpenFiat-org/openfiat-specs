# OFS-0000 — OpenFiat Protocol Suite

**Document ID:** OFS-0000

**Title:** OpenFiat Protocol Suite

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Core Specification

---

## Abstract

The OpenFiat Protocol Suite (OFS) defines the complete collection of open technical specifications that together form the OpenFiat decentralized fiat marketplace.

Rather than existing as a single monolithic specification, OpenFiat is divided into a collection of modular documents. Each document describes a specific protocol, subsystem, or architectural component.

This modular approach allows the protocol to evolve incrementally while maintaining backward compatibility, encouraging independent implementations, and enabling governance to upgrade individual protocol layers without redesigning the entire ecosystem.

---

## 1. Introduction

OpenFiat is an open protocol for decentralized peer-to-peer fiat exchange.

Its purpose is to enable anyone in the world to buy or sell digital stablecoins directly with local fiat currencies without relying on centralized exchanges or custodial intermediaries.

The protocol defines:

* Network communication
* Peer discovery
* Trading
* Escrow
* Settlement
* Reputation
* Governance
* Identity
* Notifications
* External services
* Security intelligence

Every compliant implementation follows these specifications.

---

## 2. Design Principles

The OpenFiat ecosystem is built upon several fundamental principles.

### Open Standards

All protocol specifications are publicly documented.

Anyone may implement them.

---

### Permissionless Participation

Users do not require approval to:

* Run a node
* Build a client
* Operate infrastructure
* Become a merchant
* Integrate OpenFiat

---

### Self Custody

Users always control their own wallets.

Neither OpenFiat nor any infrastructure provider has custody of user assets.

---

### Deterministic Behavior

Identical protocol inputs must produce identical protocol outputs.

This guarantees interoperability between implementations.

---

### Modular Architecture

Each subsystem is independently specified.

New protocol versions can improve one layer without affecting unrelated components.

---

### Progressive Decentralization

OpenFiat initially launches with AllenHark leading development and ecosystem funding.

Governance progressively transitions protocol stewardship to the wider community as the network matures.

---

## 3. Protocol Layer Architecture

The protocol suite is organized into layers.

```text id="ofs-layers"
┌─────────────────────────────────────────────┐
│           Applications & Wallets            │
├─────────────────────────────────────────────┤
│     Reputation • Governance • Identity      │
├─────────────────────────────────────────────┤
│ Trading • Settlement • Disputes • Escrow    │
├─────────────────────────────────────────────┤
│ Discovery • Gossip • Sessions • Registry    │
├─────────────────────────────────────────────┤
│      Transport • Cryptography • Storage     │
└─────────────────────────────────────────────┘
```

Each layer depends only on the layers beneath it.

---

## 4. Protocol Numbering

Every specification belongs to a protocol family.

| Range    | Category                   |
| -------- | -------------------------- |
| OFS-0000 | Core                       |
| OFS-1000 | Network                    |
| OFS-2000 | Marketplace                |
| OFS-3000 | Reputation                 |
| OFS-4000 | Governance                 |
| OFS-5000 | Identity                   |
| OFS-6000 | Notifications              |
| OFS-7000 | Oracle & Risk Intelligence |
| OFS-8000 | Future Extensions          |
| OFS-9000 | Experimental               |

Additional specifications may be introduced through governance.

---

## 5. Current Specifications

### Core

* OFS-0000 — OpenFiat Protocol Suite

---

### Network Layer

* OFS-1000 — OpenFiat Network Protocol (OFNP)
* OFS-1100 — Peer Discovery
* OFS-1200 — Gossip Protocol
* OFS-1300 — Snapshot Synchronization
* OFS-1400 — Session Synchronization
* OFS-1500 — Service Registry
* **OFS-1600 — Service Quality (SWQoS)**
* **OFS-1700 — Node Synchronization**
* **OFS-1800 — Network Security**
* **OFS-1900 — Storage Engine**

---

### Marketplace Layer

* OFS-2000 — OpenFiat Trade Protocol
* OFS-2100 — Advertisement Protocol
* OFS-2200 — Reservation Protocol
* OFS-2300 — Settlement Protocol
* OFS-2400 — Dispute Protocol

---

### Trust Layer

* OFS-3000 — Reputation Engine

---

### Governance Layer

* OFS-4000 — Governance Protocol
* OFS-4100 — Tokenomics Specification
* OFS-4200 — On-Chain Program Architecture

---

### Identity Layer

* OFS-5000 — Identity Claims Protocol

---

### Infrastructure Layer

* OFS-6000 — Notification Protocol

---

### External Services

* OFS-7000 — Oracle Protocol
* OFS-7100 — Risk Intelligence Protocol

---

## 6. Protocol Dependencies

The specifications intentionally build upon one another.

```text id="protocol-dependencies"
          OFS-1000
      Network Foundation
               │
     ┌─────────┴─────────┐
     ▼                   ▼
 OFS-1100           OFS-1200
 Discovery           Gossip
     │                   │
     └─────────┬─────────┘
               ▼
          OFS-1400
         Sessions
               │
               ▼
          OFS-1500
      Service Registry
               │
               ▼
          OFS-2000
      Trade Protocol
               │
      ┌────────┼────────┐
      ▼        ▼        ▼
 O2100     O2200     O2300
 Adverts Reservations Settlement
               │
               ▼
          OFS-2400
          Disputes
               │
      ┌────────┼────────┐
      ▼        ▼        ▼
 O3000     O5000     O6000
 Reputation Identity Notify
               │
      ┌────────┴────────┐
      ▼                 ▼
 O7000              OFS-7100
 Oracle          Risk Intelligence
               │
               ▼
          OFS-4000
         Governance
```

---

## 7. Reference Implementations

The OpenFiat Foundation intends to maintain official reference implementations for educational and interoperability purposes.

Reference implementations are not authoritative.

The specifications themselves define protocol behavior.

Independent implementations are encouraged.

---

## 8. Versioning

Each specification includes:

* Document Version
* Protocol Version
* Compatibility Information
* Status

Possible statuses include:

* Draft
* Proposed Standard
* Standard
* Deprecated
* Obsolete

---

## 9. Backward Compatibility

Protocol changes SHOULD preserve backward compatibility whenever practical.

Breaking changes SHOULD:

* Be explicitly documented.
* Include migration guidance.
* Specify activation procedures.
* Receive governance approval.

---

## 10. Governance

All protocol specifications evolve through the OpenFiat Governance Protocol (OFS-4000).

Governance may:

* Create specifications.
* Modify specifications.
* Deprecate specifications.
* Archive obsolete specifications.

Historical specifications remain permanently accessible.

---

## 11. Licensing

The OpenFiat Protocol Suite is intended to be published under a permissive open specification license, allowing anyone to:

* Read the specifications.
* Implement compatible software.
* Build commercial products.
* Contribute improvements.

Reference implementations may use separate open-source software licenses.

---

## 12. Future Protocol Families

Future protocol groups may include:

* OFS-8000 — Cross-chain interoperability
* OFS-8100 — Cross-chain settlement
* OFS-8200 — Cross-chain identity
* OFS-8300 — Zero-knowledge extensions
* OFS-8400 — AI-assisted marketplace services
* OFS-8500 — Institutional settlement
* OFS-8600 — Hardware security integration
* OFS-8700 — Mobile-first optimizations

These ranges are reserved for future governance.

---

## 13. Summary

The OpenFiat Protocol Suite is the architectural foundation of the OpenFiat ecosystem.

By organizing the protocol into independent, interoperable specifications, OpenFiat enables developers, businesses, infrastructure providers, researchers, and governments to implement only the components they require while remaining fully compatible with the wider network.

This modular architecture encourages innovation, simplifies protocol evolution, and ensures that OpenFiat can continue growing for decades without sacrificing interoperability or decentralization.

The OpenFiat Protocol Suite answers one overarching question:

**"How do all OpenFiat specifications work together to create a global, decentralized, self-custodial fiat marketplace?"**

I would make one structural improvement before moving to the whitepaper: the network layer currently stops at **OFS-1900**, but several critical networking topics deserve their own specifications rather than being buried inside OFS-1000. Specifically:

* **OFS-1600 — Service Quality Protocol (SWQoS)** (priority classes, bandwidth allocation, congestion control)
* **OFS-1700 — Node Synchronization Protocol** (state convergence, checkpoints, recovery)
* **OFS-1800 — Network Security Protocol** (peer authentication, DoS resistance, encryption, abuse prevention)
* **OFS-1900 — Storage Engine Specification** (RocksDB layout, indexing, snapshots, pruning)

These would make the network layer as comprehensive as the trading layer and give OpenFiat a protocol suite comparable in completeness to mature systems like Ethereum's networking stack or the IETF RFC series.
