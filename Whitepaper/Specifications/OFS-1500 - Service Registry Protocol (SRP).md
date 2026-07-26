# OFS-1500 — Service Registry Protocol (SRP)

**Document ID:** OFS-1500

**Title:** Service Registry Protocol

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Network

**Depends On:** OFS-1000, OFS-1100, OFS-1200

---

# Abstract

The OpenFiat Service Registry Protocol (SRP) defines how infrastructure providers advertise, discover, verify, and update services available across the OpenFiat network.

Rather than hardcoding infrastructure endpoints, OpenFiat uses a decentralized service registry that allows any qualified participant to offer protocol services.

The Service Registry enables automatic discovery of services such as Snapshot Providers, Notification Gateways, Oracle Providers, Risk Intelligence Providers, Public API Nodes, Bootstrap Nodes, and future protocol extensions.

The registry itself is decentralized and replicated across every OpenFiat node through the Gossip Protocol.

---

# 1. Introduction

OpenFiat is more than a marketplace.

It is an ecosystem composed of many independent infrastructure providers.

Examples include:

* Snapshot Providers
* Notification Providers
* SMS Gateways
* Email Gateways
* Push Notification Providers
* Oracle Providers
* Risk Intelligence Providers
* Public API Providers
* Bootstrap Nodes
* Explorer Nodes

Applications need a standardized way to discover these services.

The Service Registry provides that mechanism.

---

# 2. Scope

This specification defines:

* Service registration
* Service advertisement
* Service discovery
* Service updates
* Service withdrawal
* Service verification
* Service capabilities
* Service health
* Service metadata

This specification does not define:

* Service-specific APIs
* Notification delivery
* Oracle protocols
* Risk intelligence formats

Those are defined by their respective OFS specifications.

---

# 3. Design Goals

The Service Registry SHALL:

* Operate without centralized infrastructure.
* Allow permissionless participation.
* Support millions of registered services.
* Enable deterministic discovery.
* Support future service types.
* Remain cryptographically verifiable.

---

# 4. Registry Philosophy

The Service Registry is **a directory**, not a marketplace.

It answers questions such as:

* Which Snapshot Providers exist?
* Which Oracle Providers support KES/USD?
* Which Notification Gateway supports Telegram?
* Which provider offers SMS in Kenya?
* Which Risk Intelligence Providers are online?

The registry does **not** recommend providers.

Selection remains the responsibility of the client.

---

# 5. Service Provider Identity

Every service provider SHALL possess:

* Wallet Address
* Node Identity
* Peer ID
* Public Key

All registrations MUST be digitally signed.

A provider MAY operate multiple services.

Each service receives its own Service ID.

---

# 6. Service Types

Initial service types include:

Infrastructure

* Bootstrap Node
* Snapshot Provider
* Public API Node

Marketplace

* Merchant Gateway
* Analytics Provider

Notifications

* Email Gateway
* Telegram Gateway
* SMS Gateway
* Push Gateway
* Webhook Gateway

Market Data

* Price Oracle
* FX Oracle

Security

* Risk Intelligence Provider
* Wallet Flagging Provider

Future service types MAY be introduced through governance.

---

# 7. Service Registration

To register a service, a provider submits a signed registration event.

The registration SHALL include:

* Service ID
* Service Type
* Provider Identity
* Network Endpoints
* Supported Protocol Versions
* Geographic Region
* Capabilities
* Pricing Information (if applicable)
* Timestamp
* Signature

Registration events are propagated through the Gossip Protocol.

---

# 8. Service Identifier

Every registered service SHALL possess a globally unique Service ID.

The Service ID remains constant for the lifetime of the service.

Changing endpoints or metadata SHALL NOT change the Service ID.

---

# 9. Service Metadata

Every service advertises metadata describing its capabilities.

Examples include:

Notification Gateway

* Telegram
* Email
* SMS
* Web Push
* Mobile Push

Oracle

* Supported Currency Pairs
* Update Frequency
* Data Sources

Snapshot Provider

* Snapshot Frequency
* Compression Formats
* Historical Retention

Risk Provider

* Supported Risk Categories
* Refresh Frequency

Metadata enables intelligent client selection.

---

# 10. Geographic Coverage

Providers MAY advertise regions served.

Example:

```text id="geo-services"
SMS Gateway

Regions

✓ Kenya

✓ Uganda

✓ Tanzania

✗ Europe

✓ Rwanda
```

Regional information assists clients in selecting nearby or jurisdiction-specific services.

---

# 11. Service Health

Providers periodically publish health updates.

Health information MAY include:

* Online
* Maintenance
* Degraded
* Offline

Health advertisements allow applications to avoid unavailable infrastructure.

---

# 12. Service Discovery

Clients discover services by querying their local registry.

Examples:

* Find Telegram providers.
* Find Snapshot Providers.
* Find KES/USD Oracles.
* Find Wallet Flagging Providers.
* Find Public API Nodes.

No centralized lookup server is required.

---

# 13. Multiple Providers

Clients SHOULD support multiple providers for every service category.

Example:

```text id="provider-selection"
Telegram

Provider A

Provider B

Provider C

↓

Client Chooses
```

Multiple providers improve resilience and competition.

---

# 14. Provider Selection

Selection policies are implementation specific.

Factors MAY include:

* Reputation
* Latency
* Geographic proximity
* Supported features
* Cost
* Availability
* Historical reliability

The protocol intentionally does not mandate provider selection algorithms.

---

# 15. Provider Pricing

Some infrastructure providers charge fees.

Examples:

* Notification delivery
* Premium API access
* High-frequency oracle feeds

Pricing information MAY be advertised as metadata.

Actual payment mechanisms are defined by the individual service specifications.

---

# 16. Service Updates

Providers MAY update:

* Endpoints
* Capabilities
* Pricing
* Regions
* Protocol Versions
* Contact Information

Updates SHALL preserve the existing Service ID.

Every update MUST be digitally signed.

---

# 17. Service Withdrawal

Providers may voluntarily remove services.

Withdrawal sequence:

```text id="withdraw-service"
Provider

↓

Signed Withdrawal

↓

Broadcast

↓

Registry Updated

↓

Clients Remove Service
```

Withdrawn services SHALL no longer be returned during discovery.

---

# 18. Automatic Expiration

If a provider stops publishing health updates for an extended period, its registration SHALL automatically expire.

Expired services remain recoverable through re-registration.

This prevents stale entries from accumulating.

---

# 19. Registry Replication

The registry is fully decentralized.

Every node maintains a local copy.

Changes propagate through the Gossip Protocol.

The registry therefore remains available even if individual providers become unreachable.

---

# 20. Service Categories and Future Expansion

The Service Registry is intentionally extensible.

Future services may include:

* AI Fraud Analysis
* Compliance Providers
* Payment Verification Providers
* Merchant Analytics
* Regional Liquidity Services
* Escrow Auditors

No changes to the underlying registry protocol are required.

Only new Service Types need to be defined.

---

# 21. Security Considerations

Nodes MUST reject:

* Invalid signatures
* Duplicate Service IDs
* Malformed registrations
* Unauthorized updates
* Replay attacks
* Version incompatibilities

Nodes SHOULD rate-limit excessive registration attempts to reduce spam.

---

# 22. Privacy Considerations

Providers SHOULD advertise only operational metadata.

Sensitive internal infrastructure details SHOULD NOT be included.

The registry is intended for service discovery, not infrastructure inventory.

---

# 23. Performance Considerations

The registry is expected to scale to hundreds of thousands of infrastructure providers.

Implementations SHOULD optimize:

* Fast service lookup
* Efficient indexing
* Incremental updates
* Compact metadata storage

The reference implementation stores registry entries in RocksDB for efficient local queries.

---

# 24. Conformance

A compliant implementation MUST:

* Support signed service registrations.
* Maintain a local replicated registry.
* Verify provider identities.
* Support service discovery.
* Support updates.
* Support withdrawals.
* Support expiration.
* Reject invalid registrations.
* Preserve deterministic registry state.

---

# 25. Relationship to Other Specifications

The Service Registry is the discovery layer for infrastructure services across the OpenFiat ecosystem.

```text id="service-registry-architecture"
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
             OFS-1500
         Service Registry
                  │
      ┌───────────┼────────────┬────────────┐
      ▼           ▼            ▼            ▼
 Snapshot     Notification   Oracle      Risk
 Providers     Providers    Providers   Providers
                  │
                  ▼
          Client Applications
```

The Service Registry answers one essential question:

**"What services are available on the OpenFiat network, and how can they be discovered without relying on centralized infrastructure?"**

By maintaining a decentralized, cryptographically verifiable directory of infrastructure providers, the Service Registry enables OpenFiat to grow organically while preserving interoperability, competition, and the permissionless nature of the network.
