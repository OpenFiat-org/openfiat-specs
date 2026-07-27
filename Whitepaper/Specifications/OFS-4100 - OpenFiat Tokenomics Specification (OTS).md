# OFS-4100 — OpenFiat Tokenomics Specification (OTS)

**Document ID:** OFS-4100

**Title:** OpenFiat Tokenomics Specification

**Version:** 0.1.0 (**DRAFT — NOT YET SIGNED OFF**)

**Status:** Draft

**Category:** Economics

**Depends On:** OFS-1600, OFS-2300, OFS-2400, OFS-4000

---

# Status Banner

**This document is a DRAFT.** Every numeric parameter below is either a value the protocol steward has explicitly confirmed, or a proposed default awaiting sign-off. Each parameter is tagged:

- **[CONFIRMED]** — explicitly decided; implementations may treat this as final.
- **[PROPOSED — NEEDS SIGN-OFF]** — a reasonable default chosen so implementation work isn't blocked, but not yet a final decision. Anything tagged this way MUST be implemented as a governance-updatable parameter, never as a hardcoded constant, so a later sign-off change doesn't require a code rewrite.

Chapters 13, 14, 15, 16, and 23 of the whitepaper repeatedly defer exact figures to "the Tokenomics Specification." This is that document. It does not repeat the behavioral/state-machine design already specified in those chapters or in OFS-1600/2300/2400/4000 — it supplies the numbers those documents deliberately omitted.

---

# 1. Supply and Denomination

| Parameter | Value | Status |
|---|---|---|
| Token name | OPEN | [CONFIRMED] |
| Total / max supply | 1,000,000,000 OPEN | [CONFIRMED] |
| Decimals | 9 | [CONFIRMED] |
| Post-genesis minting | None. Mint authority is permanently revoked at genesis (see OFS-4200 §3). | [CONFIRMED] |

**Note on Ch.13 vs. Ch.23:** Chapter 13 states supply is fixed absolutely; Chapter 23 allows a governance-authorized minting exception. This specification resolves the inconsistency: **supply is strictly fixed for v1** (mint authority is revoked, not merely restricted). Any future mint event would require a new token (or a wrapped/bridged mechanism) rather than a code path that exists in v1 — this is a deliberate simplification to remove an entire class of governance-can-inflate-supply risk from the initial system.

# 2. Genesis Allocation

Seven buckets, named in Chapter 14, with no percentages given there. Proposed split:

| Bucket | % of supply | OPEN | Status |
|---|---|---|---|
| Community Presale | 3% | 30,000,000 | [PROPOSED — NEEDS SIGN-OFF] |
| AllenHark Treasury | 17% | 170,000,000 | [PROPOSED — NEEDS SIGN-OFF] |
| Ecosystem Treasury | 20% | 200,000,000 | [PROPOSED — NEEDS SIGN-OFF] |
| Infrastructure / Node Bootstrap Program | 15% | 150,000,000 | [PROPOSED — NEEDS SIGN-OFF] |
| Community Incentives | 20% | 200,000,000 | [PROPOSED — NEEDS SIGN-OFF] |
| Liquidity Programs | 15% | 150,000,000 | [PROPOSED — NEEDS SIGN-OFF] |
| Strategic Reserve | 10% | 100,000,000 | [PROPOSED — NEEDS SIGN-OFF] |
| **Total** | **100%** | **1,000,000,000** | |

**Why the Presale bucket is small (3%, not the 10% first floated in planning):** at the confirmed 1 OPEN = 1 USDC presale price (§3), bucket size directly sets the maximum possible raise. A 10% bucket would imply a $100,000,000 hard cap ceiling, which is not a realistic near-term raise target for a pre-revenue protocol. 3% (30,000,000 OPEN) implies a $30,000,000 ceiling, still generous headroom above a realistic hard cap (§3) without absurdly overshooting. **The actual hard cap (§3) and this bucket size must be moved together, not independently, if either changes.**

**Vesting** (Chapter 14 gives philosophy only, no durations):

| Bucket | Cliff | Vesting after cliff | Status |
|---|---|---|---|
| Community Presale | 0 (immediate unlock) | — | [PROPOSED — NEEDS SIGN-OFF; see §3 tradeoff note] |
| AllenHark Treasury | 12 months | 36 months linear | [PROPOSED — NEEDS SIGN-OFF] |
| Ecosystem Treasury | 12 months | 36 months linear | [PROPOSED — NEEDS SIGN-OFF] |
| Infrastructure / Node Bootstrap | 0 | Emitted programmatically per node-reward rules (OFS-1600), not a simple linear release | [PROPOSED — NEEDS SIGN-OFF] |
| Community Incentives | 0 | Emitted programmatically as incentives are earned | [PROPOSED — NEEDS SIGN-OFF] |
| Liquidity Programs | 3 months | 24 months linear | [PROPOSED — NEEDS SIGN-OFF] |
| Strategic Reserve | 12 months | 48 months linear | [PROPOSED — NEEDS SIGN-OFF] |

# 3. Presale Terms

| Parameter | Value | Status |
|---|---|---|
| Price | 1 OPEN = 1 USDC | [CONFIRMED] |
| Accepted contribution assets | Native SOL, and any stablecoin on the whitelist below | [CONFIRMED] |
| Conversion mechanism | Atomic on-chain swap via Jupiter's aggregator program (CPI), executed inside the same transaction as the contribution | [CONFIRMED] |
| Hard cap | 30,000,000 USDC-equivalent (sized to match the Presale bucket, §2) | [PROPOSED — NEEDS SIGN-OFF] |
| Soft cap | 5,000,000 USDC-equivalent; contributions are refundable if unmet by end time | [PROPOSED — NEEDS SIGN-OFF] |
| Minimum contribution per wallet | 50 USDC-equivalent | [PROPOSED — NEEDS SIGN-OFF] |
| Maximum contribution per wallet | 250,000 USDC-equivalent | [PROPOSED — NEEDS SIGN-OFF] |
| Vesting on presale tokens | None — immediate unlock at claim | [PROPOSED — NEEDS SIGN-OFF] |
| Max swap slippage tolerance | 1% | [PROPOSED — NEEDS SIGN-OFF] |
| Stablecoin whitelist | USDC, USDT, PYUSD (devnet equivalents/test mints during the devnet phase of this build) | [PROPOSED — NEEDS SIGN-OFF, and structurally must remain a governance-updatable list, never an open "any SPL token claiming to be a stablecoin"] |

**Immediate-unlock tradeoff, stated explicitly:** no vesting on presale tokens is simpler to ship and consistent with "the presale unlocks development funding" urgency, but it means presale buyers can sell into the open market the instant tokens are claimable, with no lockup smoothing out sell pressure. A short cliff + linear vest would reduce that risk at the cost of slower time-to-market for the presale itself. This specification proposes immediate unlock and flags the tradeoff rather than silently picking a side.

**Refund semantics:** if the soft cap is unmet, refunds are paid in **USDC** (the post-swap asset), not in whatever the contributor originally sent (SOL or another stablecoin). The presale UI must state this plainly before a contributor confirms a non-USDC contribution.

# 4. Staking

Seven staked roles per Chapter 15/23: Merchant, Arbitrator, Node Operator, Notification Provider, Oracle Provider, Risk Intelligence Provider, Snapshot Provider.

| Parameter | Value | Status |
|---|---|---|
| Minimum stake (all roles, flat for v1) | 1,000 OPEN | [PROPOSED — NEEDS SIGN-OFF] |
| Arbitrator minimum stake (higher bar, per Ch.11 §11.6) | 10,000 OPEN | [PROPOSED — NEEDS SIGN-OFF] |
| Unbonding / unlock period (all roles, flat for v1) | 7 days | [PROPOSED — NEEDS SIGN-OFF] |
| Effective-stake timing on unstake request | Reduces immediately at request time, not only at unlock release | [DECISION — whitepaper wording is ambiguous between these two readings; this specification picks the immediate-reduction interpretation to prevent a participant from requesting unstake and still counting the stake toward eligibility/voting/priority during the unbonding window] |
| Slashing percentage (flat, all misconduct types, v1) | 10% of staked amount | [PROPOSED — NEEDS SIGN-OFF] |
| Arbitrator minimum protocol age before eligibility | 30 days since first stake | [PROPOSED — NEEDS SIGN-OFF] |
| Arbitrator "no active penalties" definition | No unresolved slash event in the trailing 90 days | [PROPOSED — NEEDS SIGN-OFF] |

**Enumerated slashing triggers** (Chapter 15 gives categories, not concrete triggers — this is the concrete list a program can actually check):

1. Arbitrator misses a case decision deadline (deadline itself set by OFS-2400's dispute-lifecycle timing, currently undefined — flag as a dependency on a future OFS-2400 amendment).
2. Oracle-submitted price deviates from the Jupiter-referenced market price (OFS-4200 §5) by more than 5% at time of submission. [PROPOSED — NEEDS SIGN-OFF on the 5% figure]
3. Node operator's uptime/availability falls below a to-be-defined SWQoS threshold for a sustained period (OFS-1600 defines the priority formula but not a slashing threshold — flag as a dependency on a future OFS-1600 amendment).
4. Proven double-settlement attempt (an instruction sequence attempting to release the same escrow twice) — this is caught by the escrow program's own state machine (OFS-2300, OFS-4200 §6) and should never actually succeed on-chain, but a *proven attempt* (detected off-chain via transaction-log analysis) is still slashable as a deterrent.

Triggers 1 and 3 depend on future amendments to OFS-2400 and OFS-1600 respectively to define concrete deadlines/thresholds; until those exist, only triggers 2 and 4 are enforceable in v1.

# 5. Governance

| Parameter | Value | Status |
|---|---|---|
| Proposal identifier | OFIP-#### (e.g. OFIP-0001) | [DECISION — whitepaper's "OFIP" chosen over OFS-4000's "OFP"; more descriptive, whitepaper-facing naming wins] |
| Proposal categories | Informational, Standards, Parameter, Treasury, Protocol-Upgrade, Constitutional (whitepaper's 6-category taxonomy) | [DECISION — chosen over OFS-4000's 5-category Protocol/Economics/Marketplace/Infrastructure/Governance taxonomy] |
| Proposal stake deposit | 5,000 OPEN | [PROPOSED — NEEDS SIGN-OFF] |
| Deposit refund condition | Refunded if quorum is met by the voting deadline; forfeited to the Ecosystem Treasury otherwise | [DECISION — approximates the whitepaper's more subjective "frivolous/abandoned" language with a programmatically-checkable quorum-met proxy] |
| Vote-lock duration | 7 days (matches the voting period) | [PROPOSED — NEEDS SIGN-OFF] |
| Vote-weight snapshot timing | At the moment each vote is cast, not at proposal creation | [DECISION — either is defensible; cast-time chosen for v1 simplicity] |
| Quorum | 10% of circulating staked-for-governance supply | [PROPOSED — NEEDS SIGN-OFF] |
| Approval threshold — Informational, Standards, Parameter | Simple majority (>50%) | [PROPOSED — NEEDS SIGN-OFF] |
| Approval threshold — Treasury | 60% supermajority | [PROPOSED — NEEDS SIGN-OFF] |
| Approval threshold — Protocol-Upgrade, Constitutional | 66% supermajority + 20% quorum (higher than the standard 10%) | [PROPOSED — NEEDS SIGN-OFF] |

# 6. Treasury / Revenue Router

Fee amounts are deliberately **not** fixed here — Chapter 23 explicitly wants fees adjustable as OPEN's price moves, and this specification respects that: every fee is a governance-updatable parameter, never a constant.

| Parameter | Value | Status |
|---|---|---|
| Treasury sub-accounts | Development, Ecosystem, Infrastructure, Emergency Reserve | [CONFIRMED — named in Ch.14/16] |
| Dust-remainder-on-fee-split rule | Any remainder from integer-division fee splits goes to the Development sub-account | [PROPOSED — NEEDS SIGN-OFF] |
| Default ad-listing fee | 1 OPEN | [PROPOSED — NEEDS SIGN-OFF] |
| Default dispute-filing fee | 20 OPEN | [PROPOSED — NEEDS SIGN-OFF] |

# 7. Explicitly Out of Scope for v1

The following are real parts of the whitepaper's long-term vision but are **not** built in the phase of work this specification supports:

- Chapter 11's decentralized commit-reveal arbitration with per-case arbitrator staking and dynamic thresholds. v1 uses OFS-2400's simpler governance-appointed trusted-arbitrator model instead.
- Chapter 23's governance-authorized post-genesis minting exception (§1 — supply is strictly fixed for v1).
- Partial Settlement as a dispute resolution outcome (OFS-2400 marks this "Future").
- Per-role-differentiated staking minimums (v1 ships flat + one arbitrator-specific minimum; further differentiation is a future governance parameter change, not new code).

# 8. Decision Log

Every place this specification made a call rather than citing a whitepaper number:

1. Decimals = 9.
2. Presale bucket = 3% (not a naive 10%), sized against the hard cap.
3. All allocation percentages, vesting cliffs/durations.
4. Presale hard/soft cap, min/max contribution, no vesting on presale tokens (with tradeoff stated), 1% max slippage, stablecoin whitelist contents.
5. Staking minimums (flat + arbitrator-specific), unbonding period, slashing %, the four enumerated slashing triggers (two of which depend on future amendments elsewhere).
6. Unstake-request effective-stake timing = immediate reduction.
7. Governance: "OFIP" naming, 6-category taxonomy, deposit amount, quorum/threshold numbers, vote-lock duration, cast-time snapshot, quorum-met-as-frivolity-proxy.
8. Fee amounts, dust-handling rule.
9. The strict-fixed-supply resolution of the Ch.13/Ch.23 inconsistency.
