# OFS-1100 — Peer Discovery Protocol (PDP)

**Document ID:** OFS-1100

**Title:** Peer Discovery Protocol

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Network

**Depends On:** OFS-1000

---

## Abstract

The OpenFiat Peer Discovery Protocol (PDP) defines how OpenFiat nodes locate, authenticate, evaluate, and maintain connections with peers.

The protocol enables a node joining the network for the first time to discover other participants without requiring centralized coordination.

Peer Discovery is responsible only for locating peers.

It does **not** synchronize marketplace state, exchange advertisements, transfer snapshots, or replicate trade sessions.

Those responsibilities are delegated to other OFS specifications.

---

## 1. Introduction

A decentralized network cannot depend upon static server lists.

Nodes continuously join, leave, upgrade, fail, and relocate.

The purpose of Peer Discovery is to ensure every node can efficiently locate healthy peers while maintaining a resilient and geographically distributed network.

The protocol is designed around four principles:

* Fast bootstrap
* High resiliency
* Geographic diversity
* Sybil resistance

---

## 2. Scope

This specification defines:

* Bootstrap nodes
* Initial network entry
* Peer advertisements
* Peer exchange
* Peer verification
* Peer scoring
* Peer selection
* Connection maintenance
* Peer expiration
* Reconnection

It does not define:

* Gossip messages
* Snapshots
* Reputation scoring
* Marketplace synchronization

---

## 3. Design Goals

Peer Discovery SHALL:

* Operate without centralized control
* Recover from large-scale failures
* Scale to millions of peers
* Prefer healthy infrastructure
* Minimize unnecessary bandwidth
* Prevent excessive peer churn

---

## 4. Peer Identity

Every peer is uniquely identified by:

* Peer ID
* Public Key
* Node Version
* Supported OFS Versions

Peer IDs MUST remain stable across restarts unless intentionally regenerated.

---

## 5. Bootstrap Nodes

New nodes require an initial point of contact.

OpenFiat defines a set of well-known bootstrap endpoints.

Examples:

```text id="bootstrap01"
entry01.openfiat.network

entry02.openfiat.network

entry03.openfiat.network

openfiat.allenhark.com
```

Bootstrap nodes have only one responsibility:

Introduce new participants to the network.

Bootstrap nodes:

* DO NOT approve peers.
* DO NOT maintain exclusive state.
* DO NOT act as coordinators.
* DO NOT become mandatory after initial synchronization.

Once discovery completes, bootstrap nodes become optional.

---

## 6. Bootstrap Process

Initial network entry proceeds as follows.

```text id="bootstrap02"
Node Starts

↓

Load Local Peer Cache

↓

If Empty

↓

Contact Bootstrap Node

↓

Receive Peer List

↓

Validate Peers

↓

Begin Direct Connections

↓

Disconnect Bootstrap
```

Whenever a local cache already exists, bootstrap nodes SHOULD only be contacted if insufficient healthy peers remain.

---

## 7. Local Peer Cache

Every node maintains a persistent peer database.

The peer database SHOULD survive restarts.

Stored information includes:

* Peer ID
* Addresses
* Last Seen
* Latency
* Supported Services
* Protocol Versions
* Success History

The reference implementation stores this information inside RocksDB.

---

## 8. Peer Advertisement

Every node periodically advertises itself.

Advertisements include:

* Peer ID
* Network Addresses
* Supported Services
* Software Version
* OFS Versions
* Timestamp

Advertisements MUST be signed.

Unsigned advertisements MUST be rejected.

---

## 9. Peer Exchange

Connected peers exchange known peer information.

Example:

```text id="peerexchange01"
Node A

knows

120 peers

↓

shares

20 random healthy peers

↓

Node B
```

Peer exchange enables rapid network expansion without overloading bootstrap infrastructure.

---

## 10. Peer Verification

Discovered peers MUST be verified before becoming trusted neighbors.

Verification includes:

* Identity verification
* Successful handshake
* Version compatibility
* Supported services
* Reachability

Peers failing verification SHALL be discarded.

---

## 11. Peer Selection

Nodes SHOULD avoid connecting only to nearby peers.

Selection SHOULD maximize diversity.

Factors include:

* Geography
* Network Provider
* Organization
* Latency
* Node Reputation
* Software Version

A geographically diverse peer set increases network resilience.

---

## 12. Connection Limits

Implementations SHOULD define:

Minimum peers

Target peers

Maximum peers

Example:

```text id="peerlimits01"
Minimum

32

Preferred

96

Maximum

256
```

Exact values are implementation configurable.

---

## 13. Connection Replacement

Nodes periodically evaluate peer quality.

Poor peers MAY be replaced by healthier candidates.

Replacement factors include:

* High latency
* Frequent disconnects
* Outdated protocol versions
* Missing services
* Poor infrastructure reputation

Connection replacement SHOULD occur gradually.

---

## 14. Peer Liveness

Healthy peers periodically exchange:

* Heartbeats
* Ping
* Pong

Failure to respond within timeout results in:

```text id="liveness01"
Healthy

↓

Missed Heartbeats

↓

Temporary Failure

↓

Disconnect

↓

Reconnect Attempt

↓

Peer Removed
```

---

## 15. Reconnection Strategy

Unexpected disconnects SHOULD trigger exponential backoff.

Example:

```text id="backoff01"
1 second

2 seconds

4 seconds

8 seconds

16 seconds

30 seconds

60 seconds
```

Randomized jitter SHOULD be added to reduce synchronized reconnect storms.

---

## 16. Network Diversity

Nodes SHOULD avoid concentrating connections within:

* One cloud provider
* One autonomous system
* One country
* One organization

Diverse connectivity reduces correlated failures.

---

## 17. Bootstrap Independence

After joining the network, normal operation SHALL rely exclusively on peer-to-peer discovery.

If every official bootstrap node disappeared, the network SHOULD continue operating indefinitely.

This property is fundamental to OpenFiat's decentralization goals.

---

## 18. Peer Metadata

Nodes MAY maintain additional metadata.

Examples:

* Average latency
* Successful synchronizations
* Snapshot availability
* Notification support
* Oracle support
* Risk Intelligence support

Metadata assists future peer selection.

### 18.1 Reading a node's own discovery state

`[PROPOSED — NEEDS SIGN-OFF]`

A node SHOULD expose its discovery state for reading (OFS-8200 §7.3, `getPeers`): the peers in its local cache with the metadata above, **its own peer identity**, and the addresses it announces to others.

The last two are the operationally important part. An operator publishing an entrypoint has to hand other operators a complete dialable address, and their own peer identity is the one component of it they cannot read from anywhere else — reconstructing it from a log line is how it gets typed wrong. Reporting the announced addresses answers the other question that is otherwise invisible from outside: whether an operator-declared external address actually took effect, or whether the node is announcing nothing at all.

Counts of successful and failed exchanges are **one node's own measurements of its own exchanges**. Two honest nodes can disagree about both, because they had different exchanges. An implementation SHOULD report them as the counts they are and SHOULD NOT fold them into an uptime percentage or a health score, which would present one node's local experience as a network-wide verdict about a peer.

---

## 19. Service Awareness

Peer discovery includes service discovery.

Nodes advertise supported capabilities.

Examples:

```text id="services01"
Node

✓ Gossip

✓ Snapshot Provider

✓ Oracle

✓ Notification Gateway

✗ Risk Intelligence

✓ Public API
```

Applications may later choose peers based upon required services.

---

## 20. Peer Reputation Integration

Peer Discovery itself does not calculate reputation.

However, it MAY consume scores from OFS-3200.

Nodes SHOULD prefer infrastructure with higher reputation when selecting peers.

Discovery remains decentralized.

No reputation provider can force connection decisions.

---

## 21. Handling Malicious Peers

Nodes SHOULD reject peers exhibiting:

* Invalid signatures
* Excessive spam
* Protocol violations
* Repeated malformed messages
* Resource exhaustion attacks

Temporary bans MAY be applied.

Permanent bans SHOULD require repeated malicious behavior.

---

## 22. Network Partition Recovery

Temporary Internet failures may partition the network.

Recovery occurs naturally.

```text id="partition01"
Network Split

↓

Independent Operation

↓

Connectivity Restored

↓

Peer Exchange

↓

Topology Restored
```

No manual coordination is required.

---

## 23. Peer Expiration

Peers not observed for extended periods SHOULD expire from the local cache.

Expiration policies are implementation configurable.

Expired peers MAY be rediscovered through normal peer exchange.

---

## 24. Privacy Considerations

Peer Discovery intentionally exposes only information required for network operation.

Nodes SHOULD avoid publishing unnecessary metadata.

Private operator information SHOULD NEVER be advertised through Peer Discovery.

---

## 25. Security Considerations

Implementations MUST defend against:

* Sybil attacks
* Eclipse attacks
* Peer poisoning
* Replay attacks
* Fake bootstrap responses
* Address flooding
* Connection exhaustion

Mitigations include:

* Cryptographic identities
* Diverse peer selection
* Connection limits
* Signed advertisements
* Continuous peer validation

---

## 26. Conformance

A compliant implementation MUST:

* Support bootstrap discovery
* Maintain a persistent peer cache
* Verify peer identities
* Exchange peer advertisements
* Support peer exchange
* Periodically evaluate peer health
* Implement peer expiration
* Support reconnection
* Prefer diverse peer selection
* Reject invalid advertisements

---

## 27. Relationship to Other Specifications

Peer Discovery establishes the network topology upon which every other OpenFiat protocol operates.

```text id="discoverymap01"
OFS-1000
Network Protocol
        │
        ▼
OFS-1100
Peer Discovery
        │
        ▼
Connected Peers
        │
 ┌──────┼──────────┐
 ▼      ▼          ▼
OFS-1200  OFS-1300  OFS-1500
Gossip    Snapshot  Services
```

Peer Discovery answers one question:

**"Who should I connect to?"**

Once those connections exist, higher-level protocols synchronize marketplace state, advertisements, trade sessions, governance data, and all other OpenFiat services.
