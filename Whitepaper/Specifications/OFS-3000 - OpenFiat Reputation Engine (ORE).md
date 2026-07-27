# OFS-3000 — OpenFiat Reputation Engine (ORE)

**Document ID:** OFS-3000

**Title:** OpenFiat Reputation Engine

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Marketplace

**Depends On:** OFS-2000, OFS-2100, OFS-2200, OFS-2300, OFS-2400, OFS-5000

---

## Abstract

The OpenFiat Reputation Engine (ORE) defines how trust is established, measured, maintained, and utilized across the OpenFiat ecosystem.

Unlike traditional marketplaces that rely on simple star ratings, OpenFiat employs a multi-dimensional reputation system derived from objectively measurable protocol events.

Every completed trade, settlement, dispute, cancellation, and operational metric contributes to a participant's reputation.

Reputation is earned through consistent, verifiable behavior over time.

---

## 1. Introduction

Trust is the foundation of peer-to-peer trading.

Without an effective reputation system:

* Users cannot distinguish experienced merchants.
* Fraud becomes easier.
* Honest participants receive no long-term advantage.
* Marketplace quality deteriorates.

OpenFiat addresses these challenges through a decentralized reputation engine that rewards measurable reliability instead of subjective opinions.

---

## 2. Scope

This specification defines:

* Reputation metrics
* Reputation calculation
* Reputation updates
* Reputation decay
* Merchant tiers
* Reputation events
* Reputation portability
* Anti-abuse mechanisms

This specification does not define:

* Node reputation (OFS-1600)
* Governance voting
* Token rewards

---

## 3. Design Goals

The Reputation Engine SHALL:

* Reward honest behavior.
* Penalize abusive behavior.
* Resist manipulation.
* Scale globally.
* Be deterministic.
* Remain explainable.
* Operate without centralized moderation.

---

## 4. Design Philosophy

OpenFiat intentionally avoids simplistic rating systems.

Users do **not** rate one another using stars.

Instead, reputation is derived from protocol-observed behavior.

Examples include:

* Settlement completed
* Payment confirmed
* Dispute resolved
* Reservation cancelled
* Merchant online
* Response time

Protocol data is significantly harder to manipulate than subjective reviews.

---

## 5. Reputation Identity

Every wallet possesses its own reputation profile.

The profile is tied to the wallet, not to a device or application.

Reputation travels with the wallet across all OpenFiat-compatible clients.

---

## 6. Reputation Dimensions

Rather than a single score, reputation consists of multiple independent dimensions.

These dimensions allow applications to evaluate participants based on different aspects of performance.

The initial protocol defines the following core dimensions:

* Settlement Speed
* Trade Success Rate
* Dispute Rate
* Trade Volume
* Average Ticket Size
* Merchant Age
* Availability
* Payment Accuracy

Future protocol versions may introduce additional dimensions through governance.

---

## 7. Settlement Speed

Settlement Speed measures how quickly a participant completes trades after reservation.

Factors include:

* Average settlement duration
* Median settlement duration
* Slow settlement frequency
* Fast settlement consistency

Consistently fast settlement improves marketplace ranking.

---

## 8. Trade Success Rate

Trade Success Rate measures successfully completed trades relative to initiated trades.

Example calculation:

```text id="trade-success"
Completed Trades

÷

Started Trades

=

Trade Success %
```

Only protocol-completed trades contribute.

Cancelled trades are evaluated separately.

---

## 9. Dispute Rate

Dispute Rate measures the percentage of completed trades that required formal dispute resolution.

Example:

```text id="dispute-rate"
Disputed Trades

÷

Completed Trades

=

Dispute Rate
```

Occasional disputes are expected.

Repeated disputes reduce trust.

---

## 10. Trade Volume

Trade Volume measures the cumulative value settled by a participant.

Metrics MAY include:

* Lifetime volume
* 30-day volume
* Annual volume
* Stablecoin-specific volume
* Fiat-specific volume

Large volume alone does not guarantee high reputation.

---

## 11. Average Ticket Size

Average Ticket Size measures the typical value of completed trades.

This allows users to identify merchants experienced with:

* Small retail trades
* Medium-sized trades
* Institutional-sized settlements

Applications MAY filter merchants based on preferred trade size.

---

## 12. Merchant Age

Merchant Age represents the duration since a merchant first became active on the network.

Older merchants generally possess longer trading histories.

Merchant Age alone does not increase reputation.

It serves as contextual information.

---

## 13. Availability

Availability measures operational reliability.

Examples include:

* Online time
* Response rate
* Response latency
* Reservation acceptance
* Missed reservations

Merchants who remain consistently available receive higher availability scores.

---

## 14. Payment Accuracy

Payment Accuracy measures the frequency of settlement discrepancies.

Examples include:

* Incorrect payment amount
* Wrong payment reference
* Duplicate payments
* Incorrect account usage

High payment accuracy benefits both buyers and merchants.

---

## 15. Reputation Events

The following protocol events update reputation:

Positive:

* Trade Completed
* Settlement Approved
* Fast Settlement
* Reservation Honored
* Dispute Won (where appropriate)

Negative:

* Reservation Timeout
* Excessive Cancellation
* Fraud Attempt
* Invalid Payment
* False Evidence
* Repeated Disputes

Each event contributes only to relevant reputation dimensions.

---

## 16. Reputation Calculation

The protocol intentionally does not define a fixed mathematical formula.

Instead, it defines:

* Input metrics
* Event definitions
* Update rules

This allows governance to improve scoring algorithms without changing the protocol itself.

Implementations SHALL produce deterministic results for a given algorithm version.

---

## 17. Reputation Decay

Operational metrics SHOULD emphasize recent activity.

Inactive merchants gradually lose operational prominence while preserving their historical accomplishments.

Examples of decaying metrics include:

* Availability
* Response Speed
* Settlement Speed

Historical metrics such as lifetime trade volume remain permanent.

---

## 18. Merchant Tiers

Applications MAY classify merchants into tiers.

Example:

```text id="merchant-tier"
Explorer

↓

Verified

↓

Professional

↓

Elite

↓

Institutional
```

Tier requirements are defined by governance.

Tiers are descriptive.

They never bypass protocol rules.

---

## 19. Reputation Portability

Reputation belongs to the wallet.

Changing:

* Client application
* Device
* Node
* Region

does not affect reputation.

Users retain their marketplace history across the entire OpenFiat ecosystem.

---

## 20. Anti-Manipulation

The Reputation Engine is designed to resist abuse.

Examples include:

* Self-trading
* Wash trading
* Fake disputes
* Reputation farming
* Coordinated manipulation
* Artificial settlement inflation

Future protocol versions MAY incorporate graph analysis and anomaly detection to identify sophisticated abuse patterns.

---

## 21. Reputation Visibility

Applications MAY expose:

* Overall score
* Individual dimensions
* Historical trends
* Merchant tier
* Volume statistics
* Settlement statistics

Applications SHOULD explain why a reputation score was assigned.

Transparency builds trust.

---

## 22. Marketplace Ranking

Marketplace search results MAY consider:

* Reputation dimensions
* Price competitiveness
* Geographic proximity
* Preferred payment method
* Merchant specialization
* Availability

The protocol deliberately avoids mandating a universal ranking algorithm.

---

## 23. Reputation History

Every reputation update creates an immutable protocol event.

Historical reputation enables:

* Trend analysis
* Fraud detection
* Merchant growth tracking
* Governance research

Historical events are never rewritten.

---

## 24. Reputation Synchronization

Reputation updates propagate using the Gossip Protocol.

Every compliant node eventually reaches identical reputation state after processing the same sequence of events.

---

## 25. Privacy

While overall reputation is generally public, applications SHOULD protect sensitive operational details.

Examples include:

* Exact banking information
* Device identifiers
* Private communication
* Internal risk scores

Only information necessary for marketplace trust should be publicly exposed.

---

## 26. Security Considerations

Implementations MUST protect against:

* Reputation forgery
* Duplicate events
* Replay attacks
* Identity impersonation
* Artificial reputation inflation
* Reputation rollback

Only cryptographically verified protocol events SHALL modify reputation.

---

## 27. Performance Considerations

The Reputation Engine is expected to process millions of updates.

Implementations SHOULD optimize:

* Incremental calculations
* Historical indexing
* Fast merchant lookup
* Efficient synchronization

The reference implementation stores reputation state within RocksDB for deterministic local computation.

---

## 28. Conformance

A compliant implementation MUST:

* Support multi-dimensional reputation.
* Update reputation from protocol events.
* Preserve historical records.
* Support deterministic synchronization.
* Resist replay attacks.
* Support merchant tiers.
* Support reputation portability.
* Reject unauthorized reputation modifications.

---

## 29. Relationship to Other Specifications

The Reputation Engine integrates information from nearly every OpenFiat trading protocol.

```text id="reputation-architecture"
             OFS-2100
        Advertisements
                  │
                  ▼
             OFS-2200
         Reservations
                  │
                  ▼
             OFS-2300
          Settlement
                  │
                  ▼
             OFS-2400
            Disputes
                  │
                  ▼
             OFS-3000
       Reputation Engine
                  │
      ┌───────────┼────────────┐
      ▼           ▼            ▼
 Merchant     Search      Merchant
  Ranking    Ordering      Tiers
```

---

## 30. Summary

The OpenFiat Reputation Engine transforms trust from subjective opinion into objective protocol behavior.

Rather than relying on star ratings or user reviews, OpenFiat measures real marketplace performance through settlement speed, trade success, dispute history, trading volume, payment accuracy, availability, and other verifiable protocol events.

This approach produces a reputation system that is transparent, portable, resistant to manipulation, and capable of scaling to millions of participants while remaining entirely decentralized.

The Reputation Engine answers one fundamental question:

**"Based on verifiable marketplace behavior, how trustworthy is this participant today?"**
