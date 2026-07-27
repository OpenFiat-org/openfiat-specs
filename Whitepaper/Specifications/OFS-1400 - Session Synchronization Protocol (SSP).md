# OFS-1400 — Session Synchronization Protocol (SSP)

**Document ID:** OFS-1400

**Title:** Session Synchronization Protocol

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Network

**Depends On:** OFS-1000, OFS-1100, OFS-1200, OFS-1300

---

## Abstract

The OpenFiat Session Synchronization Protocol (SSP) defines how authenticated user sessions are propagated, synchronized, recovered, and invalidated across the decentralized OpenFiat network.

Unlike blockchain state, user sessions are ephemeral. They represent temporary authenticated connections between wallets, user interfaces, merchant software, notification providers, and infrastructure services.

Session Synchronization ensures that any OpenFiat node can continue servicing an authenticated user without requiring centralized session storage while preserving security, privacy, and fault tolerance.

---

## 1. Introduction

Traditional applications store user sessions inside centralized databases.

This approach introduces:

* Single points of failure.
* Vendor lock-in.
* Poor failover.
* Difficult horizontal scaling.

OpenFiat adopts a different approach.

Authenticated sessions are replicated across the peer-to-peer network using deterministic synchronization rules.

A user's session should survive:

* UI refreshes.
* Node restarts.
* Load balancing.
* Infrastructure failures.
* Geographic failover.

At no point should a centralized session server be required.

---

## 2. Scope

This specification defines:

* Session creation
* Session identifiers
* Session propagation
* Session replication
* Session recovery
* Session expiration
* Session revocation
* Session migration
* Session validation

This specification does not define:

* Wallet authentication
* Identity claims
* Trade logic
* Notification delivery

Authentication is defined separately in **OFS-5100**.

---

## 3. Design Goals

The protocol SHALL:

* Eliminate centralized session databases.
* Preserve user privacy.
* Support seamless failover.
* Minimize synchronization latency.
* Scale to millions of active sessions.
* Remain cryptographically verifiable.

---

## 4. Session Philosophy

A session represents **temporary authorization**, not permanent identity.

Identity belongs to the wallet.

Authorization belongs to the signed session.

Application state belongs to the client.

Marketplace state belongs to the network.

This separation minimizes trust and simplifies recovery.

---

## 5. Session Creation

Sessions are created after successful wallet authentication.

Typical flow:

```text id="session-create"
Wallet Connect

↓

Challenge Generated

↓

Wallet Signs Challenge

↓

Signature Verified

↓

Session Created

↓

Session Broadcast

↓

User Authenticated
```

The signing process is specified in OFS-5100.

---

## 6. Session Identifier

Every session SHALL possess a globally unique Session ID.

A Session ID SHALL include sufficient entropy to prevent prediction or collision.

Session IDs MUST remain immutable throughout the session lifetime.

---

## 7. Session Record

Each session contains:

* Session ID
* Wallet Address
* Authentication Timestamp
* Expiration Time
* Connected Client
* Current Node
* Supported Permissions
* Session Version
* Cryptographic Signature

No private keys are ever stored.

---

## 8. Signed Sessions

Every session MUST be cryptographically signed.

The signature proves that:

* The wallet authenticated.
* The session has not been modified.
* The session originated from a valid authentication flow.

Unsigned sessions MUST be rejected.

---

## 9. Session Propagation

New sessions are propagated through the Gossip Protocol.

Only authenticated nodes may relay session records.

Propagation ensures neighboring nodes can recognize valid sessions immediately.

---

## 10. Session Replication

Sessions SHALL be replicated across multiple peers.

Replication enables:

* High availability.
* Fast recovery.
* Node replacement.
* Geographic redundancy.

No single node owns a session.

---

## 11. Session Ownership

Although replicated, only one node is considered the **Primary Session Host** at any given time.

The Primary Session Host:

* Maintains live client communication.
* Processes session activity.
* Emits session updates.

If the host becomes unavailable, another synchronized node may assume responsibility.

---

## 12. Session Migration

Users may seamlessly move between nodes.

Example:

```text id="session-migrate"
Node A

↓

Disconnect

↓

Node B

↓

Session Verified

↓

Continue Without Login
```

Provided the session remains valid, re-authentication is not required.

---

## 13. Session Updates

Session changes generate synchronization events.

Examples include:

* Permission changes.
* Notification preferences.
* Connected devices.
* Active client metadata.

Every update creates a new signed session version.

---

## 14. Session Expiration

Sessions expire automatically.

Expiration criteria MAY include:

* Maximum lifetime.
* User logout.
* Wallet disconnect.
* Extended inactivity.
* Security policy.

Expired sessions MUST NOT be accepted.

---

## 15. Session Renewal

Before expiration, a client MAY request renewal.

Renewal requires:

* Existing valid session.
* Fresh cryptographic proof if required by policy.
* Updated expiration timestamp.

Renewal does not create a new identity.

It extends authorization.

---

## 16. Session Revocation

Sessions may be revoked immediately.

Reasons include:

* User logout.
* Device compromise.
* Wallet compromise.
* Administrative security action.
* Protocol violation.

Revocation events SHALL propagate immediately across the network.

---

## 17. Session Recovery

If a node unexpectedly fails:

```text id="session-recovery"
Node Failure

↓

Neighbor Detects Loss

↓

Session Retrieved

↓

New Host Assigned

↓

Client Reconnects

↓

Continue Session
```

Recovery SHOULD occur without requiring user intervention whenever possible.

---

## 18. Session Consistency

Session synchronization guarantees eventual consistency.

Temporary differences between replicas are acceptable.

Permanent divergence is not.

Conflicting session versions are resolved using deterministic version ordering defined by this specification.

---

## 19. Offline Clients

Temporary client disconnections do not immediately terminate sessions.

Grace periods are implementation configurable.

This allows:

* Mobile network changes.
* Browser refreshes.
* Wi-Fi interruptions.

---

## 20. Concurrent Devices

A wallet MAY maintain multiple simultaneous sessions.

Examples:

* Desktop
* Mobile
* Merchant Terminal
* API Client

Each session receives an independent Session ID.

Revoking one session MUST NOT revoke unrelated sessions.

---

## 21. Session Privacy

Session synchronization intentionally minimizes replicated information.

Nodes SHOULD replicate only the metadata necessary for protocol operation.

Application-specific user data SHOULD remain on the client whenever practical.

---

## 22. Storage

The reference implementation stores session records inside RocksDB.

Implementations MAY use different storage engines provided deterministic behavior is preserved.

Session storage MUST survive unexpected node restarts.

---

## 23. Security Considerations

Implementations MUST protect against:

* Session replay.
* Session hijacking.
* Session forgery.
* Unauthorized migration.
* Expired session reuse.
* Duplicate Session IDs.
* Tampered session records.

All session state SHALL remain cryptographically verifiable.

---

## 24. Performance Considerations

Session synchronization is expected to support millions of concurrent authenticated users.

Implementations SHOULD optimize:

* Incremental updates.
* Efficient replication.
* Fast lookup.
* Low-latency failover.
* Minimal bandwidth consumption.

---

## 25. Relationship to Wallet Authentication

Wallet authentication proves identity.

Session Synchronization distributes temporary authorization.

Identity is never transferred between nodes.

Only signed authorization records are synchronized.

---

## 26. Conformance

A compliant implementation MUST:

* Support signed session records.
* Generate globally unique Session IDs.
* Replicate sessions across peers.
* Support seamless migration.
* Support session expiration.
* Support immediate revocation.
* Reject invalid signatures.
* Preserve deterministic session state.

---

## 27. Relationship to Other Specifications

Session Synchronization provides the authenticated user layer built on top of the networking stack.

```text id="session-architecture"
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
                  ▼
             OFS-1400
     Session Synchronization
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
  OFS-2000   OFS-6000  OFS-4000
   Trading  Notifications Governance
```

Session Synchronization answers one fundamental question:

**"How can authenticated users move seamlessly across a decentralized network without relying on centralized session infrastructure?"**

By replicating signed authorization records instead of centralized login state, OpenFiat provides resilient, fault-tolerant authentication while preserving user sovereignty, decentralization, and interoperability across every compliant implementation.
