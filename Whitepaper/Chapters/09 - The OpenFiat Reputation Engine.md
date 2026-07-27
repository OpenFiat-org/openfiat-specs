# Chapter 9 — The OpenFiat Reputation Engine

## 9.1 Introduction

A decentralized marketplace cannot rely on customer support teams or centralized administrators to determine who is trustworthy.

Instead, trust must emerge naturally from measurable behavior.

OpenFiat achieves this through the Reputation Engine.

Rather than assigning trust based on identity, geography, or manual verification, the protocol continuously evaluates every participant according to objectively observable actions.

Every successful trade strengthens reputation.

Every failed obligation weakens it.

Every dispute contributes additional information.

The result is a transparent, deterministic reputation system that allows honest participants to distinguish themselves over time while making dishonest behavior increasingly expensive.

Unlike traditional rating systems, OpenFiat reputation is not based on simple "five-star reviews."

Instead, it is calculated from protocol events that can be independently verified by any compliant implementation.

---

## 9.2 Design Objectives

The Reputation Engine was designed around several core principles.

### Objective

Reputation should be based on measurable facts rather than opinions.

### Deterministic

Every compliant implementation must calculate identical reputation values from identical protocol events.

### Difficult to Manipulate

Artificially inflating reputation should be economically impractical.

### Role-Specific

Different participant roles should be evaluated using different metrics.

### Recoverable

Participants should be able to improve reputation through consistent honest behavior.

### Transparent

The scoring methodology should be publicly documented.

---

## 9.3 Reputation Is Earned

OpenFiat deliberately separates **identity** from **reputation**.

Identity answers:

> "Who controls this wallet or communication channel?"

Reputation answers:

> "How has this participant behaved over time?"

This distinction is fundamental.

An anonymous merchant who has completed 25,000 successful trades is often more trustworthy than a newly created verified account with no trading history.

The protocol therefore rewards proven behavior rather than claimed identity.

---

## 9.4 Participant Categories

OpenFiat maintains independent reputation profiles for different participant roles.

These include:

* Buyers
* Merchants
* Arbitrators
* Node Operators
* Notification Providers
* Oracle Providers
* Snapshot Providers

A participant may hold multiple roles simultaneously.

Each role accumulates reputation independently.

For example, an excellent merchant is not automatically considered an excellent arbitrator.

Likewise, an outstanding node operator does not automatically receive marketplace advantages as a trader.

---

## 9.5 Merchant Reputation

Merchant reputation is the most visible reputation score within the marketplace.

It influences:

* Advertisement ranking.
* Advertisement capacity.
* Maximum trade limits.
* Merchant tier progression.
* Discovery priority.

Merchant reputation is calculated using several measurable indicators.

### Settlement Speed

How quickly the merchant completes trades after receiving payment.

### Trade Success Rate

The percentage of successfully completed trades.

### Dispute Rate

The percentage of completed trades that enter arbitration.

### Trade Volume

The cumulative value of completed trades.

### Average Trade Size

Larger trades generally provide stronger evidence of reliability than numerous very small trades.

### Merchant Age

Long-term participation demonstrates consistency.

### Availability

The percentage of time the merchant remains reachable through at least one compatible client or registered notification provider.

### Payment Accuracy

The frequency with which payments are correctly acknowledged without dispute.

---

## 9.6 Buyer Reputation

Buyers also accumulate reputation.

Metrics include:

* Successful trade completion.
* Payment punctuality.
* Dispute frequency.
* Cancellation rate.
* Payment accuracy.
* Evidence quality during disputes.
* Long-term participation.

Although buyer reputation does not influence advertisement publication, it helps merchants evaluate counterparties before accepting large transactions.

---

## 9.7 Arbitrator Reputation

Arbitration directly affects protocol trust.

Consequently, arbitrators accumulate their own specialized reputation.

Metrics include:

* Cases participated in.
* Majority alignment rate.
* Evidence review quality.
* Voting consistency.
* Participation reliability.
* Response time.

Repeatedly joining cases without voting or demonstrating poor participation reduces arbitrator reputation.

High-performing arbitrators become preferred participants for future disputes.

---

## 9.8 Node Reputation

OpenFiat nodes provide the communication infrastructure that powers the protocol.

Node reputation encourages reliable network participation while discouraging poor-quality infrastructure.

Metrics include:

### Uptime

How consistently the node remains online.

### Peer Reliability

Successful communication with other nodes.

### Synchronization Quality

Ability to remain current with network state.

### Advertisement Propagation

Successful forwarding of marketplace information.

### Session Availability

Reliable participation in active trade synchronization.

### Snapshot Quality

Where applicable, successful distribution of verified snapshots.

### Protocol Compliance

Consistent adherence to protocol rules and supported versions.

Nodes that repeatedly fall below acceptable performance thresholds may receive reduced network priority or become ineligible for certain reward distributions.

---

## 9.9 Service Provider Reputation

Notification providers, oracle providers, and snapshot providers are evaluated independently.

Typical metrics include:

Notification Providers

* Delivery success rate.
* Delivery latency.
* Availability.
* User satisfaction signals.

Oracle Providers

* Price consistency.
* Publication frequency.
* Availability.
* Historical accuracy.

Snapshot Providers

* Snapshot integrity.
* Download reliability.
* Availability.
* Synchronization success.

Service providers compete on measurable quality rather than exclusive partnerships.

---

## 9.10 Reputation Decay

The marketplace changes over time.

A participant who earned an excellent reputation years ago but becomes inactive should not permanently dominate rankings.

For this reason, OpenFiat introduces gradual reputation decay.

Decay is intentionally conservative.

Long-term honest behavior remains valuable, but recent activity carries greater weight than distant history.

This encourages ongoing participation while preventing abandoned accounts from retaining disproportionate influence indefinitely.

---

## 9.11 Reputation Recovery

OpenFiat does not permanently punish participants for isolated mistakes.

Instead, reputation is designed to recover through sustained honest behavior.

For example:

A merchant who experiences several disputes due to temporary banking problems may gradually rebuild trust by completing future trades successfully.

Likewise, a node operator who resolves infrastructure issues can recover reputation through consistent uptime and reliable performance.

The protocol rewards improvement rather than permanent exclusion.

---

## 9.12 Anti-Manipulation Measures

Reputation systems are valuable only if manipulation remains difficult.

OpenFiat incorporates multiple safeguards.

### Stake Requirements

Meaningful participation requires economic commitment.

### Trade Value Weighting

Higher-value legitimate trades contribute more information than numerous trivial trades.

### Self-Trading Detection

The protocol monitors for suspicious trading patterns intended solely to inflate reputation.

### Network Analysis

Repeated interactions between the same small group of participants receive reduced influence where appropriate.

### Dispute Analysis

Frequent coordinated disputes may trigger additional risk evaluation.

The exact algorithms are deterministic and fully documented within the Protocol Specification.

---

## 9.13 Reputation and the Risk Engine

The Reputation Engine and Marketplace Risk Engine complement one another.

Reputation measures long-term trustworthiness.

The Risk Engine evaluates current behavior.

For example:

A merchant with years of excellent reputation who suddenly experiences a wave of failed trades may temporarily receive reduced marketplace visibility while the Risk Engine evaluates abnormal behavior.

Once normal operation resumes, visibility returns automatically.

This approach balances historical trust with real-time marketplace protection.

---

## 9.14 Reputation Is Never Purchased

OpenFiat deliberately prohibits purchasing reputation.

Increasing stake may expand advertisement capacity.

Holding additional OPEN may increase governance participation.

Operating infrastructure may generate protocol rewards.

However, none of these actions directly increase reputation.

Only observable behavior may improve trust.

This distinction preserves the integrity of the marketplace.

---

## 9.15 Reputation Transparency

Every participant may inspect the components contributing to reputation.

Rather than presenting only a single score, compliant clients should display meaningful metrics, including:

* Total completed trades.
* Trade success percentage.
* Average settlement time.
* Dispute rate.
* Merchant age.
* Trade volume.
* Average trade size.
* Availability.
* Current reputation tier.

Providing underlying metrics allows users to make informed decisions rather than relying solely on an abstract numerical score.

---

## 9.16 Reputation Tiers

To simplify the marketplace experience, OpenFiat groups participants into reputation tiers.

Example tiers may include:

* New
* Established
* Trusted
* Professional
* Elite

These tiers are descriptive rather than subjective.

They are earned automatically through protocol-defined metrics.

Governance may refine tier thresholds over time without changing the underlying reputation architecture.

---

## 9.17 Reputation and Rewards

Although reputation is not directly tradable, it influences participation throughout the ecosystem.

Examples include:

* Increased advertisement capacity.
* Higher marketplace visibility.
* Eligibility for larger trade limits.
* Eligibility for specialized merchant categories.
* Preferred arbitrator selection.
* Enhanced node responsibilities.
* Improved service provider visibility.

Reputation therefore becomes a valuable protocol asset earned through consistent contribution rather than financial expenditure.

---

## 9.18 Why the Reputation Engine Matters

OpenFiat replaces institutional trust with measurable behavior.

Instead of asking users to trust a company, the protocol enables them to evaluate years of observable performance recorded through cryptographically verifiable protocol events.

This creates a marketplace where trust grows organically, manipulation becomes economically costly, and long-term honest participation is consistently rewarded.

---

## 9.19 Looking Ahead

Trust extends beyond trading behavior alone.

Participants must also establish secure methods of communication and verify control of the channels through which they receive trade notifications and protocol messages.

The next chapter introduces the OpenFiat Identity Framework, explaining how participants verify ownership of communication channels while preserving privacy and avoiding mandatory identity verification.
