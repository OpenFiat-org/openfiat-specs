# Chapter 10 — The OpenFiat Marketplace Risk Engine

## 10.1 Introduction

Financial marketplaces are constantly changing.

A participant who has behaved honestly for years may suddenly begin exhibiting unusual behavior.

A bank account may become temporarily unavailable.

A payment provider may experience outages.

A merchant's systems may become compromised.

A node operator may suffer network instability.

An arbitrator may begin participating dishonestly.

These situations cannot be identified solely by examining historical reputation.

For this reason, OpenFiat introduces the **Marketplace Risk Engine**.

Unlike the Reputation Engine, which measures long-term trust earned through historical behavior, the Risk Engine continuously evaluates current protocol activity for signs of abnormal or potentially harmful behavior.

The Risk Engine does **not** determine guilt.

It does **not** confiscate funds.

It does **not** alter escrow.

It does **not** replace arbitration.

Instead, it provides transparent, deterministic risk signals that allow participants and client applications to make better-informed decisions.

The Risk Engine is advisory, not authoritative.

---

## 10.2 Design Objectives

The Marketplace Risk Engine was designed around several guiding principles.

### Real-Time Awareness

Identify unusual behavior as it develops rather than relying exclusively on historical reputation.

### Deterministic

Every compliant implementation must produce identical risk calculations from identical protocol events.

### Explainable

Every risk signal must be traceable to observable protocol activity.

### Transparent

The algorithms used to calculate risk are publicly documented.

### Non-Custodial

Risk signals never authorize movement of user assets.

### Adaptive

Risk naturally decreases as abnormal behavior subsides.

---

## 10.3 Reputation vs. Risk

Although closely related, reputation and risk measure fundamentally different characteristics.

**Reputation answers:**

> "How trustworthy has this participant been over time?"

**Risk answers:**

> "Is something unusual happening right now?"

A merchant may have completed twenty-five thousand successful trades over several years while simultaneously experiencing elevated short-term risk because of:

* Banking outages.
* Sudden connectivity problems.
* A compromised payment account.
* Rapid increases in disputes.
* Abnormal trading behavior.

Likewise, a new participant may have little reputation but exhibit very low operational risk.

The protocol therefore maintains these systems independently.

---

## 10.4 Advisory by Design

One of the most important principles of OpenFiat is that the Marketplace Risk Engine never controls funds.

It cannot:

* Freeze vaults.
* Release escrow.
* Reverse completed trades.
* Cancel arbitration.
* Confiscate assets.
* Suspend wallets.

Those responsibilities belong exclusively to the OpenFiat Vault Program and the Dispute Resolution Protocol.

Instead, the Risk Engine produces recommendations that applications may use to improve marketplace safety.

Examples include:

* Displaying warnings.
* Adjusting advertisement ranking.
* Highlighting elevated operational risk.
* Recommending additional caution.
* Temporarily reducing marketplace visibility according to protocol rules.

Financial custody always remains governed by deterministic smart contracts.

---

## 10.5 Risk Vectors

Rather than maintaining a single numerical score, OpenFiat stores a collection of independent risk vectors.

Each vector measures a different aspect of participant behavior.

Examples include:

### Trading Risk

Measures abnormal marketplace activity.

### Operational Risk

Measures participant responsiveness and availability.

### Financial Risk

Measures unusual payment-related behavior.

### Infrastructure Risk

Measures the health of protocol service providers.

### Network Risk

Measures communication reliability.

### Arbitration Risk

Measures unusual arbitrator participation patterns.

Additional vectors may be introduced through future protocol specifications without changing the overall architecture.

---

## 10.6 Merchant Risk Signals

The protocol continuously evaluates merchant activity.

Examples include:

* Sudden increase in settlement time.
* Rapid increase in cancelled trades.
* Unusual dispute frequency.
* Large deviations from historical trade size.
* Significant liquidity withdrawals.
* Frequent advertisement creation and removal.
* Long periods of unexpected unavailability.
* Significant deviations from prevailing market prices.

Individual events do not necessarily indicate malicious behavior.

Instead, multiple signals collectively influence the merchant's current operational risk.

---

## 10.7 Buyer Risk Signals

Buyer behavior also contributes to marketplace risk.

Examples include:

* Repeated reservation cancellations.
* Excessive payment failures.
* Frequent disputes.
* Suspicious payment timing.
* Repeated submission of invalid evidence.
* Attempts to reserve unusually large numbers of advertisements without settlement.

These signals help merchants evaluate counterparties while preserving the protocol's open participation model.

---

## 10.8 Arbitrator Risk Signals

Arbitrators occupy a position of trust within the ecosystem.

The Risk Engine therefore evaluates arbitration behavior separately.

Examples include:

* Repeated failure to reveal committed votes.
* Inconsistent participation.
* Unusually selective case participation.
* Statistically abnormal voting behavior.
* Evidence access without meaningful participation.
* Excessive inactivity after joining cases.

Persistent abnormalities may reduce arbitrator visibility or eligibility according to governance-defined rules.

---

## 10.9 Infrastructure Risk

Infrastructure providers contribute directly to marketplace reliability.

The Risk Engine evaluates:

### Node Operators

* Network uptime.
* Peer connectivity.
* Advertisement propagation.
* Trade session synchronization.
* Snapshot availability.

### Notification Providers

* Delivery success.
* Delivery latency.
* Service availability.

### Oracle Providers

* Price publication consistency.
* Data freshness.
* Availability.
* Consensus with other oracle providers.

### Snapshot Providers

* Snapshot integrity.
* Download reliability.
* Synchronization success.

These evaluations help maintain a healthy decentralized infrastructure ecosystem.

---

## 10.10 Time-Based Analysis

Risk changes over time.

Some events should lose significance quickly.

Examples include:

* Temporary internet outages.
* Short-lived banking maintenance.
* Brief notification delays.

Other events remain relevant for much longer.

Examples include:

* Proven fraud.
* Repeated payment abuse.
* Coordinated manipulation attempts.
* Persistent infrastructure instability.

Each risk vector therefore applies an appropriate decay model according to the nature of the underlying event.

---

## 10.11 Anti-Manipulation

Participants may attempt to manipulate marketplace signals.

The Risk Engine incorporates safeguards against such behavior.

Examples include:

* Detection of wash trading.
* Repeated interaction between the same small group of wallets.
* Artificial reputation inflation.
* Coordinated dispute creation.
* Advertisement spam.
* Automated reservation abuse.

The objective is not to accuse participants of wrongdoing but to identify statistically unusual behavior requiring additional caution.

---

## 10.12 Risk Responses

The protocol intentionally limits the actions available to the Risk Engine.

Possible responses include:

* Reduced advertisement ranking.
* Temporary reduction in advertisement capacity.
* Increased marketplace warnings.
* Recommendation for additional verification.
* Increased visibility of operational statistics.
* Enhanced monitoring by compliant applications.

No response may alter ownership of assets or interfere with deterministic settlement.

---

## 10.13 Explainability

Every risk assessment must be explainable.

Applications should never present opaque numerical values without context.

Instead, participants should be able to inspect the underlying signals contributing to elevated risk.

For example:

```text
Current Operational Risk

Settlement latency increased
+18

Dispute frequency increased
+9

Large liquidity withdrawal
+11

Advertisement churn
+4
```

This transparency enables informed decision-making and encourages confidence in the protocol.

---

## 10.14 Deterministic Algorithms

The Marketplace Risk Engine deliberately avoids opaque machine learning models.

Every calculation is based upon publicly documented formulas.

Every compliant implementation observing identical protocol events must produce identical results.

Governance may adjust weighting parameters over time.

However, the calculation methodology remains transparent and reproducible.

Artificial intelligence may later assist client applications in explaining risk, but it does not participate in the protocol itself.

---

## 10.15 Client Interpretation

The protocol stores measurable facts rather than user-facing labels.

Applications may interpret these facts differently while remaining protocol compatible.

For example, one application may display:

* Normal
* Elevated Risk
* High Risk
* Critical Risk

Another application may use numerical dashboards or graphical indicators.

Regardless of presentation, the underlying protocol data remains identical.

---

## 10.16 Privacy Considerations

Risk analysis operates primarily on protocol events rather than personal information.

The engine does not require:

* Government identification.
* Personal documents.
* Banking credentials.
* Private communications unrelated to a trade.

Only protocol-visible activity contributes to risk calculations.

Where private dispute evidence influences risk, only the outcome of the dispute—not the evidence itself—is incorporated into future risk analysis.

---

## 10.17 Relationship with the Reputation Engine

The Reputation Engine and Marketplace Risk Engine are complementary.

The Reputation Engine records long-term historical performance.

The Risk Engine evaluates current operational conditions.

Together they provide a balanced view of participant trustworthiness.

Historical excellence cannot permanently hide emerging problems.

Likewise, temporary operational issues do not erase years of honest participation.

This separation produces a more accurate and resilient marketplace.

---

## 10.18 Why the Marketplace Risk Engine Matters

Centralized exchanges often rely upon proprietary fraud detection systems.

Participants rarely understand why advertisements disappear, accounts become restricted, or warnings appear.

OpenFiat adopts a fundamentally different philosophy.

Risk assessment is open.

Algorithms are documented.

Signals are explainable.

Participants remain free to inspect, verify, and independently implement every aspect of the system.

By separating advisory risk assessment from deterministic financial settlement, OpenFiat improves marketplace safety while preserving the protocol's commitment to transparency, decentralization, and user sovereignty.

---

## 10.19 Looking Ahead

Understanding current risk is only one component of building a secure marketplace.

Participants must also establish trusted communication channels while preserving privacy and avoiding unnecessary collection of personal information.

The next chapter introduces the OpenFiat Identity Framework, describing how wallets and communication channels are verified through cryptographic ownership rather than centralized identity databases.
