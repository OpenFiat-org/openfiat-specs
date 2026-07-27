# OFS-7100 — OpenFiat Risk Intelligence Protocol (ORIP)

**Document ID:** OFS-7100

**Title:** OpenFiat Risk Intelligence Protocol

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Security

**Depends On:** OFS-1000, OFS-1500, OFS-2300, OFS-2400, OFS-5000

---

## Abstract

The OpenFiat Risk Intelligence Protocol (ORIP) defines how fraud intelligence, wallet reputation, sanctions information, scam reports, blockchain analytics, and financial crime indicators are shared throughout the OpenFiat ecosystem.

Unlike the Oracle Protocol, which provides informational data such as exchange rates and payment metadata, the Risk Intelligence Protocol provides **security intelligence** used to protect users and merchants from fraud.

The protocol is intentionally provider-neutral. Any qualified organization may operate a Risk Intelligence Provider by implementing the standard interfaces defined in this specification.

Risk Intelligence Providers assist marketplace participants in making informed decisions.

They **do not** replace user judgment, override protocol consensus, or automatically confiscate assets.

---

## 1. Introduction

A decentralized financial marketplace must remain open while protecting honest participants.

Threats include:

* Scam wallets
* Phishing operations
* Stolen funds
* Sanctioned addresses
* Money laundering
* Fraud rings
* Account takeovers
* Payment fraud
* Social engineering
* Terrorist financing
* Ransomware proceeds

No single organization can detect every threat.

OpenFiat therefore allows multiple independent Risk Intelligence Providers to publish security intelligence using a common protocol.

---

## 2. Scope

This specification defines:

* Risk Intelligence Providers
* Wallet intelligence
* Address risk scores
* Fraud reports
* Deposit screening
* Risk categories
* Provider registration
* Intelligence synchronization
* Risk event publication

This specification does **not** define:

* Reputation scoring
* Governance
* Dispute arbitration
* Law enforcement procedures

---

## 3. Design Goals

The Risk Intelligence Protocol SHALL:

* Improve marketplace safety.
* Support multiple providers.
* Preserve decentralization.
* Minimize false positives.
* Allow provider competition.
* Support transparent risk explanations.
* Remain extensible.

---

## 4. Design Philosophy

Risk intelligence is advisory.

It is **not** consensus.

A provider may report:

> "This wallet has been associated with multiple phishing campaigns."

Applications may choose to:

* Reject deposits.
* Warn users.
* Require additional confirmation.
* Ignore the advisory.

The protocol intentionally leaves enforcement policies to applications and governance.

---

## 5. Risk Intelligence Providers

Any qualified organization may become a provider.

Examples include:

* Blockchain analytics companies
* Compliance providers
* Security companies
* Financial institutions
* Community fraud organizations
* Government-approved watchlist publishers
* Academic research groups

Providers register through OFS-1500.

---

## 6. Provider Categories

Examples include:

Blockchain Analytics

* Wallet clustering
* Transaction tracing
* Fund flow analysis

Fraud Intelligence

* Scam wallets
* Fake merchants
* Fraud rings

Compliance

* Sanctions
* AML indicators
* High-risk jurisdictions

Community Intelligence

* User reports
* Confirmed scams
* Phishing campaigns

Infrastructure Intelligence

* Malicious nodes
* Bot networks
* Spam infrastructure

---

## 7. Risk Record

Every published record contains:

* Risk Record ID
* Provider ID
* Wallet Address
* Risk Category
* Severity
* Confidence Score
* Supporting Evidence
* Timestamp
* Expiration (optional)
* Digital Signature

---

## 8. Risk Categories

The protocol initially defines:

Critical

* Stolen funds
* Sanctions
* Terrorist financing
* Ransomware

High

* Known scam wallet
* Confirmed phishing
* Fraud ring
* Money mule

Medium

* Suspicious behavior
* Repeated disputes
* High-risk transaction patterns

Low

* Newly created wallet
* Limited trading history
* Elevated but unconfirmed risk

Informational

* Watchlist
* Ongoing investigation
* Monitoring only

Governance may introduce additional categories.

---

## 9. Confidence Levels

Every report includes a confidence level.

Example:

```text id="confidence"
Very High

High

Medium

Low

Unknown
```

Confidence represents the provider's assessment of the reliability of the reported intelligence.

Applications SHOULD consider both severity and confidence before taking action.

---

## 10. Evidence References

Providers SHOULD publish evidence whenever possible.

Examples include:

* Blockchain transaction references
* Public investigation reports
* Court records
* Sanctions publications
* Fraud reports
* Research publications

Sensitive evidence MAY remain private while exposing verifiable references or cryptographic proofs.

---

## 11. Wallet Screening

Applications SHOULD screen wallet addresses before accepting deposits.

Typical workflow:

```text id="wallet-screening"
User Initiates Deposit

↓

Wallet Submitted

↓

Risk Providers Queried

↓

Results Aggregated

↓

Application Policy Applied

↓

Deposit Accepted

or

Deposit Rejected

or

Manual Review
```

Wallet screening SHOULD occur before escrow creation whenever practical.

---

## 12. Deposit Rejection

Applications MAY automatically reject deposits from wallets matching governance-defined rejection policies.

Examples include:

* Confirmed stolen funds
* Active sanctions
* Known phishing wallets
* Confirmed scam wallets

Applications SHOULD clearly communicate why a deposit was rejected whenever legally appropriate.

---

## 13. Multiple Provider Consensus

Applications SHOULD consult multiple providers.

Example:

```text id="provider-consensus"
Provider A

Scam Wallet

Provider B

Scam Wallet

Provider C

No Record

↓

Application Policy

↓

Reject Deposit
```

Reliance on multiple providers reduces single-provider failure risk.

---

## 14. Risk Updates

Risk intelligence changes over time.

Examples:

* Wallet cleared
* Investigation completed
* Sanctions removed
* Scam confirmed
* False positive corrected

Updated intelligence creates new signed records.

Historical records remain available for audit purposes.

---

## 15. False Positives

No provider is infallible.

Applications SHOULD provide mechanisms for:

* Provider corrections
* Appeals
* Additional evidence
* Re-screening

Incorrect classifications SHOULD be corrected as rapidly as possible.

---

## 16. Community Reporting

Future protocol versions MAY permit community-submitted reports.

Such reports SHOULD:

* Be cryptographically signed.
* Require supporting evidence.
* Undergo provider review before publication.
* Never immediately create critical classifications.

---

## 17. Privacy

Providers SHOULD minimize personal data collection.

Risk records SHOULD focus on:

* Wallet addresses
* Cryptographic identifiers
* Blockchain evidence

Personally identifiable information SHOULD only be included where legally required and properly protected.

---

## 18. Synchronization

Risk intelligence updates generate signed protocol events.

Nodes synchronize updates using OFS-1200.

Applications eventually converge on identical intelligence datasets from the providers they trust.

---

## 19. Security Considerations

Implementations MUST protect against:

* Forged risk reports
* Provider impersonation
* Replay attacks
* Tampered intelligence
* Duplicate reports
* Unauthorized provider registration

Every intelligence record MUST be digitally signed.

---

## 20. Performance Considerations

Risk screening should add minimal latency to trading.

Implementations SHOULD optimize:

* Cached intelligence
* Incremental updates
* Fast address lookups
* Parallel provider queries
* Efficient signature verification

The reference implementation SHOULD maintain indexed risk datasets within RocksDB for high-performance local lookups.

---

## 21. Conformance

A compliant implementation MUST:

* Support multiple Risk Intelligence Providers.
* Verify provider signatures.
* Support wallet screening.
* Support signed intelligence records.
* Preserve historical intelligence.
* Support provider registration.
* Reject malformed intelligence records.
* Synchronize intelligence updates.

---

## 22. Relationship to Other Specifications

The Risk Intelligence Protocol provides a security layer across the entire OpenFiat ecosystem.

```text id="risk-architecture"
           External Intelligence
                  │
                  ▼
       Risk Intelligence Providers
                  │
                  ▼
             OFS-7100
    Risk Intelligence Protocol
                  │
     ┌────────────┼──────────────┐
     ▼            ▼              ▼
 Deposits    Settlements     Disputes
                  │
                  ▼
            Reputation Engine
```

---

## 23. Example Providers

The protocol is provider-neutral.

Potential providers include:

* CipherOwl
* Chainalysis
* TRM Labs
* Elliptic
* Community-operated intelligence providers
* National sanctions list publishers
* Future OpenFiat-certified security providers

Listing an organization within this specification does **not** imply endorsement or mandatory usage.

---

## 24. Future Extensions

Future versions of the protocol may support:

* Real-time fraud streaming
* Cross-chain risk intelligence
* AI-assisted fraud detection
* Device reputation
* Payment account reputation
* Merchant fraud analytics
* Decentralized fraud intelligence marketplaces
* Zero-knowledge risk attestations
* Machine-readable investigation reports

---

## 25. Summary

The OpenFiat Risk Intelligence Protocol establishes a standardized, decentralized framework for sharing security intelligence throughout the OpenFiat ecosystem.

By separating risk intelligence from protocol consensus, OpenFiat preserves its permissionless nature while giving applications powerful tools to identify scams, screen wallet addresses, reject deposits from high-risk wallets, and protect users from evolving financial threats.

The protocol encourages competition among independent intelligence providers, minimizes reliance on any single source of truth, and enables marketplaces to adopt security policies appropriate for their users and jurisdictions.

The Risk Intelligence Protocol answers one fundamental question:

**"How can a decentralized financial network share actionable fraud intelligence without sacrificing openness, interoperability, or user sovereignty?"**
