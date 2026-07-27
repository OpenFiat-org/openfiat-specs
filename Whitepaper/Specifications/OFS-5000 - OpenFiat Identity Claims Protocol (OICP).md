# OFS-5000 — OpenFiat Identity Claims Protocol (OICP)

**Document ID:** OFS-5000

**Title:** OpenFiat Identity Claims Protocol

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Identity

**Depends On:** OFS-1000, OFS-1400

---

## Abstract

The OpenFiat Identity Claims Protocol (OICP) defines how users, merchants, node operators, and service providers establish verifiable identity claims within the OpenFiat ecosystem while preserving privacy and self-sovereignty.

Unlike traditional financial platforms, OpenFiat does not require centralized identity verification or mandatory Know Your Customer (KYC) procedures to participate in the protocol.

Instead, users voluntarily build trust by proving ownership of communication channels and publishing cryptographically signed identity claims.

Identity remains wallet-centric, portable, and completely controlled by the user.

---

## 1. Introduction

Every decentralized marketplace faces the same challenge:

> **How do participants build trust without giving control of their identity to a centralized authority?**

OpenFiat answers this by separating:

* Identity
* Reputation
* Authentication
* Authorization

Identity answers:

> **Who is controlling this wallet?**

Reputation answers:

> **How has this wallet behaved historically?**

These are intentionally independent concepts.

---

## 2. Scope

This specification defines:

* Identity claims
* Wallet identity
* Contact verification
* Identity publishing
* Identity updates
* Identity revocation
* Claim verification
* Identity portability

This specification does **not** define:

* Reputation
* Governance
* Wallet authentication
* Merchant tiers

---

## 3. Design Goals

The Identity Protocol SHALL:

* Preserve user privacy.
* Avoid mandatory KYC.
* Support self-sovereign identity.
* Allow optional trust building.
* Remain cryptographically verifiable.
* Support future extensibility.

---

## 4. Design Philosophy

OpenFiat intentionally avoids centralized identity providers.

Participation in the network requires only:

* A supported wallet.
* A valid cryptographic signature.

Everything beyond this is optional.

Users reveal only the information they choose to publish.

---

## 5. Identity Model

Identity belongs to the wallet.

Applications, browsers, devices, and nodes are merely interfaces.

A user's identity remains unchanged when they:

* Change phones.
* Change computers.
* Change applications.
* Change OpenFiat nodes.

The wallet is the permanent identity anchor.

---

## 6. Identity Claims

An Identity Claim is a cryptographically signed statement about a wallet.

Examples include:

* Email verified
* Phone number verified
* Telegram account verified
* X (Twitter) account verified
* Discord account verified
* Merchant name published
* Business name published

Claims are independent.

Users decide which claims to publish.

---

## 7. Claim Structure

Every claim contains:

* Claim ID
* Wallet Address
* Claim Type
* Claim Value (or reference)
* Verification Status
* Verification Timestamp
* Expiration (optional)
* Digital Signature

Claims become immutable after publication.

Updates generate new claim versions.

---

## 8. Identity Levels

The protocol defines progressive identity levels.

Participation never requires advancing beyond Level 0.

---

### Level 0 — Wallet Identity

Requirements:

* Valid wallet.
* Successful wallet authentication.

This is the minimum requirement to use OpenFiat.

---

### Level 1 — Verified Contact

Users may verify one or more communication channels using One-Time Password (OTP) verification.

Supported contact methods include:

* Email
* Mobile phone number
* Telegram account
* Discord account
* X (Twitter) account (through account ownership verification)

Verification proves that the wallet owner controls the published communication channel.

Multiple verified contacts may be associated with a single wallet.

---

### Level 2 — Verified Merchant Identity

This level is intended for merchants who wish to publicly establish a recognizable business identity.

Examples include:

* Merchant display name
* Business name
* Brand logo
* Customer support email
* Customer support phone number

These claims improve user confidence but do not imply regulatory approval.

---

### Level 3 — Trusted Infrastructure Provider

Infrastructure providers may publish operational identity information.

Examples include:

* Organization name
* Public documentation
* Service support channels
* Infrastructure contact details
* Incident response contacts

These claims help other operators identify trusted infrastructure providers.

---

## 9. OTP Verification

Contact verification follows a standardized workflow.

```text id="otp-verification"
Wallet Signs Request

↓

Verification Requested

↓

OTP Generated

↓

User Enters OTP

↓

OTP Verified

↓

Signed Identity Claim Published
```

Verification services never obtain wallet private keys.

---

## 10. Claim Verification

Applications SHALL verify:

* Digital signature
* Claim integrity
* Claim expiration
* Claim format
* Claim version

Invalid claims MUST be rejected.

---

## 11. Claim Updates

Users may update identity claims.

Examples include:

* New phone number
* New email
* Updated merchant name
* New Telegram account

Updates create new claim versions.

Historical claims remain archived.

---

## 12. Claim Revocation

Users may revoke claims at any time.

Reasons include:

* Lost phone
* Email compromise
* Business closure
* Account migration

Revocation generates a signed protocol event.

Applications SHOULD immediately stop displaying revoked claims.

---

## 13. Multiple Claims

A wallet may simultaneously possess multiple verified claims.

Example:

```text id="multiple-claims"
Wallet

├── Verified Email

├── Verified Phone

├── Telegram Verified

├── Discord Verified

└── Business Identity
```

Claims are independent.

Removing one claim does not affect the others.

---

## 14. Privacy Model

Identity claims are voluntary.

Users decide:

* Which claims to publish.
* Which claims remain private.
* Which claims to revoke.

Applications SHOULD minimize unnecessary exposure of personal information.

---

## 15. Merchant Identity

Merchants MAY publish additional business information.

Examples:

* Display name
* Business logo
* Customer support contacts
* Operating hours
* Business description

These fields improve marketplace usability without changing protocol behavior.

---

## 16. Infrastructure Identity

Node operators and service providers MAY publish:

* Organization name
* Service documentation
* Support contacts
* Emergency contact channels

These claims assist operational coordination across the network.

---

## 17. Identity Synchronization

Identity events include:

* ClaimCreated
* ClaimUpdated
* ClaimVerified
* ClaimRevoked

Events propagate using OFS-1200.

Every compliant node eventually reaches identical identity state.

---

## 18. Identity Portability

Identity claims belong to the wallet.

Changing:

* Client software
* Device
* Operating system
* Node
* Geographic location

does not affect identity.

Identity remains portable across the OpenFiat ecosystem.

---

## 19. Security Considerations

Implementations MUST protect against:

* Forged identity claims
* Replay attacks
* OTP reuse
* Identity spoofing
* Duplicate claims
* Unauthorized claim updates

All identity claims MUST be digitally signed by the wallet owner.

---

## 20. Performance Considerations

Identity synchronization is relatively lightweight.

Implementations SHOULD optimize:

* Incremental claim updates
* Efficient claim indexing
* Fast verification
* Compact storage

Reference implementations SHOULD store claim records within RocksDB.

---

## 21. Conformance

A compliant implementation MUST:

* Support wallet-based identity.
* Support OTP-verified contact claims.
* Support voluntary identity publication.
* Support claim updates.
* Support claim revocation.
* Support deterministic synchronization.
* Verify digital signatures.
* Preserve historical claim records.

---

## 22. Relationship to Other Specifications

The Identity Claims Protocol provides the trust foundation upon which authentication, reputation, governance, and trading operate.

```text id="identity-architecture"
             Wallet
                │
                ▼
          OFS-5000
     Identity Claims
                │
     ┌──────────┼───────────┐
     ▼          ▼           ▼
OFS-1400   OFS-3000    OFS-4000
 Sessions  Reputation Governance
                │
                ▼
          OFS-2000
      Trade Protocol
```

---

## 23. Summary

The OpenFiat Identity Claims Protocol enables users to build trust without sacrificing privacy or requiring mandatory KYC.

By anchoring identity to wallet ownership and allowing optional, OTP-verified contact claims, OpenFiat creates a flexible identity framework suitable for a global, permissionless financial network.

Identity remains self-sovereign, portable, cryptographically verifiable, and entirely under the user's control, allowing trust to grow organically through voluntary disclosure and observable marketplace behavior rather than centralized identity databases.

The Identity Claims Protocol answers one fundamental question:

**"How can participants prove who they are—only to the extent they choose—while remaining fully in control of their digital identity?"**
