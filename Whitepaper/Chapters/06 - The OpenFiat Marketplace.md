# Chapter 6 — The OpenFiat Marketplace

## 6.1 Introduction

At its core, OpenFiat is a marketplace.

Unlike traditional cryptocurrency exchanges, OpenFiat does not buy or sell digital assets itself. Instead, it provides a decentralized environment in which independent participants can discover one another, negotiate prices according to predefined rules, and exchange stablecoins for local fiat currency securely.

Every trade on OpenFiat begins with an advertisement.

Advertisements are the foundation of the marketplace.

They communicate a merchant's willingness to buy or sell a particular stablecoin under specific terms, allowing buyers to discover offers that best match their needs.

Rather than storing advertisements on centralized servers, OpenFiat distributes them throughout the decentralized node network using the OpenFiat Network Protocol (OFNP). This ensures that no single organization controls which offers appear or who may participate in the marketplace.

---

# 6.2 Buyers and Merchants

OpenFiat recognizes two primary marketplace participants.

## Buyers

A buyer is any participant who accepts an existing advertisement.

Buyers do not need to publish advertisements before using the protocol.

They simply browse available offers, select one that satisfies their requirements, reserve the trade, and follow the standardized trading process defined by the protocol.

## Merchants

Merchants are participants who continuously provide liquidity to the marketplace.

A merchant may advertise one or more offers to:

* Sell stablecoins for local fiat.
* Buy stablecoins using local fiat.
* Support multiple payment methods.
* Operate across one or more countries.

Merchants compete on several factors:

* Price.
* Reputation.
* Availability.
* Settlement speed.
* Supported payment methods.
* Trade limits.
* Historical reliability.

The marketplace never ranks merchants manually.

Ordering is determined by transparent algorithms implemented consistently across compliant clients.

---

# 6.3 Merchant Registration

Before publishing advertisements, a participant must register as a merchant.

Registration creates a merchant profile within the protocol.

A merchant profile contains:

* Merchant public key.
* Display name.
* Supported countries.
* Supported payment methods.
* Reputation score.
* Availability status.
* Merchant age.
* Trade statistics.
* Current stake.
* Advertisement capacity.

Merchant registration requires staking the native OPEN token.

The required stake serves multiple purposes.

It discourages spam.

It aligns incentives with honest participation.

It provides economic accountability.

It determines the merchant's initial advertisement capacity.

Importantly, staking does not guarantee reputation.

Reputation must always be earned through successful marketplace participation.

---

# 6.4 Advertisements

An advertisement is a publicly signed statement expressing a merchant's willingness to trade.

Each advertisement defines the exact conditions under which the merchant is prepared to transact.

A typical advertisement includes:

* Buy or sell direction.
* Stablecoin.
* Country.
* Currency.
* Supported payment methods.
* Minimum trade size.
* Maximum trade size.
* Pricing method.
* Payment window.
* Merchant requirements.
* Optional instructions.

Every advertisement is digitally signed by the merchant's wallet.

Nodes independently verify signatures before distributing advertisements throughout the network.

Unsigned or modified advertisements are rejected automatically.

---

# 6.5 Fixed and Floating Prices

OpenFiat supports two pricing models.

## Fixed Price

The merchant specifies an exact exchange rate.

For example:

> 1 USDC = 130.50 KES

The price remains unchanged until the merchant updates the advertisement.

Fixed pricing provides predictability but requires manual maintenance during volatile market conditions.

## Floating Price

Rather than specifying an exact price, the merchant defines a pricing formula.

For example:

> Oracle Price + 1.5%

or

> Oracle Price − 0.8%

Floating advertisements automatically update as market prices change.

Reference prices are obtained from registered oracle providers selected through governance.

This approach allows merchants to remain competitive without continuously editing advertisements.

---

# 6.6 Payment Methods

OpenFiat intentionally supports regional payment systems.

Each advertisement may specify one or more accepted payment methods.

Examples include:

* Local bank transfer.
* Mobile money.
* Instant payment networks.
* Cash deposit.
* Domestic payment applications.

The protocol does not attempt to standardize how external payment systems operate.

Instead, it standardizes how payment methods are described and verified within the marketplace.

This flexibility allows OpenFiat to operate globally while respecting regional financial infrastructure.

---

# 6.7 Advertisement Capacity

Publishing advertisements consumes marketplace resources.

To prevent spam while encouraging healthy competition, OpenFiat limits the number of simultaneously active advertisements a merchant may publish.

Advertisement capacity is determined by several measurable factors.

These include:

* Merchant stake.
* Reputation score.
* Merchant age.
* Successful trade history.
* Dispute rate.

As merchants demonstrate consistent reliability, the protocol gradually increases their available advertisement capacity.

This creates an incentive to build long-term trust rather than repeatedly creating new merchant identities.

---

# 6.8 Merchant Availability

Merchants may control whether advertisements are available for new orders.

Three operating states are defined.

## Online

The merchant is actively accepting new trades.

Advertisements remain visible.

New reservations are permitted.

## Offline

Advertisements remain published but are temporarily unavailable for new reservations.

Merchants may enter offline mode manually or automatically if no compatible client or registered notification provider remains connected.

This prevents buyers from reserving trades that cannot be serviced promptly.

## Vacation Mode

Vacation mode suspends all advertisements until manually re-enabled.

Unlike offline mode, vacation mode indicates that the merchant does not expect to return immediately.

Reputation is not negatively affected while vacation mode is active.

---

# 6.9 Automatic Advertisement Refresh

Advertisements are not permanent.

Each advertisement includes an expiration time.

Before expiration, merchants may automatically renew advertisements if all publication requirements remain satisfied.

Automatic renewal verifies:

* Sufficient merchant stake.
* Required listing fee.
* Merchant availability.
* Protocol compatibility.

Advertisements that fail renewal are automatically removed from the marketplace.

This prevents stale listings from accumulating over time.

---

# 6.10 Marketplace Discovery

Buyers discover advertisements through any compatible OpenFiat client.

Search filters may include:

* Country.
* Currency.
* Stablecoin.
* Payment method.
* Minimum amount.
* Maximum amount.
* Merchant reputation.
* Merchant response time.
* Price.

Because every client accesses the same decentralized marketplace, users remain free to select whichever interface best suits their preferences.

No client possesses exclusive advertisements.

---

# 6.11 Marketplace Fairness

OpenFiat deliberately avoids allowing applications or infrastructure providers to manipulate marketplace visibility.

Advertisement ordering follows transparent, deterministic rules.

Relevant factors may include:

* Effective price.
* Merchant reputation.
* Settlement performance.
* Availability.
* Advertisement freshness.

Every compliant implementation should produce substantially similar ordering given identical marketplace data.

This transparency prevents hidden favoritism or paid promotion outside protocol-defined mechanisms.

---

# 6.12 The Marketplace Risk Engine

While OpenFiat remains permissionless, it continuously evaluates marketplace health using objective protocol metrics.

Examples include:

* Sudden increases in disputes.
* Excessive order cancellations.
* Abnormal response times.
* Repeated payment inaccuracies.
* Network abuse.
* Advertisement spam.
* Automated manipulation attempts.

The risk engine does not permanently ban participants automatically.

Instead, it adjusts protocol behavior by reducing advertisement capacity, lowering visibility, increasing staking requirements, or triggering additional verification where appropriate.

All risk calculations are deterministic and publicly documented.

---

# 6.13 Marketplace Economics

Publishing advertisements is not free.

Each active advertisement requires a protocol listing fee denominated in OPEN.

These fees serve several purposes.

They discourage spam.

They encourage merchants to maintain only genuine advertisements.

They contribute to protocol revenue.

They fund infrastructure providers, governance initiatives, and ecosystem development according to the token economic model described later in this document.

---

# 6.14 Why This Marketplace Is Different

Most existing P2P marketplaces are websites.

OpenFiat is a protocol.

Advertisements are not owned by a company.

Merchant profiles are not stored in proprietary databases.

Applications do not require permission to access marketplace information.

Infrastructure providers compete openly.

Users remain free to change clients without losing reputation or marketplace access.

These characteristics transform the marketplace from a commercial platform into shared public infrastructure.

---

# 6.15 Looking Ahead

Publishing advertisements represents only the beginning of a trade.

The next chapter introduces the OpenFiat Trade Protocol (OFTP), describing how buyers reserve advertisements, how escrow is funded, how payment is coordinated, how disputes are handled, and how trades are settled securely without relying on a centralized intermediary.
