# Chapter 8 — The OpenFiat Liquidity Vault Architecture

## 8.1 Introduction

The most fundamental question in any peer-to-peer marketplace is:

> **How can two strangers exchange value without trusting one another?**

In traditional P2P cryptocurrency exchanges, the answer is escrow.

A centralized platform temporarily takes custody of digital assets until the trade has been completed.

While effective, this model requires users to trust the exchange operator.

OpenFiat replaces institutional trust with cryptographic guarantees.

Rather than depositing assets into accounts controlled by a company, participants interact directly with audited Solana smart contracts that enforce deterministic settlement rules.

However, OpenFiat goes one step further.

Instead of treating every trade as an independent escrow deposit, OpenFiat introduces **Liquidity Vaults**.

Liquidity Vaults allow merchants to maintain continuously available trading inventory, enabling multiple concurrent trades without repeatedly depositing funds for every order.

This architecture reduces latency, eliminates fake liquidity, improves capital efficiency, and creates a marketplace that behaves more like a professional exchange than a traditional escrow platform.

---

## 8.2 Design Objectives

The Liquidity Vault Architecture was designed around several primary objectives.

### Guaranteed Liquidity

Advertisements must represent real, verifiable assets.

### Instant Reservations

Once an order is accepted, funds are immediately reserved without requiring additional merchant action.

### Capital Efficiency

A single liquidity deposit should support many trades.

### Non-Custodial

Assets remain controlled exclusively by audited protocol smart contracts.

### Deterministic

Every compliant implementation produces identical vault behavior.

### Transparent

Every vault balance can be independently verified on Solana.

---

## 8.3 Two Types of Vaults

OpenFiat intentionally uses two different vault models depending on the direction of the trade.

### Liquidity Vault

Used when a merchant is **selling** stablecoins.

The merchant deposits inventory before advertisements become active.

The deposited balance may be reserved across multiple trades.

### Trade Escrow Vault

Used when a merchant is **buying** stablecoins.

The participant selling stablecoins deposits funds only after a reservation has been accepted.

This distinction minimizes unnecessary capital lock-up while guaranteeing that sell-side liquidity is always genuine.

---

## 8.4 Liquidity Vaults

A Liquidity Vault is a persistent smart contract account owned by the OpenFiat Escrow Program.

The vault contains stablecoins that a merchant has committed to selling.

Unlike traditional escrow, the vault is **not** created for an individual trade.

Instead, it continuously serves one or more advertisements until the merchant withdraws unused inventory or closes the advertisement.

A merchant may maintain multiple Liquidity Vaults for different stablecoins.

For example:

* USDC Vault
* USDT Vault
* EURC Vault

Each vault tracks:

* Total deposited balance.
* Available balance.
* Reserved balance.
* Settled balance.
* Pending settlements.

This allows multiple buyers to reserve liquidity simultaneously without conflict.

---

## 8.5 Publishing a Sell Advertisement

Before a merchant can publish a sell advertisement, sufficient stablecoins must already exist within the corresponding Liquidity Vault.

Example:

Merchant deposits:

> 10,000 USDC

The protocol records:

```text
Liquidity Vault

Total Balance:      10,000 USDC
Reserved:                0 USDC
Available:          10,000 USDC
```

Only after the deposit has been confirmed may the advertisement become visible on the marketplace.

This guarantees that every published sell advertisement represents real, immediately available liquidity.

Advertisements can never exceed the vault's available balance.

---

## 8.6 Reservation

Suppose a buyer wishes to purchase:

> 2,000 USDC

The reservation succeeds.

The protocol immediately updates the vault.

```text
Liquidity Vault

Total Balance:      10,000 USDC
Reserved:            2,000 USDC
Available:           8,000 USDC
```

No additional transfer is required.

No merchant confirmation is required.

No manual escrow funding occurs.

The liquidity already exists.

The reservation simply marks a portion of the vault as unavailable for other buyers.

This process occurs atomically through the OpenFiat Escrow Program.

---

## 8.7 Settlement

Once the buyer has completed fiat payment and the merchant confirms receipt, the Escrow Program transfers the reserved stablecoins directly from the Liquidity Vault to the buyer.

The vault is updated automatically.

```text
Liquidity Vault

Total Balance:       8,000 USDC
Reserved:                0 USDC
Available:           8,000 USDC
```

The merchant may continue accepting additional reservations without interruption.

One deposit supports many independent trades.

---

## 8.8 Adding Liquidity

Merchants may increase available inventory at any time.

Example:

Current available balance:

> 8,000 USDC

Merchant deposits:

> 5,000 USDC

Updated vault:

```text
Total Balance:      13,000 USDC
Reserved:                0 USDC
Available:          13,000 USDC
```

No advertisements need to be recreated.

The additional liquidity becomes available immediately after confirmation.

---

## 8.9 Withdrawing Liquidity

Merchants may withdraw **only unreserved funds**.

For example:

```text
Available: 8,000 USDC
Reserved: 2,000 USDC
```

The merchant may withdraw up to:

> 8,000 USDC

The reserved balance remains locked until the associated trades complete or are cancelled.

This guarantees that existing buyers are never affected by subsequent merchant withdrawals.

---

## 8.10 Buy Advertisements

Buying advertisements operate differently.

In this scenario, the merchant wishes to purchase stablecoins using local fiat.

Example advertisement:

> Buying USDC

Trade range:

> 150 KES — 1,000,000 KES

Because the merchant is purchasing cryptocurrency rather than selling it, there is no existing stablecoin inventory to lock.

Instead, escrow is funded by the participant selling the stablecoins.

The protocol therefore creates a temporary Trade Escrow Vault after reservation.

The flow becomes:

```text
Seller accepts advertisement

↓

Reservation succeeds

↓

Seller deposits USDC

↓

Trade Escrow Vault

↓

Merchant pays fiat

↓

Settlement

↓

USDC released to merchant
```

This ensures that the digital asset is always secured before fiat settlement begins, regardless of trade direction.

---

## 8.11 Program-Derived Authority

Every Liquidity Vault and Trade Escrow Vault is controlled by a Solana Program Derived Address (PDA).

No private key exists for these vaults.

As a result:

* Merchants cannot bypass protocol rules.
* Buyers cannot withdraw funds.
* Node operators cannot access assets.
* AllenHark cannot access assets.
* Arbitrators cannot access assets.

Only the OpenFiat Escrow Program may authorize transfers according to the protocol state machine.

---

## 8.12 Vault State Machine

Every reservation follows a deterministic lifecycle.

```text
Available

↓

Reserved

↓

Awaiting Fiat Settlement

↓

Released
     │
     │
     └───────────────┐
                     │
                 Cancelled
                     │
                     │
               Returned to Available
```

Each transition is validated by the Escrow Program.

No implementation may skip intermediate states.

---

## 8.13 Timeouts

Funds should never remain reserved indefinitely.

The protocol defines deterministic timeout rules for situations such as:

* Buyer inactivity.
* Merchant inactivity.
* Escrow funding failures.
* Abandoned trades.
* Arbitration delays.

When a timeout occurs, reserved liquidity is either released to the buyer, returned to the available balance, or held pending dispute resolution according to the protocol rules.

---

## 8.14 Fee Distribution

Whenever settlement occurs, protocol fees are distributed automatically.

Potential recipients include:

* Protocol treasury.
* Node reward pool.
* Notification provider.
* Oracle provider.
* Governance treasury.

Distribution is performed entirely by the Escrow Program.

No manual accounting is required.

---

## 8.15 Security Guarantees

The Liquidity Vault Architecture provides several important guarantees.

### Real Liquidity

Sell advertisements cannot exceed deposited inventory.

### Automatic Reservation

Accepted trades immediately reserve funds.

### No Manual Escrow

Merchants do not transfer assets for every trade.

### Concurrent Trading

One vault supports many simultaneous orders.

### Non-Custodial

Only audited smart contracts control assets.

### Transparent

Every balance is publicly verifiable.

---

## 8.16 Why This Architecture Is Different

Most existing peer-to-peer exchanges create escrow only after an order has been accepted.

OpenFiat treats sell-side liquidity as persistent inventory.

This approach resembles professional financial markets more closely than traditional P2P platforms.

It provides several advantages:

* Buyers know liquidity is genuine.
* Merchants deposit funds only once.
* Reservations complete instantly.
* Multiple trades can occur simultaneously.
* On-chain transactions are reduced.
* User experience improves significantly.

For buy advertisements, OpenFiat retains the familiar escrow model because the seller supplies the digital asset.

By using two complementary vault models, the protocol achieves both efficiency and security without introducing unnecessary complexity.

---

## 8.17 Looking Ahead

Guaranteeing the availability of funds is only one aspect of creating a trustworthy marketplace.

The protocol must also determine which participants consistently behave honestly over time.

The next chapter introduces the OpenFiat Reputation Engine, explaining how merchants, buyers, arbitrators, node operators, and service providers earn trust through measurable behavior rather than centralized approval.
