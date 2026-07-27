# OFS-2100 — Advertisement Protocol (OAP)

**Document ID:** OFS-2100

**Title:** OpenFiat Advertisement Protocol

**Version:** 1.0.0 (Draft)

**Status:** Draft

**Category:** Trading

**Depends On:** OFS-1000, OFS-1200, OFS-1500, OFS-2000, OFS-7000

---

## Abstract

The OpenFiat Advertisement Protocol (OAP) defines how merchants create, publish, update, synchronize, prioritize, and remove trading advertisements across the OpenFiat Network.

Advertisements represent publicly discoverable offers to buy or sell stablecoins using supported fiat payment methods.

Unlike traditional exchanges, advertisements are **not** owned by a centralized marketplace.

They exist as decentralized protocol objects synchronized across the OpenFiat network through the Gossip Protocol.

---

## 1. Introduction

Advertisements are the foundation of the OpenFiat marketplace.

Every trade begins with an advertisement.

Merchants publish advertisements describing:

* The asset
* Trade direction
* Pricing model
* Limits
* Payment methods
* Availability
* Settlement preferences

Applications display advertisements locally after synchronizing them from the decentralized network.

No central order book exists.

---

## 2. Scope

This specification defines:

* Advertisement creation
* Advertisement lifecycle
* Merchant limits
* Pricing models
* Stake-based capacity
* Availability
* Vacation mode
* Offline behavior
* Payment methods
* Advertisement synchronization
* Automatic updates
* Advertisement expiration

This specification does **not** define:

* Reservation
* Settlement
* Reputation
* Disputes

---

## 3. Design Goals

The Advertisement Protocol SHALL:

* Be fully decentralized.
* Support millions of advertisements.
* Prevent duplicate advertisements.
* Support automatic inventory updates.
* Support dynamic pricing.
* Support multiple payment methods.
* Synchronize efficiently.

---

## 4. Advertisement Types

OpenFiat supports two advertisement directions.

### Sell Advertisement

Merchant offers stablecoins.

Example:

> Selling 10,000 USDC for Kenyan Shillings.

---

### Buy Advertisement

Merchant wishes to purchase stablecoins.

Example:

> Buying up to KES 1,000,000 worth of USDC.

Both advertisement types follow the same lifecycle.

---

## 5. Advertisement Identifier

Every advertisement SHALL possess a globally unique Advertisement ID.

The Advertisement ID remains constant until the advertisement is permanently removed.

---

## 6. Advertisement Structure

Every advertisement SHALL contain:

* Advertisement ID
* Merchant ID
* Wallet Address
* Asset
* Trade Direction
* Fiat Currency
* Minimum Trade
* Maximum Trade
* Available Liquidity
* Pricing Model
* Payment Methods
* Merchant Tier
* Advertisement Status
* Creation Time
* Last Updated
* Digital Signature

---

## 7. Merchant Capacity

The number of simultaneously active advertisements is determined by the merchant's protocol capacity.

Capacity is influenced by:

* Merchant reputation
* Merchant age
* Successful trading history
* Staked OPEN tokens

Higher-quality merchants may operate more advertisements simultaneously.

The exact scoring formula is defined by the Reputation Engine.

---

## 8. Advertisement Visibility

Advertisements become visible only after:

* Successful validation
* Digital signature verification
* Gossip synchronization

Applications SHALL ignore advertisements that fail validation.

---

## 9. Liquidity

Every advertisement publishes available liquidity.

For Sell advertisements:

Available liquidity equals the unreserved balance already deposited in the merchant's Liquidity Vault — never a promise or an off-chain wallet balance.

For Buy advertisements:

Available liquidity equals the merchant's declared purchasing capacity expressed in fiat.

Liquidity automatically changes as trades occur.

---

## 10. Automatic Inventory Management

Inventory management is handled entirely by the protocol.

### Selling Stablecoins

Liquidity Vault (program-owned; see OFS-2300 §6)

10,000 USDC

↓

Buyer reserves

2,500 USDC

↓

Program automatically locks

2,500 USDC

↓

Advertisement updates

7,500 USDC available

No new deposit occurs at reservation time — the inventory was already deposited into the vault before the advertisement became visible (§8). Reservation only marks a portion of the existing vault balance as unavailable.

---

### Buying Stablecoins

Merchant advertises:

Buying up to KES 500,000

↓

Seller accepts

↓

Seller deposits escrow

↓

Advertisement purchasing capacity decreases

Everything occurs automatically.

The merchant never manually edits available inventory after every trade.

---

## 11. Pricing Models

OpenFiat supports two pricing models.

### Fixed Pricing

Example:

1 USDC = KES 129

Price remains constant until updated.

---

### Floating Pricing

Price is calculated dynamically using trusted Oracle Providers.

Example:

Oracle Mid Price

*

Merchant Premium

=

Final Trade Price

Floating pricing automatically updates as market prices move.

---

## 12. Oracle Integration

Floating advertisements rely on OFS-7000.

Merchants specify:

* Oracle Provider
* Premium
* Discount
* Refresh Frequency

Applications calculate the current price locally using the latest verified oracle data.

---

## 13. Payment Methods

Every advertisement may support multiple payment methods.

Examples:

* Bank Transfer
* Mobile Money
* Cash Deposit
* RTGS
* ACH
* PIX
* SEPA
* Faster Payments
* Local Instant Payment Systems

Payment methods are independently selectable by the merchant.

---

## 14. Merchant Availability

Merchants publish current availability.

States include:

* Online
* Busy
* Away
* Offline
* Vacation

Availability updates propagate immediately through the Gossip Protocol.

---

## 15. Offline Mode

OpenFiat supports offline merchants.

A merchant is considered available if:

* At least one authenticated UI session remains active,

**or**

* A registered Notification Provider is actively monitoring trades on behalf of the merchant.

This enables merchants to receive trade requests even when no trading interface is open.

If neither condition is met, advertisements automatically transition to Offline.

---

## 16. Vacation Mode

Vacation Mode allows merchants to temporarily suspend trading without deleting advertisements.

When enabled:

* Advertisements remain synchronized.
* Advertisements disappear from active search results.
* Existing reputation remains unchanged.
* Historical statistics remain intact.

Vacation Mode may be enabled or disabled at any time.

---

## 17. Advertisement Refresh

Advertisements automatically refresh whenever:

* Inventory changes
* Price changes
* Availability changes
* Payment methods change
* Reputation tier changes

Manual refresh is not required.

---

## 18. Automatic Disable

Advertisements SHALL automatically become inactive when:

* Available liquidity reaches zero.
* Merchant loses required permissions.
* Merchant session becomes unavailable.
* Escrow requirements cannot be satisfied.
* Merchant manually disables trading.

Applications MUST stop displaying inactive advertisements.

---

## 19. Merchant Tiers

Merchant tiers are determined using a hybrid scoring model.

Inputs include:

* Reputation Score
* Merchant Age
* Settlement Success
* Availability
* Historical Volume
* Staked OPEN Tokens

Higher tiers MAY receive:

* More simultaneous advertisements
* Larger trading limits
* Greater marketplace visibility
* Additional protocol capabilities

Merchant tiers never bypass protocol rules.

---

## 20. Advertisement Priority

Search ordering MAY consider:

* Reputation
* Price competitiveness
* Geographic proximity
* Response speed
* Settlement speed
* Merchant tier

Applications remain free to implement custom sorting.

The protocol does not mandate a single ranking algorithm.

---

## 21. Advertisement Expiration

Advertisements may expire automatically due to:

* Merchant inactivity
* Long-term offline status
* Unsupported protocol version
* Invalid signatures
* Manual expiration

Expired advertisements are removed through the Gossip Protocol.

---

## 22. Marketplace Risk Integration

Before an advertisement becomes visible, participating applications SHOULD evaluate merchant risk using the Marketplace Risk Engine.

Risk signals MAY include:

* Excessive dispute history
* Unusual pricing behavior
* Linked fraudulent wallets
* High cancellation rates
* Suspicious trading patterns
* Regulatory risk indicators

Applications MAY warn users or reduce advertisement visibility for elevated-risk merchants.

The protocol itself remains permissionless—risk evaluation influences presentation, not protocol validity.

---

## 23. Advertisement Synchronization

Advertisement changes generate immutable protocol events.

Examples include:

* Advertisement Created
* Advertisement Updated
* Advertisement Disabled
* Advertisement Deleted
* Availability Changed
* Inventory Updated
* Price Updated

All changes propagate through OFS-1200.

---

## 24. Security Considerations

Implementations MUST reject:

* Duplicate Advertisement IDs
* Invalid signatures
* Negative liquidity
* Invalid pricing models
* Unsupported assets
* Malformed payment methods
* Unauthorized advertisement updates

Every advertisement MUST be cryptographically authenticated.

---

## 25. Conformance

A compliant implementation MUST:

* Support Buy and Sell advertisements.
* Support fixed and floating pricing.
* Integrate with Oracle Providers for floating prices.
* Support automatic inventory updates.
* Support multiple payment methods.
* Support Offline Mode.
* Support Vacation Mode.
* Support automatic advertisement disabling.
* Generate signed advertisement events.
* Synchronize advertisements through the Gossip Protocol.

---

## 26. Relationship to Other Specifications

The Advertisement Protocol represents the public entry point into the OpenFiat marketplace.

```text id="advertisement-architecture"
              OFS-2000
        OpenFiat Trade Protocol
                    │
                    ▼
              OFS-2100
      Advertisement Protocol
                    │
      ┌─────────────┼─────────────┐
      ▼             ▼             ▼
OFS-2200      OFS-3000      OFS-7000
Reservation   Reputation     Oracle
                    │
                    ▼
            Marketplace Search
```

The Advertisement Protocol answers one fundamental question:

**"How does a merchant publicly offer liquidity in a decentralized marketplace?"**

By treating advertisements as signed, synchronized protocol objects rather than centralized database records, OpenFiat enables any compliant application to display the same global marketplace while preserving decentralization, automatic inventory management, and deterministic behavior across the entire network.
