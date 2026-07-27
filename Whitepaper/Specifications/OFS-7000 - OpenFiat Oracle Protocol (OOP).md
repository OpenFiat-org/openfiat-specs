# OFS-7000 — OpenFiat Oracle Protocol (OOP)

**Document ID:** OFS-7000

**Title:** OpenFiat Oracle Protocol

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Infrastructure

**Depends On:** OFS-1000, OFS-1500

---

## Abstract

The OpenFiat Oracle Protocol (OOP) defines how trusted external information is introduced into the OpenFiat ecosystem without compromising decentralization.

Unlike blockchain-native information, many values required by a global peer-to-peer fiat marketplace exist outside the blockchain. Examples include fiat exchange rates, public holidays, banking schedules, payment network availability, and stablecoin metadata.

The Oracle Protocol standardizes how external information is published, authenticated, synchronized, versioned, and consumed by OpenFiat-compatible implementations.

Importantly, oracles provide **information**, not **authority**. Oracle data may influence application behavior, but it never alters protocol consensus or user ownership of funds.

---

## 1. Introduction

OpenFiat operates across hundreds of countries and payment systems.

Many trading decisions require external information.

Examples include:

* USD/KES exchange rate
* EUR/USD exchange rate
* Stablecoin metadata
* Bank holiday calendars
* Payment network availability
* Country-specific payment methods
* Supported fiat currencies

Without a standard oracle protocol, every implementation would integrate these services differently.

---

## 2. Scope

This specification defines:

* Oracle Providers
* Oracle registration
* Oracle data publication
* Oracle authentication
* Oracle synchronization
* Oracle versioning
* Oracle redundancy
* Oracle failover

This specification does **not** define:

* Risk intelligence
* Scam detection
* Reputation
* Governance voting

---

## 3. Design Goals

The Oracle Protocol SHALL:

* Support multiple independent providers.
* Avoid centralized data sources.
* Preserve deterministic protocol behavior.
* Authenticate every oracle update.
* Support redundancy and failover.
* Allow governance to approve new oracle categories.

---

## 4. Design Philosophy

Oracles provide observations.

They do not execute protocol logic.

For example:

An exchange rate oracle may report:

> 1 USDC ≈ 129.52 KES

Applications may use this information to display estimated fiat values.

However, users remain free to negotiate any exchange rate they choose.

Oracle information never forces a trade price.

---

## 5. Oracle Providers

Anyone may operate an Oracle Provider.

Examples include:

* Financial institutions
* Exchange-rate aggregators
* Stablecoin issuers
* Payment network operators
* Community-operated oracle services
* Infrastructure companies

Providers register through OFS-1500.

---

## 6. Oracle Categories

The initial protocol defines the following categories:

### Exchange Rate

Examples:

* USD/KES
* EUR/USD
* GBP/USD
* USDC/USD

---

### Stablecoin Metadata

Examples:

* Token symbol
* Decimals
* Issuer
* Supported networks
* Contract addresses

---

### Payment Infrastructure

Examples:

* Supported payment rails
* Banking holidays
* Payment network maintenance
* Regional payment availability

---

### Regional Metadata

Examples:

* Supported fiat currencies
* Country identifiers
* Payment method metadata
* Locale information

---

## 7. Oracle Record

Every published record contains:

* Oracle ID
* Provider ID
* Category
* Data payload
* Version
* Timestamp
* Expiration
* Digital signature

---

## 8. Oracle Publication

Updates follow this lifecycle:

```text
Provider

↓

Collect Data

↓

Sign Update

↓

Publish

↓

Network Synchronization

↓

Applications Consume
```

Unsigned oracle updates MUST be rejected.

---

## 9. Exchange Rate Example

```text
Oracle

↓

USD/KES

↓

129.52

↓

Timestamp

↓

Signed Record
```

Applications SHOULD display update timestamps alongside exchange rates.

---

## 10. Stablecoin Metadata

Metadata MAY include:

* Symbol
* Name
* Decimals
* Blockchain
* Mint address
* Official website
* Asset status

Metadata allows applications to present assets consistently.

---

## 11. Redundancy

Applications SHOULD consult multiple providers.

Example:

```text
Provider A

129.50

Provider B

129.54

Provider C

129.51

↓

Median

↓

129.52
```

No single provider should become a mandatory dependency.

---

## 12. Oracle Expiration

Oracle records expire.

Expired data SHOULD NOT be treated as current.

Applications SHOULD indicate stale information to users.

---

## 13. Synchronization

Oracle updates generate protocol events.

Nodes synchronize updates through OFS-1200.

Applications eventually converge on identical oracle datasets.

---

## 14. Failure Handling

If a provider becomes unavailable:

* Cached records remain usable until expiration.
* Alternate providers should be consulted.
* Applications SHOULD warn users if fresh data is unavailable.

Marketplace operation continues even without oracle availability.

---

## 15. Security Considerations

Implementations MUST reject:

* Invalid signatures
* Expired records
* Tampered payloads
* Duplicate updates
* Replay attacks
* Unauthorized providers

---

## 16. Performance Considerations

Oracle updates are relatively infrequent.

Implementations SHOULD optimize:

* Incremental synchronization
* Efficient caching
* Signature verification
* Compact storage

---

## 17. Conformance

A compliant implementation MUST:

* Support multiple Oracle Providers.
* Verify signatures.
* Support record expiration.
* Synchronize oracle updates.
* Reject unauthorized providers.
* Support provider redundancy.

---

## 18. Relationship to Other Specifications

```text
External Data Sources

        │

        ▼

   Oracle Providers

        │

        ▼

     OFS-7000

 Oracle Protocol

        │

 ┌──────┼────────┐

 ▼      ▼        ▼

Wallet UI Trading Analytics
```

---

## 19. Summary

The OpenFiat Oracle Protocol provides a standardized, decentralized mechanism for introducing external information into the OpenFiat ecosystem.

By treating oracle data as authenticated observations rather than authoritative protocol inputs, OpenFiat remains permissionless while enabling rich user experiences, accurate pricing information, and globally interoperable marketplace applications.

The Oracle Protocol answers one essential question:

**"How can decentralized applications safely consume trusted external information without creating centralized dependencies?"**

---

At this point, the protocol suite is essentially complete. The remaining companion documents are no longer protocol specifications but ecosystem documents:

* **OpenFiat Whitepaper** (vision, architecture, economics)
* **Tokenomics Paper**
* **Foundation Charter**
* **Developer Handbook**
* **Reference Client Specification**
* **Node Operator Guide**
* **Merchant Guide**
* **Security Best Practices**
* **Governance Handbook**

I would also create **OFS-7100 — Risk Intelligence Protocol** as a separate specification to formally define wallet flag providers (e.g., CipherOwl), sanctions lists, fraud signals, and deposit rejection policies. That keeps security intelligence independent from the general-purpose oracle framework and aligns well with the architecture we've already designed.
