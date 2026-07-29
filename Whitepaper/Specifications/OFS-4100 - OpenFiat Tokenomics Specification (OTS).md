# OFS-4100 — OpenFiat Tokenomics Specification (OTS)

**Document ID:** OFS-4100

**Title:** OpenFiat Tokenomics Specification

**Version:** 0.1.0 (**DRAFT — NOT YET SIGNED OFF**)

**Status:** Draft

**Category:** Economics

**Depends On:** OFS-1600, OFS-2300, OFS-2400, OFS-4000

---

## Status Banner

**This document is a DRAFT.** Every numeric parameter below is either a value the protocol steward has explicitly confirmed, or a proposed default awaiting sign-off. Each parameter is tagged:

- **[CONFIRMED]** — explicitly decided; implementations may treat this as final.
- **[PROPOSED — NEEDS SIGN-OFF]** — a reasonable default chosen so implementation work isn't blocked, but not yet a final decision. Anything tagged this way MUST be implemented as a governance-updatable parameter, never as a hardcoded constant, so a later sign-off change doesn't require a code rewrite.

Chapters 13, 14, 15, 16, and 23 of the whitepaper repeatedly defer exact figures to "the Tokenomics Specification." This is that document. It does not repeat the behavioral/state-machine design already specified in those chapters or in OFS-1600/2300/2400/4000 — it supplies the numbers those documents deliberately omitted.

---

## 1. Supply and Denomination

| Parameter | Value | Status |
|---|---|---|
| Token name | OPEN | [CONFIRMED] |
| Total / max supply | 1,000,000,000 OPEN | [CONFIRMED] |
| Decimals | 9 | [CONFIRMED] |
| Post-genesis minting | None. Mint authority is permanently revoked at genesis (see OFS-4200 §3). | [CONFIRMED] |

**Note on Ch.13 vs. Ch.23:** Chapter 13 states supply is fixed absolutely; Chapter 23 allows a governance-authorized minting exception. This specification resolves the inconsistency: **supply is strictly fixed for v1** (mint authority is revoked, not merely restricted). Any future mint event would require a new token (or a wrapped/bridged mechanism) rather than a code path that exists in v1 — this is a deliberate simplification to remove an entire class of governance-can-inflate-supply risk from the initial system.

## 2. Genesis Allocation

Seven buckets, named in Chapter 14, with no percentages given there. Proposed split:

| Bucket | % of supply | OPEN | Status |
|---|---|---|---|
| Community Presale | 20% | 200,000,000 | [CONFIRMED] |
| Ecosystem Treasury | 17% | 170,000,000 | [PROPOSED — NEEDS SIGN-OFF] |
| Community Incentives | 17% | 170,000,000 | [PROPOSED — NEEDS SIGN-OFF] |
| AllenHark Treasury | 14% | 140,000,000 | [PROPOSED — NEEDS SIGN-OFF] |
| Infrastructure / Node Bootstrap Program | 12% | 120,000,000 | [PROPOSED — NEEDS SIGN-OFF] |
| Liquidity Programs | 12% | 120,000,000 | [PROPOSED — NEEDS SIGN-OFF] |
| Strategic Reserve | 8% | 80,000,000 | [PROPOSED — NEEDS SIGN-OFF] |
| **Total** | **100%** | **1,000,000,000** | |

**Why the Presale bucket is the entire 20% (200,000,000 OPEN), confirmed by the protocol steward:** this is no longer a raise-ceiling sizing choice, because the presale itself has no fixed hard cap on demand (§3). The presale sells from this bucket at 1 OPEN = 1 USDC toward a $2,000,000 *target* — a goal, not a cap — and keeps selling into the same 200,000,000-token bucket for as long as demand continues. Whatever remains unsold when the presale closes is offered afterward in a **Public Sale** at 1 OPEN = 1.25 USDC (§3). Because the bucket funds two sequential sale phases rather than one capped raise, its size is decoupled from the $2,000,000 target. The other six buckets absorb the resulting reduction proportionally to their prior share (each individually rounded to the nearest whole percent via largest-remainder so the split still sums to exactly 100%).

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

## 3. Presale Terms

| Parameter | Value | Status |
|---|---|---|
| Price | 1 OPEN = 1 USDC | [CONFIRMED] |
| Accepted contribution assets | Native SOL, and any stablecoin on the whitelist below | [CONFIRMED] |
| Conversion mechanism | Atomic on-chain swap via Jupiter's aggregator program (CPI), executed inside the same transaction as the contribution | [CONFIRMED] |
| Raise target | 2,000,000 USDC-equivalent — a goal, not a cap. The presale keeps selling out of the full 200,000,000 OPEN Community Presale bucket (§2) if demand continues past this target. | [CONFIRMED] |
| Hard cap | None distinct from the bucket itself — the presale may sell up to the full 200,000,000 OPEN Community Presale allocation. | [CONFIRMED] |
| Soft cap | 5,000,000 USDC-equivalent; contributions are refundable if unmet by end time | [PROPOSED — NEEDS SIGN-OFF] |
| Minimum contribution per wallet | 50 USDC-equivalent | [PROPOSED — NEEDS SIGN-OFF] |
| Maximum contribution per wallet | 1,000,000 USDC-equivalent | [CONFIRMED] |
| Vesting on presale tokens | None — immediate unlock at claim | [PROPOSED — NEEDS SIGN-OFF] |
| Max swap slippage tolerance | 1% | [PROPOSED — NEEDS SIGN-OFF] |
| Stablecoin whitelist | USDC, USDT, PYUSD (devnet equivalents/test mints during the devnet phase of this build) | [PROPOSED — NEEDS SIGN-OFF, and structurally must remain a governance-updatable list, never an open "any SPL token claiming to be a stablecoin"] |

**Immediate-unlock tradeoff, stated explicitly:** no vesting on presale tokens is simpler to ship and consistent with "the presale unlocks development funding" urgency, but it means presale buyers can sell into the open market the instant tokens are claimable, with no lockup smoothing out sell pressure. A short cliff + linear vest would reduce that risk at the cost of slower time-to-market for the presale itself. This specification proposes immediate unlock and flags the tradeoff rather than silently picking a side.

**Refund semantics:** if the soft cap is unmet, refunds are paid in **USDC** (the post-swap asset), not in whatever the contributor originally sent (SOL or another stablecoin). The presale UI must state this plainly before a contributor confirms a non-USDC contribution.

**Public Sale (follow-on phase):**

| Parameter | Value | Status |
|---|---|---|
| Trigger | Presale closes (end time reached) with unsold OPEN remaining in the Community Presale bucket | [CONFIRMED] |
| Price | 1 OPEN = 1.25 USDC | [CONFIRMED] |
| Offered amount | Whatever remains of the 200,000,000 OPEN Community Presale bucket after the presale closes | [CONFIRMED] |
| Accepted contribution assets, conversion mechanism | Same as the presale (§3, above) | [PROPOSED — NEEDS SIGN-OFF, pending confirmation the same program/mechanism is reused rather than a separate deployment] |

The Public Sale is not a separate token bucket — it is the second, higher-priced phase for selling whatever the presale itself did not sell. A contributor cannot buy at the presale's 1 OPEN = 1 USDC rate once the presale phase has closed.

## 4. Staking

Seven staked roles per Chapter 15/23: Merchant, Arbitrator, Node Operator, Notification Provider, Oracle Provider, Risk Intelligence Provider, Snapshot Provider.

| Parameter | Value | Status |
|---|---|---|
| Minimum stake — default, every role without a specific figure below | 1,000 OPEN | [PROPOSED — NEEDS SIGN-OFF] |
| Minimum stake — Arbitrator (higher bar, per Ch.11 §11.6) | 10,000 OPEN | [PROPOSED — NEEDS SIGN-OFF] |
| Minimum stake — Notification Provider | 5,000 OPEN | [DECISION — protocol steward] |
| How minimums are stored | `StakingConfig.min_stake_by_role`, a `[u64; 7]` indexed by `Role` | [IMPLEMENTED — replaced a flat field plus a special-cased arbitrator field, which made §7's "future governance parameter change, not new code" impossible to honour] |
| How minimums are enforced | `stake` and `request_unstake` both reject a resulting balance that is neither zero nor at least the role's minimum | [IMPLEMENTED — until this landed the minimums were stored and read by nothing, so any amount was accepted] |
| Unstaking below the minimum | Refused; a full exit to zero is always allowed, so a minimum can never trap tokens | [DECISION — the alternative, letting a balance sit below its minimum, leaves an account that still reads as staked while no longer qualifying] |
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

## 5. Governance

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

## 6. Treasury / Revenue Router

Fee amounts are deliberately **not** fixed here — Chapter 23 explicitly wants fees adjustable as OPEN's price moves, and this specification respects that: every fee is a governance-updatable parameter, never a constant.

| Parameter | Value | Status |
|---|---|---|
| Treasury sub-accounts | Development, Ecosystem, Infrastructure, Emergency Reserve | [CONFIRMED — named in Ch.14/16] |
| Dust-remainder-on-fee-split rule | Any remainder from integer-division fee splits goes to the Emergency Reserve sub-account | [PROPOSED — NEEDS SIGN-OFF; amended to match the deployed `openfiat-escrow`, which sweeps the remainder to the emergency reserve. The earlier text said Development. The amounts are sub-unit rounding dust, but the spec and the program must not disagree — if Development is the intended destination, the program is what needs changing, not this row] |
| Default ad-listing fee | 1 OPEN | [PROPOSED — NEEDS SIGN-OFF] |
| Default dispute-filing fee | 20 OPEN | [PROPOSED — NEEDS SIGN-OFF] |

## 7. Explicitly Out of Scope for v1

The following are real parts of the whitepaper's long-term vision but are **not** built in the phase of work this specification supports:

- Chapter 23's governance-authorized post-genesis minting exception (§1 — supply is strictly fixed for v1).
- Partial Settlement as a dispute resolution outcome (OFS-2400 marks this "Future").

## 8. Decision Log

Every place this specification made a call rather than citing a whitepaper number:

1. Decimals = 9.
2. Presale bucket = 20% (200,000,000 OPEN), confirmed by the protocol steward — funds two sequential sale phases (a $2,000,000-target presale at 1 OPEN = 1 USDC, then a public sale at 1 OPEN = 1.25 USDC for whatever remains unsold) rather than sizing a single capped raise.
3. All allocation percentages, vesting cliffs/durations.
4. Presale hard/soft cap, min/max contribution, no vesting on presale tokens (with tradeoff stated), 1% max slippage, stablecoin whitelist contents.
5. Staking minimums (per-role, §4), unbonding period, slashing %, the four enumerated slashing triggers (two of which depend on future amendments elsewhere). Minimums were originally specified as flat-plus-arbitrator and listed in §7 as not-yet-differentiated; they are now a per-role array and are actually enforced.
6. Unstake-request effective-stake timing = immediate reduction.
7. Governance: "OFIP" naming, 6-category taxonomy, deposit amount, quorum/threshold numbers, vote-lock duration, cast-time snapshot, quorum-met-as-frivolity-proxy.
8. Fee amounts, dust-handling rule.
9. The strict-fixed-supply resolution of the Ch.13/Ch.23 inconsistency.
10. §9 in full — epoch length, bootstrap emission schedule, the stake×connectivity×availability
    share formula, the 1.0/0.4 connectivity multiplier, and funding arbitrator rewards from the
    dispute-filing fee. OFS-1600 §16 and this specification previously deferred to each other,
    so there was no formula to cite.
11. Dust-remainder destination amended from Development to Emergency Reserve to match the
    deployed program (§6).

---

## 9. Reward Distribution

OFS-1600 §16 defers reward calculation to this specification; this specification
previously deferred emission to "node-reward rules (OFS-1600)". Neither defined a
formula, so no reward was computable and none was ever paid. This section closes
that circular reference.

Every number here is `[PROPOSED — NEEDS SIGN-OFF]`. The *mechanism* is what this
section fixes; the rates are a starting point, and all of them are governance
parameters rather than constants, consistent with §6.

### 9.1 Funding

| Source | Role | Status |
|---|---|---|
| Infrastructure / Node Bootstrap genesis bucket (12%, 120,000,000 OPEN) | Bootstrap emission, while protocol revenue is too small to matter | [PROPOSED — NEEDS SIGN-OFF] |
| Infrastructure sub-account's share of the settlement fee | Steady-state funding, growing with real usage | [PROPOSED — NEEDS SIGN-OFF] |

The whitepaper's stated position is that returns come from protocol revenue rather
than token inflation. The genesis bucket is therefore a bootstrap, not a permanent
emission: it is finite, and once exhausted the reward pool is exactly what the
network earned.

### 9.2 Node operator rewards

| Parameter | Value | Status |
|---|---|---|
| Epoch length | 24 hours | [PROPOSED — NEEDS SIGN-OFF] |
| Bootstrap emission | 120,000,000 OPEN linear over 4 years (≈82,192 OPEN per epoch), capped by the remaining bucket | [PROPOSED — NEEDS SIGN-OFF] |
| Per-node share | Proportional to `effective_stake × connectivity × availability` | [PROPOSED — NEEDS SIGN-OFF] |
| Connectivity multiplier | `RpcConnected` = 1.0, `GossipOnly` = 0.4 | [PROPOSED — NEEDS SIGN-OFF] |
| Availability multiplier | The fraction of the epoch a node was observed live, measured from its own signed chain-bridge announcements and gossip participation as seen by the paying node | [PROPOSED — NEEDS SIGN-OFF] |
| Eligibility floor | Stake at or above the role minimum (§4), and registered in the OFS-1500 registry | [PROPOSED — NEEDS SIGN-OFF] |

Stake alone must not determine reward, or the network pays for capital rather than
service — OFS-1600 §5's "reputation is earned" applies here. The connectivity
multiplier exists because a node bridging to Solana does strictly more work than one
only gossiping, and the difference is externally observable from its own signed
blockhash announcements rather than self-reported: a node cannot fabricate a valid
recent (blockhash, slot) pair.

Availability deliberately does **not** reuse OFS-3000 §13. That dimension measures a
*trader's* responsiveness on a settlement and says nothing about a node. Node
liveness is a separate measurement with its own hazard: OFS-4300 §6's amplification
control suppresses re-forwarding a `BlockhashAnnounced` whose (blockhash, slot) has
already been seen, so a naive network-wide count rewards whichever node announces
first rather than whichever node is actually up. An implementation MUST account for
that — measuring from what the paying node itself observes, rather than from
propagation reach.

A fully manipulation-resistant node-availability measure is **not** solved by this
specification. Announcement cadence is a lower bound on liveness, not a proof of
service quality, and it is stated here as the best currently-observable signal rather
than a finished answer.

### 9.3 Arbitrator rewards

| Parameter | Value | Status |
|---|---|---|
| Source | The dispute-filing fee (§6, default 20 OPEN), which must actually be collected on `open_dispute_case` | [PROPOSED — NEEDS SIGN-OFF] |
| Distribution | Split among arbitrators whose revealed vote matched the tallied outcome, pro-rata by revealed weight | [PROPOSED — NEEDS SIGN-OFF] |
| Outside-consensus penalty | The existing slashing rate (§4) applied to the arbitrator's active stake | [PROPOSED — NEEDS SIGN-OFF] |
| Frivolous-dispute case | Fee forfeited rather than refunded, and distributed as above | [PROPOSED — NEEDS SIGN-OFF] |

This is the incentive the commit-reveal design depends on: without it, voting
carries cost and no return, and voting *against* consensus carries no cost at all.

### 9.4 Auditability

Reward and slash events MUST be emitted as on-chain events. Without them there is no
history: a participant cannot reconstruct what they earned, and no explorer can show
it. This is a hard requirement, not a nice-to-have — an unauditable reward system is
indistinguishable from an arbitrary one.

### 9.5 Not specified here

Service-provider rewards (notification, oracle, risk intelligence, snapshot) are
deliberately left open. Each needs a metered unit of work before a fee can be
attached to it, and no such metering exists yet. Stating a revenue share before the
meter exists would be a promise the protocol cannot keep.
