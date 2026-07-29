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
| Default ad-listing fee | 1 OPEN, charged from the merchant's liquidity vault | [PROPOSED — NEEDS SIGN-OFF on the amount; the vault as the source is a protocol-steward DECISION] |
| Default settlement fee | 0.85% (85 bps) of the traded amount, **borne by the buyer** — deducted from the stablecoin released to them, in the stablecoin traded | [DECISION — protocol steward] |
| Default dispute-filing fee | 20 OPEN, charged from the merchant's liquidity vault (§9.3) | [PROPOSED — NEEDS SIGN-OFF on the amount; the payer is a protocol-steward DECISION] |

**Who pays what.** The three fees do not fall on the same party, and the split is
deliberate:

| Fee | Payer | Source |
|---|---|---|
| Settlement fee (0.85%) | Buyer | Deducted from the escrowed stablecoin before payout |
| Ad-listing fee | Merchant | Their liquidity vault |
| Arbitration deposit | Merchant | Their liquidity vault, whoever opened the dispute (§9.3) |

A buyer pays only when a trade actually completes, and pays nothing to raise a
dispute. A merchant pays to advertise and to be arbitrated, and recovers the
arbitration deposit unless the outcome goes against them. The party with an ongoing
business carries the standing costs; the party who may be transacting once carries a
cost only on success.

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
12. §9.3's payer asymmetry — the merchant funds the arbitration deposit from their liquidity
    vault whichever party opens the dispute, forfeited to the arbitration OPEN pool only when
    the outcome goes against them — and the ad-listing fee drawing on the same vault. Both are
    protocol-steward decisions; what happens to the deposit when the merchant does not lose was
    not stated and is this specification's reading.
13. Settlement fee default of 0.85%, borne by the buyer (§6). Supersedes the 15 bps that was
    documented and deployed. Governance-updatable like every other fee.
14. §9.6's oracle formula shape — freshness weighting and the in-use eligibility condition are
    this specification's anti-gaming proposal; the steward's decision was that payment scales
    with currencies covered. The new-corridor bootstrap gap is recorded, not solved.
15. Snapshot providers receive no compensation (§9.5); oracle and risk-intelligence
    compensation are both scaled by observed uptime (§9.6, §9.7); the risk-intelligence
    subscription is paid by the treasury; AllenHark's default service key is
    `ALLENLMtV1zEAHT3xpVryqcbdPCB8c9JhM1Jdbe5XHg5`. All protocol-steward decisions.
16. Risk-intelligence compensation scales with wallets processed rather than nodes served
    (§9.7). The real-activity condition on what counts as processed is this specification's
    anti-gaming term, not the steward's instruction.
17. A wallet is scanned at most once per 48 hours, cached node-side, and is billable once per
    that window (§9.7).

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
| Who posts the arbitration deposit | The **merchant**, from their liquidity vault — regardless of which party opened the dispute | [DECISION — protocol steward] |
| Source | The dispute-filing fee (§6, default 20 OPEN), charged on `open_dispute_case` | [PROPOSED — NEEDS SIGN-OFF] |
| If the merchant loses | Deposit forfeited to the arbitration OPEN pool, a destination distinct from the four settlement-fee treasuries | [DECISION — protocol steward] |
| If the merchant does not lose | Deposit returns to the merchant's vault | [PROPOSED — NEEDS SIGN-OFF; the steward stated the losing case only] |
| No-consensus terminal split | Deposit returns to the merchant — there is no loser, and forfeiting would punish a merchant who did nothing wrong | [PROPOSED — NEEDS SIGN-OFF] |
| Distribution | From the arbitration OPEN pool, split among arbitrators whose revealed vote matched the tallied outcome, pro-rata by revealed weight | [PROPOSED — NEEDS SIGN-OFF] |
| Outside-consensus penalty | The existing slashing rate (§4) applied to the arbitrator's active stake | [PROPOSED — NEEDS SIGN-OFF] |

This is the incentive the commit-reveal design depends on: without it, voting
carries cost and no return, and voting *against* consensus carries no cost at all.

The merchant always funding the deposit is a deliberate asymmetry, not an
oversight. A buyer is frequently a one-time participant with no vault and no
standing balance; requiring them to fund a deposit to be heard would price the
dispute mechanism out of reach of exactly the party it most protects. The merchant
has an ongoing vault and an ongoing business, and only loses the deposit when the
outcome goes against them — so an honest merchant faces a temporary lock, not a
cost, while a merchant in the wrong funds the arbitration that found them so.

The consequence to watch is that a buyer can open disputes at no cost to
themselves, which bounds how much merchant liquidity an adversary can tie up. The
reputation dimensions in OFS-3000 and the merchant's own ability to decline
counterparties are the intended defence; if that proves insufficient in practice,
a buyer-side cost is a parameter change rather than a redesign.

### 9.4 Auditability

Reward and slash events MUST be emitted as on-chain events. Without them there is no
history: a participant cannot reconstruct what they earned, and no explorer can show
it. This is a hard requirement, not a nice-to-have — an unauditable reward system is
indistinguishable from an arbitrary one.

### 9.5 Service-provider earnings

Provider compensation takes three different shapes, and which one applies depends on
the role rather than on any general rule:

- **Billing a consumer directly** — a notification gateway charges the participant
  who enabled notifications.
- **Paid by the protocol** — oracle providers (§9.6) and risk intelligence (§9.7),
  because their output is free at the point of use and would otherwise go unfunded.
- **Nothing** — snapshot providers, by decision (§9.5's table).

An earlier draft of this section asserted that providers are never paid a protocol
reward and never draw on the emission pool. That is no longer true and was corrected
here rather than left to contradict §9.6, which funds oracle payment from the same
Infrastructure allocation and treasury share that §9.2 draws on.

| Parameter | Value | Status |
|---|---|---|
| Billing currency | OPEN, USDC, or another configured token, chosen by the provider | [DECISION — protocol steward] |
| Price | Declared in the provider's OFS-1500 registration, as a token and an amount rather than free text | [DECISION — protocol steward] |
| Payout destination | The wallet registered against the service | [DECISION — protocol steward] |
| Earnings visibility | The provider proves control of the service by signing a message with the registered key, and reads their earnings back | [DECISION — protocol steward] |

Proving ownership by signature rather than by account is what keeps this consistent
with the rest of the protocol: there is no provider login, no session, and no
registry of people — only a key that can demonstrate control of a Service ID it already
owns. The same mechanism already authenticates arbitrators.

**Per-role billing triggers.**

Consumption and compensation are separate questions, and conflating them is the
mistake this table exists to prevent. A service can be free to consume *and* paid
for by the protocol.

| Role | What a consumer pays | What the provider receives | Status |
|---|---|---|---|
| Notification gateway | Per delivery, by the participant who enabled it | That fee | [PROPOSED — NEEDS SIGN-OFF] |
| Oracle provider | **Nothing. Reads are free** | Paid by the protocol, scaled by currency coverage (§9.6) | [DECISION — protocol steward] |
| Snapshot provider | **Nothing. Downloads are free** | **Nothing.** No provider compensation | [DECISION — protocol steward] |
| Risk intelligence | Nothing directly | A subscription paid by the protocol, default 1,000 USDC/month (§9.7) | [DECISION — protocol steward] |

Oracle rates and snapshots are free to consume because charging would work against
the protocol: a priced rate feed is consulted less, which makes the median it
contributes to thinner and easier to move, and a priced snapshot slows the thing
that lets a new node join at all. Paying their providers from the protocol keeps the
good free at the point of use while still funding its supply.

Oracle rates and snapshots are free because charging for them would work against
the protocol: a priced rate feed is consulted less, which makes the median it
contributes to thinner and easier to move, and a priced snapshot slows the thing
that lets a new node join at all. Both are load-bearing public goods, and the
network is stronger when they are cheap to consume.

**Snapshot providers are not compensated.** Downloads are free and the role carries
no payment, by decision. It is run by parties already operating a node — serving a
snapshot is a marginal cost on infrastructure that already exists and is already
paid for through the node reward pool (§9.2). A provider considering a standalone
snapshot service should read this as: there is no revenue in it, and none is
planned.

### 9.6 Oracle provider compensation

Paid by the protocol, scaled by how much of the market a provider actually covers.
Everything below is `[PROPOSED — NEEDS SIGN-OFF]`; the steward's decision is that
payment scales with the number of currencies provided, and the shape is this
specification's proposal.

| Parameter | Value |
|---|---|
| Basis | Distinct currency pairs for which the provider published a fresh, unexpired rate during the epoch |
| Freshness weight | Each pair counts by the fraction of the epoch it was covered by an unexpired rate, not merely touched once |
| Eligibility of a pair | A pair counts only if it is **in use** — referenced by at least one active advertisement, or independently covered by at least one other provider |
| Uptime | The share is scaled by the provider's observed uptime over the epoch, on the same basis §9.2 uses for nodes — measured from what other participants observed, never self-asserted |
| Epoch | 24 hours, matching §9.2 |
| Funding | The same Infrastructure allocation and treasury share that funds §9.2 |

The in-use condition is the anti-gaming term and is the reason this is not simply a
count. Paying per declared pair invites a provider to publish hundreds of invented
or dead pairs at near-zero cost and collect on all of them; requiring that somebody
either trades against a pair or independently quotes it ties payment to coverage
that matters. Freshness weighting exists for the same reason at the time axis: a
single rate posted once an epoch should not earn what continuous coverage earns.

**Not solved here:** a provider quoting a pair *nobody else does* and that no
advertisement yet references earns nothing, which is exactly backwards for
bootstrapping a new corridor. AllenHark providing default initial coverage mitigates
this in practice, but a bootstrap allowance for genuinely new corridors is a gap
worth closing before the formula is signed off.

### 9.7 Risk intelligence: a governance-approved, paid slot

Risk intelligence differs from every other provider role in two ways, both
protocol-steward decisions.

| Parameter | Value | Status |
|---|---|---|
| Compensation | A base of **1,000 USDC per month**, plus a component scaling with the number of **wallets processed** over the period | [DECISION — protocol steward] |
| Scaling metric | Wallets processed — **not** nodes, not queries served, not records published | [DECISION — protocol steward] |
| Adjustability | Both the base and the per-wallet rate are governance-configurable, and expected to change as the network grows | [DECISION — protocol steward] |
| Who may provide | **Only providers approved by governance.** Registration is permissioned for this role alone | [DECISION — protocol steward] |
| Uptime | The subscription is scaled by observed uptime over the billing period — a provider that is down does not draw a full month | [DECISION — protocol steward] |
| Default provider | AllenHark, service key `ALLENLMtV1zEAHT3xpVryqcbdPCB8c9JhM1Jdbe5XHg5` | [DECISION — protocol steward] |
| Payer | **The treasury** | [DECISION — protocol steward] |

Approval is what makes a fixed subscription safe. Every other role in this protocol
is permissionless because the cost of a bad actor is bounded — a useless oracle is
outvoted by the median, a dead notification gateway simply fails to deliver. A
standing payment has no such bound: without a gate, anyone could register as a risk
provider and draw a subscription. Gating who may *receive* is therefore not a
departure from the protocol's permissionless design so much as the condition that
lets a paid slot exist at all.

The treasury pays, which is what makes the approval gate load-bearing: an unapproved
party drawing a standing subscription would be drawing directly on protocol funds.

**Volume scaling ties pay to work, and needs one guard.** A provider's real cost is
proportional to how many wallets it must screen, not to how many nodes consult the
result — a single flagged wallet read by a thousand nodes is one piece of work. The
base plus per-wallet shape reflects that, and lets the fee grow with the network the
way §9.7's adjustability requires without governance having to re-vote a flat figure
every quarter.

The guard: a **risk provider is the party that publishes risk records**, so paying
per wallet processed pays them for output they alone generate. Left unqualified, a
provider could publish records against arbitrary addresses and bill for every one.
A wallet therefore counts only when it was processed against **real protocol
activity** — it was party to a reservation or settlement during the period, or was
the subject of a client screening request — and never merely because the provider
emitted a record naming it. This is the same shape as §9.6's in-use condition for
oracle pairs, and for the same reason: any per-unit payment to the party that
creates the units needs an independent witness to the unit being real.

**Scan cadence caps the billable unit.** A wallet is scanned **at most once every 48
hours**, and nodes cache the result for that window rather than re-querying — both
to keep risk providers from being overloaded by repeated identical questions, and
because a risk assessment that changes faster than that is not an assessment, it is
noise.

This settles what "processed" counts as: a wallet is billable **once per 48-hour
window**, however many times it is looked up in that window. Repeated lookups are
served from cache and cost the provider nothing, so they should earn nothing. It
also caps the protocol's exposure — the maximum bill in a period is the number of
distinct wallets in real activity divided by the cadence, which is a figure
governance can reason about in advance rather than discover afterwards.

The cache is a node-side concern, not a provider-side one: a provider cannot be
trusted to report how often it was asked, for exactly the reason §9.7 already gives
about self-reported volume.

`[PROPOSED — NEEDS SIGN-OFF]`: the per-wallet rate, and whether the base is a floor
or an advance against volume. The 48-hour cadence and the once-per-window billing
unit are protocol-steward decisions.

**Uptime scaling applies to both paid provider roles.** A fixed monthly figure that
pays the same whether a provider served every query or none is an invitation to
register and go quiet. Uptime must be measured the way §9.2 measures it for nodes —
from what other participants observed, never from a provider's own assertion about
itself. The registry's health-update path is a provider's *claim* about its state
and is not admissible as the basis for payment.
