# OFS-1000 — OpenFiat Network Protocol (OFNP)

**Document ID:** OFS-1000

**Title:** OpenFiat Network Protocol

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Network

**Updates:** None

**Obsoletes:** None

---

# Abstract

The OpenFiat Network Protocol (OFNP) defines the foundational networking layer of the OpenFiat ecosystem.

OFNP specifies how nodes identify themselves, establish secure peer-to-peer connections, negotiate supported protocol versions, exchange messages, synchronize state, and expose protocol services.

OFNP intentionally defines only the transport and communication framework. Higher-level functionality—including peer discovery, gossip synchronization, order propagation, reputation exchange, governance, notifications, and oracle updates—is defined by separate OFS specifications.

This separation allows individual protocol components to evolve independently while remaining interoperable.

---

# 1. Introduction

OpenFiat is a decentralized protocol composed of independent nodes operating without centralized coordination.

In order for these nodes to communicate, they require a standardized networking protocol.

OFNP provides this foundation.

Every OpenFiat node implementing the protocol SHALL communicate using OFNP.

The protocol is designed around several key objectives:

* Decentralization.
* Interoperability.
* Forward compatibility.
* High availability.
* Low latency.
* Strong authentication.
* Extensibility.

OFNP does not define business logic.

Its sole responsibility is transporting authenticated protocol messages between participants.

---

# 2. Scope

This specification defines:

* Node identities
* Network transports
* Connection establishment
* Session negotiation
* Protocol version negotiation
* Authentication
* Message framing
* Compression
* Encryption
* Heartbeats
* Peer lifecycle
* Service multiplexing
* Error handling
* Connection termination

This specification does **not** define:

* Peer discovery (OFS-1100)
* Gossip synchronization (OFS-1200)
* Snapshots (OFS-1300)
* Trade messages (OFS-2000)
* Reputation (OFS-3000)
* Governance (OFS-4000)

---

# 3. Design Goals

OFNP is designed to satisfy the following requirements.

## 3.1 Decentralized

No central coordinator SHALL be required.

---

## 3.2 Secure

Every connection SHALL be authenticated.

---

## 3.3 Efficient

Bandwidth overhead SHALL remain minimal.

---

## 3.4 Extensible

Future protocol services SHALL be introduced without modifying OFNP.

---

## 3.5 Language Independent

Implementations SHALL interoperate regardless of programming language.

---

## 3.6 Transport Agnostic

OFNP SHALL define protocol semantics independently of transport implementation.

---

# 4. Underlying Transport

OpenFiat adopts **libp2p** as its standard networking stack.

Every compliant node MUST implement libp2p.

The reference implementation uses Rust-libp2p.

Future compatible implementations may use:

* Rust
* JavaScript
* Go
* Java
* Python

provided they fully conform to this specification.

---

# 5. Transport Protocols

The reference transport stack is:

```text
Application Protocols
        │
      OFNP
        │
Multiplexing (Yamux)
        │
Security (Noise)
        │
Transport (QUIC)
        │
UDP/IP
```

Alternative libp2p transports MAY be introduced through future governance.

---

# 6. Node Identity

Every node possesses a permanent cryptographic identity.

A Node Identity consists of:

* Public key
* Private key
* Peer ID

The Peer ID SHALL be deterministically derived from the public key using libp2p standards.

Private keys MUST remain under the exclusive control of the node operator.

---

# 7. Node Types

OFNP recognizes multiple logical node roles.

A single node MAY implement multiple roles simultaneously.

Examples include:

* Full Node
* Bootstrap Node
* Snapshot Provider
* Notification Gateway
* Oracle Provider
* Risk Intelligence Provider
* Merchant Gateway
* Public API Node

Roles are advertised during peer negotiation.

---

# 8. Connection Lifecycle

Every connection progresses through the following states.

```text
Disconnected

↓

TCP/QUIC Connection

↓

Noise Handshake

↓

Identity Verification

↓

Protocol Negotiation

↓

Service Negotiation

↓

Active Session

↓

Graceful Disconnect
```

Connections SHALL NOT exchange application messages before negotiation completes successfully.

---

# 9. Secure Handshake

The secure handshake MUST establish:

* Peer identity.
* Encryption.
* Session keys.
* Mutual authentication.

OpenFiat adopts the libp2p Noise protocol for authenticated encryption.

Unauthenticated connections MUST be rejected.

---

# 10. Protocol Negotiation

Immediately following authentication, both peers SHALL exchange:

* OFNP version
* Supported OFS specifications
* Supported protocol versions
* Supported optional extensions

Example:

```text
OFNP: 1.0

OFS Supported:

1100 Peer Discovery

1200 Gossip

1300 Snapshots

1400 Sessions

1500 Registry

2000 Trade

3000 Reputation

6000 Notifications
```

Nodes SHALL reject mandatory protocol incompatibilities.

---

# 11. Capability Advertisement

Each node advertises the services it provides.

Examples:

```text
NODE

├── Gossip

├── Snapshot Provider

├── Oracle

├── Notification Gateway

├── REST API

└── Risk Intelligence
```

Capability advertisement enables intelligent peer selection.

---

# 12. Session Establishment

Once negotiation succeeds:

A Session ID SHALL be created.

The Session ID uniquely identifies the active connection.

Sessions terminate when:

* Peer disconnects.
* Timeout occurs.
* Protocol error.
* Manual shutdown.

---

# 13. Message Framing

Every OFNP message SHALL follow a common envelope.

Conceptually:

```text
Envelope

Header

Payload

Authentication
```

The envelope format is intentionally independent of payload type.

Higher-level specifications define payload contents.

---

# 14. Standard Header Fields

Every message header SHALL include:

* Protocol Version
* Specification ID (OFS)
* Message Type
* Sequence Number
* Timestamp
* Payload Length
* Compression Flag
* Authentication Flag

Future versions MAY introduce optional extension fields.

---

# 15. Sequence Numbers

Every session maintains monotonically increasing sequence numbers.

Sequence numbers provide:

* Replay protection.
* Ordering verification.
* Duplicate detection.
* Session integrity.

Duplicate sequence numbers SHALL be discarded.

---

# 16. Message Authentication

Application messages SHALL be authenticated.

Authentication provides:

* Integrity.
* Sender authenticity.
* Tamper detection.

Authentication methods are defined by individual specifications where appropriate.

---

# 17. Compression

Payload compression is optional.

Supported compression algorithms SHALL be negotiated during session establishment.

Compression SHOULD only be applied to payloads exceeding implementation-defined thresholds.

---

# 18. Heartbeats

Active peers SHALL periodically exchange heartbeat messages.

Heartbeats verify:

* Connectivity.
* Latency.
* Session liveness.

Failure to receive heartbeats within the configured timeout SHALL terminate the session.

Heartbeat intervals are implementation-configurable but SHOULD remain consistent across deployments.

---

# 19. Flow Control

Nodes SHALL implement flow control to prevent resource exhaustion.

Implementations SHOULD:

* Limit outstanding requests.
* Apply backpressure.
* Rate-limit excessive peers.
* Prevent buffer overflow.

Flow control mechanisms MUST NOT interfere with legitimate protocol operation.

---

# 20. Multiplexing

Multiple protocol services SHALL share a single network connection.

Examples:

* Gossip
* Trade synchronization
* Notifications
* Reputation updates
* Governance
* Snapshots

Multiplexing minimizes connection overhead while improving network efficiency.

---

# 21. Connection Prioritization

Implementations MAY prioritize traffic based upon message class.

Suggested priority order:

1. Session control
2. Trade synchronization
3. Gossip propagation
4. Reputation updates
5. Governance
6. Snapshots
7. Background synchronization

Prioritization policies SHOULD remain deterministic.

---

# 22. Error Handling

Standard error categories include:

* Invalid Message
* Unsupported Protocol
* Authentication Failure
* Authorization Failure
* Malformed Payload
* Sequence Violation
* Timeout
* Resource Exhaustion
* Internal Error

Errors SHALL be machine-readable.

Human-readable descriptions MAY accompany errors.

---

# 23. Graceful Shutdown

Nodes SHOULD terminate connections gracefully.

Shutdown sequence:

```text
Stop Accepting Requests

↓

Complete Outstanding Responses

↓

Notify Peer

↓

Close Session

↓

Close Transport
```

Abrupt disconnections SHOULD be avoided whenever practical.

---

# 24. Version Compatibility

Nodes MUST advertise:

* Supported OFNP version.
* Supported OFS versions.
* Supported optional extensions.

Version negotiation SHALL occur before application traffic begins.

---

# 25. Future Extensions

OFNP is intentionally minimal.

New capabilities SHALL be introduced through additional OFS specifications rather than modifying the transport protocol.

Examples include:

* New marketplace services.
* Additional notification transports.
* Enhanced governance.
* Future oracle protocols.

---

# 26. Security Considerations

Implementations MUST consider:

* Replay attacks.
* Man-in-the-middle attacks.
* Resource exhaustion.
* Peer impersonation.
* Message tampering.
* Denial-of-service attacks.
* Malicious peers.

Implementations SHOULD apply conservative connection limits and continuous peer validation.

---

# 27. Conformance

An implementation claiming compliance with OFS-1000 MUST:

* Implement libp2p networking.
* Support authenticated Noise sessions.
* Implement protocol negotiation.
* Support multiplexed services.
* Maintain session sequence numbers.
* Authenticate application messages.
* Support capability advertisement.
* Implement graceful shutdown.
* Reject incompatible protocol versions.
* Conform to all mandatory requirements described in this specification.

---

# 28. Relationship to Other Specifications

OFS-1000 defines the transport layer for the entire OpenFiat ecosystem.

Higher-level specifications build upon OFNP but do not modify its behavior.

```
                OFS-0000
        OpenFiat Protocol Suite
                     │
                     ▼
          OFS-1000 Network Protocol
                     │
     ┌───────────────┼────────────────┐
     ▼               ▼                ▼
OFS-1100       OFS-1200         OFS-1300
Peer           Gossip           Snapshot
Discovery      Protocol         Sync
     │               │                │
     └───────────────┼────────────────┘
                     ▼
             All Higher-Level OFS
```

OFNP serves as the universal transport layer for OpenFiat. Every compliant node, regardless of its role or implementation language, relies on this specification to establish secure, authenticated, and interoperable communication across the network.
