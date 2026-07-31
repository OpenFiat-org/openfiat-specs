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
* Asset Mint (see §6.1)
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

### 6.1 An advertisement names a mint, not a ticker

| Parameter | Value | Status |
|---|---|---|
| Asset field on an advertisement | A base58-encoded 32-byte Solana mint address | [CONFIRMED — OFS-4100 §4.2] |
| Asset ticker on an advertisement | Not carried. The record has no symbol field at all | [CONFIRMED — OFS-4100 §4.2] |
| Displayed symbol | Resolved from the mint at the presentation edge | [CONFIRMED — OFS-4100 §4.2] |

The asset field was previously free text the merchant chose — an advertisement said `USDC` and the buyer read `USDC`. Nothing connected that string to the token the escrow would actually move. A merchant could advertise one asset and settle in another, and every layer would agree the trade had completed correctly, because each layer did exactly what it was asked: the escrow moved the mint it was handed, the settlement recorded the amount it was given, and the label was only ever a label.

**A ticker is a label; a mint address is an identity.** A ticker is additionally cluster-dependent — `USDC` names one mint on mainnet and a different one on devnet — and it is spoofable by construction, because it is a string somebody else filled in. An address is neither.

So the label does not travel. An advertisement carries the mint, and the symbol a buyer sees is derived from that mint by the application showing it. A merchant never supplies the name of the thing they are selling.

**A mint with no known name is displayed as its address.** Symbol resolution is a lookup that may fail, and its failure is not an error: the honest answer for an address nobody has named is the address. An implementation MUST NOT substitute a guess, and MUST NOT fall back to any name the merchant supplied — there is no such field to fall back to.

**Validation is well-formedness, not membership.** A node MUST verify that the asset mint is a real address: base58, decoding to exactly 32 bytes. A node MUST NOT reject an advertisement because its mint is absent from any list the node itself compiles in — see §24.1.

OFS-4100 §4.2 records the other half of this defence, the governance-updatable on-chain settlement allowlist, and states that until an advertisement carries its mint the allowlist bounds which tokens can move but not which one a taker believes they are buying. This section is that other half.

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
* Price Decimals

Applications calculate the current price locally using the latest verified oracle data.

### 12.0 Two fields this list used to name, and why they are gone

**Discount.** The Premium is **signed**. A merchant quoting below mid — a
Buy advertisement competing for flow — expresses that as a negative
Premium, and `-10 000` bps is exactly zero. A separate Discount field would
be a second way to say the same thing, and two ways to say one thing can
contradict: an advertisement carrying Premium `+100` *and* Discount `50`
has no defined price, and every implementation would have to invent one.

**Refresh Frequency.** A merchant does not control how often the price is
recomputed, so a merchant-declared frequency would be a claim they are not
in a position to make. The price is resolved **at read time**, from the
freshest oracle record the answering node holds, and the record carries its
own expiry. What actually bounds staleness is the `mid_expires_at` on the
resolved quote (§12.1), which tells a reader how long the number they were
given remains good — a property of the oracle data, not a merchant setting.

**Price Decimals** was missing from this list and is a real merchant-set
field. See §12.1 for why the precision is declared per advertisement rather
than inferred from the currency.

---

### 12.1 The floating formula, and the two fields it needs

`[PROPOSED — NEEDS SIGN-OFF]`

| Parameter | Value |
|---|---|
| Premium | Signed basis points over the oracle mid. A discount is a negative premium, not a separate field |
| Premium floor | −10,000 bps is exactly zero. Below that is a negative price, and resolves to no price rather than clamping to zero |
| Price precision | Declared by the merchant, in decimal places of the **fiat** currency (2 for KES/NGN/USD, 0 for JPY) |
| Rounding | Half-to-even, at the declared precision |

`price = mid × (1 + premium_bps ÷ 10,000)`, rounded half-to-even to the declared precision.

**Why the premium is signed rather than paired with a discount field.** A merchant competing for flow may legitimately quote below mid. Two fields for one number invite an advertisement that sets both, and then every implementation has to decide which wins.

**Why the precision is declared rather than inferred.** Nothing else on the record carries it: the trade limits and the available liquidity are denominated in the *asset*, and a floating advertisement has no fixed price to borrow the precision from. Inferring it from the fiat currency code would mean a currency table inside the protocol, silently mis-rounding every currency missing from it.

**Why half-to-even.** The last minor unit has to go to somebody, and whoever it goes to gets it on *every* trade — a merchant is a repeat player, so a systematic half-cent is a real transfer even where any single instance is negligible. Rounding toward the merchant is a small theft from every taker; rounding toward the taker is the same transfer pointed the other way, and merchants would widen the premium to recover it, moving the cost back onto takers where it is less visible than a premium they can read. Half-to-even has no directional drift and its worst case is half a minor unit rather than a whole one. It also does not depend on the advertisement's direction, so the price shown to a taker and the price quoted to the merchant are the same number.

**A resolved price is never written onto the advertisement.** It is computed at the instant somebody asks, from an oracle read they made. A price refreshed onto a gossip-replicated record would be stale between refreshes and different on every node, because each node would refresh on its own clock from its own oracle view. This is why the record carries a formula and no refresh interval: there is nothing being refreshed.

**Crossing from a mint to a symbol.** An oracle publishes a rate against a symbol (`USDC/KES`) because a rate is a fact about an asset rather than about one cluster's mint of it, while an advertisement names a mint (§6.1). Pricing a floating advertisement is the one place the two must meet, and an implementation SHOULD perform that crossing in a single place rather than by string comparison scattered through the pricing path. A mint the implementation has no name for is simply **unpriceable**: no oracle publishes a pair it cannot name, and inventing one would be inventing a rate.

**No price is an answer.** A floating advertisement whose feed has lapsed, or whose pair nobody publishes, has no price — not a last-known price, not zero, not the mid alone. An implementation MUST distinguish the two reasons when it reports this (OFS-7000 §12), because a lapsed feed will likely return and an unpublished corridor will not, and MUST NOT present either as a number.

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

### 20.1 Narrowing and paging the order book

`[PROPOSED — NEEDS SIGN-OFF]`

| Parameter | Value |
|---|---|
| Filters | Asset mint, fiat currency, direction, payment method, trade amount, status |
| Default status filter | Active only |
| Page size — default | 25 |
| Page size — maximum | 100 |
| Resume mechanism | A cursor naming the last row seen. Never a numeric offset |
| Ordering | By Advertisement ID, which is unique and immutable |

A listing read that takes no parameters and returns every advertisement on the network works at pilot volume and fails in both directions at any real one: the response grows without bound, and a buyer cannot find the offer they want. Filtering in the client moves the second problem and leaves the first — the node still serializes the whole book on every request, and every client still downloads it to show a page of it. So the narrowing belongs at the node.

**A cursor, not an offset.** Advertisements are published continuously. Between a reader's first page and their second, a new one can sort ahead of both, and "skip 20, take 20" then returns rows 20–39 of a list whose contents have shifted underneath it — so one advertisement is shown twice and another is never shown at all. A reader scrolling an order book has no way to notice. A cursor names the last row actually seen and asks for what comes after it, under a total order that does not change when a row is inserted. That is stable by construction rather than by luck, and it costs nothing: the identifier is already in the response.

**The cursor MUST be returned beside the rows**, not left for the caller to derive from the last one. A caller deriving it has to know the ordering, and an ordering the two sides disagree about is exactly how a page gets skipped.

**A deleted cursor row MUST NOT reset the reader.** Resumption is "strictly after this identifier", which is well-defined whether or not that row still exists — an advertisement can be removed between two pages, and a reader must not be thrown back to the beginning because the row they last saw is gone.

**The page size ceiling is not advisory.** Without one, the page size is chosen by whoever is calling, which makes "return everything" available again under a different name.

**The default status filter is Active.** A disabled, paused or deleted advertisement cannot be traded against, so returning one by default would be offering something that is not on offer. Asking for another status explicitly is a merchant reviewing their own book, which is a different question.

**An empty filter is the whole book, one page at a time.** The unparameterised call keeps working and only its size changes. The response shape does not: it carries the rows *and* the cursor, so a caller that previously read a bare array has to be updated. That is a breaking change to this domain's read method, and it is recorded here rather than in OFS-8200, which versions the envelope rather than individual methods (OFS-8200 §12).

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

### 24.1 "Unsupported assets" means malformed, not unlisted

A node MUST reject an advertisement whose asset mint is not a well-formed address (§6.1). A node MUST NOT reject one because the mint is absent from any list that node compiles in.

The settlement-mint allowlist is on chain and governance-updatable (OFS-4100 §4.2). A node built last month enforcing its own copy of that list would refuse an advertisement naming a mint governance approved last week — it would be enforcing a stale copy of a rule it is not the authority for, and two honest nodes on different releases would disagree about which advertisements are valid. That disagreement is a network partition dressed as validation.

Enforcement belongs where the funds move: a mint that is not allowlisted cannot back a liquidity vault, cannot be reserved against, and cannot fund a trade escrow, and all three refusals are made by the on-chain program regardless of what any node believed. A node's job at this layer is to make the field unambiguous, not to adjudicate it.

The symbol table an implementation uses to display a name (§6.1) is a **display** table for exactly this reason. It is not an allowlist, it enforces nothing, and a mint missing from it is an address with no nickname rather than an invalid advertisement.

---

## 25. Conformance

A compliant implementation MUST:

* Support Buy and Sell advertisements.
* Identify an advertisement's asset by mint address, never by a merchant-supplied ticker (§6.1).
* Resolve a displayed symbol from the mint, and show the address where it knows no name.
* Accept an advertisement naming a well-formed mint it does not itself recognize (§24.1).
* Support fixed and floating pricing.
* Return no price, with its reason, for a floating advertisement whose oracle read is stale or absent (§12.1).
* Serve advertisement listings filtered and paged, resuming by cursor rather than by offset (§20.1).
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
