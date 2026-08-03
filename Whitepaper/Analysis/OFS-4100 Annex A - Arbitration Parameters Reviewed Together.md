# OFS-4100 Annex A — The arbitration parameters, reviewed as one system

**Status:** `[SIGNED OFF — A and C IMPLEMENTED]` (task #125). B and D were not
built; see [What was built](#what-was-built) for what shipped, what shipped
differently from the proposal, and what could not honestly be built at all.
**Scope:** the quorum floor, the seat count, the round budget, the barring rule and the sortition threshold — reviewed together, because reviewed apart they are individually defensible and jointly broken.

This annex exists because task #125 asked for the 3-of-7 quorum and the round budget to be revisited *together with* the sortition threshold. That framing turned out to be the whole finding: each parameter is sound on its own terms, and their interaction has a failure mode none of them shows in isolation.

## The parameters as they stand

| Parameter | Value | Where | Status |
|---|---|---|---|
| `MAX_ARBITRATORS` (seats per round) | 7 | `escrow/state.rs` | shipped |
| `MIN_ARBITRATORS` (quorum floor, counted reveals) | 3 | `escrow/state.rs` | shipped |
| `MAX_DISPUTE_ROUNDS` | 3 | `escrow/constants.rs` | `[PROPOSED]` |
| `MIN_DECIDABLE_ARBITRATOR_POOL` | 17 (= 3 + 14) | `escrow/arbitration.rs` | derived, added by this annex |
| Published eligible-arbitrator count | 0 (unpublished) | `ArbitrationPolicy` | added by this annex, **ships disabled** |
| `MAX_BARRED_ARBITRATORS` | 14 (= 7 × (3−1)) | `escrow/state.rs` | derived |
| Arbitrator minimum stake | 500 OPEN | `StakingConfig` | signed off, §7 |
| Arbitrator stake age | 30 days | `FeeConfig` | signed off §4, **ships disabled (0)** |
| Opening sortition threshold | 100 bps, widening | `FeeConfig` | signed off §4.1, **ships disabled (0)** |

Each is individually justified in code, and those justifications hold. The quorum floor counts reveals the tally actually uses, so zero-weight reveals cannot pad it. The barring rule stops a losing party taking seats and going silent to force a no-decision. The sortition threshold widens across the commit window so a small pool still reaches liveness. All correct.

## The finding: barring and the round budget impose a floor on the arbitrator pool

The barring rule (#123) and the round budget (`MAX_DISPUTE_ROUNDS`) compose into a constraint neither states.

Every round, up to 7 wallets can take a seat and stay silent. Each is then barred from the case for good. Across a 3-round case that is up to **14 wallets removed from the eligible pool for this case** — which is exactly what `MAX_BARRED_ARBITRATORS = MAX_ARBITRATORS * (MAX_DISPUTE_ROUNDS - 1)` encodes.

For the final round to reach a verdict it needs 3 *counted reveals*. So the eligible pool must contain at least 14 + 3 = **17 arbitrators**, and in practice more, since qualifying for a seat is not the same as taking one and revealing.

Below that pool size, the third round cannot reach quorum **structurally** — not because arbitrators disagreed, but because there are not enough left who are allowed to vote. The case then lands on the terminal even split.

**The even split is precisely what a griefing party wants.** A merchant facing a losing dispute over 1,000 USDC recovers 500 by ensuring no round ever decides. So on a pool below ~17, the defence built in #123 becomes the attacker's instrument: barring is what exhausts the pool, and exhausting the pool is what delivers the split.

### What it costs the attacker today

With the stake age gate shipping at **0** — correctly, since nobody on a new chain can satisfy a 30-day age requirement — the only cost of a seat-squatting wallet is the arbitrator minimum:

> 15 wallets × 500 OPEN = **7,500 OPEN**, available immediately, no waiting period.

That is the entire cost of forcing an even split on any dispute, for as long as the real arbitrator pool is smaller than roughly 17 active participants. On devnet and at launch it certainly will be.

Note the stake is *locked, not slashed* — the squatter never reveals outside consensus, so nothing triggers a penalty. This is the same observation `commit_dispute_vote`'s own doc makes about seat-holding: "It is capital locked, not capital at risk." The barring rule was the answer to that within a round. Across rounds with a small pool, it stops being one.

## Why raising the sortition threshold makes it worse, not better

The instinct would be to tighten the draw. It backfires: the sortition threshold selects a *fraction* of the eligible pool, so raising it shrinks the number of wallets that can take a seat in any given round. On a pool already too small to survive 14 barrings, a tighter draw brings the structural-no-quorum point *closer*.

This is why #125 was right that these cannot be set independently. The draw defends against an attacker *choosing* seats; it does nothing about an attacker *having enough wallets*, and it makes the pool-exhaustion problem sharper. The parameter that actually defends against wallet manufacture is the **stake age gate** — and that is the one that ships disabled.

## Proposals

Four options, not mutually exclusive. My recommendation is **A + C**. That is what was signed off and built — see [What was built](#what-was-built) for how faithfully, and for the one part of A that could not be built as written.

### A. Make the pool floor explicit and enforced `[SIGNED OFF — BUILT, with one part qualified]`

A case should not open a round it cannot possibly resolve. Before opening round *n*, require

> eligible pool ≥ `MIN_ARBITRATORS` + (barred so far)

and if that fails, stop re-opening and go to the terminal split **immediately**, recording *why*. This changes no outcome — the split happens either way — but it stops the protocol pretending a decision was attempted, and it makes the condition observable rather than inferred from a case that quietly bounced three times. An operator seeing "terminal split: pool exhausted" can act; one seeing three indecisive rounds cannot tell this from genuine disagreement.

### B. Scale the round budget to the pool `[NOT BUILT]`

Replace the constant 3 with `min(3, floor((pool − MIN_ARBITRATORS) / MAX_ARBITRATORS) + 1)`. A large pool keeps all three rounds; a small one gets fewer, which is honest — a second round it cannot staff is theatre. Downside: `MAX_BARRED_ARBITRATORS` is an Anchor `#[max_len]`, so the account layout must still be sized for the maximum. Sizing stays at 14; only the behaviour adapts.

### C. Enable the stake age gate before the sortition threshold `[SIGNED OFF — BUILT]`

Both currently ship at 0. The ordering in which governance switches them on matters and is not written down anywhere:

1. **Age gate first.** It is the only parameter that costs an attacker *time* rather than capital, and it is what makes wallet manufacture expensive. Enable it once the chain is old enough for real arbitrators to satisfy it — 30 days after the first arbitrators stake, not 30 days after genesis.
2. **Sortition second**, and only once the eligible pool comfortably exceeds 17. Turning the draw on earlier reduces an already-thin pool for no security gain, since the draw does not defend against the attack that matters at small scale.

Enabling them in the other order is actively harmful. That deserves to be stated in §4.1 rather than left to whoever writes the governance proposal.

### D. Recount the quorum floor against participation, not a constant

Considered and **not recommended.** A floor expressed as a fraction of the eligible pool (say ⅓) self-adjusts, but it lets a small pool ratify a decision on 1–2 reveals, which is the exact regression #123 and #124 were filed to close. The constant 3 is right; the pool is what needs attention.

## What is not proposed

- **No change to `MIN_ARBITRATORS = 3` or `MAX_ARBITRATORS = 7.**` 3-of-7 gives a genuine majority that still resolves with absentees, and nothing in this analysis argues against it. The task asked for them to be revisited; they were, and they hold.
- **No admission control on who may be an arbitrator.** That contradicts the protocol's stance elsewhere (#111): a dishonest participant should gain nothing, not be kept out.

## Verification note

The pool-floor arithmetic above is read directly from the shipped constants: `MAX_BARRED_ARBITRATORS = MAX_ARBITRATORS * (MAX_DISPUTE_ROUNDS - 1)` in `escrow/state.rs`, and the `counted < MIN_ARBITRATORS` check in `execute_dispute_outcome.rs`. It had **not** been demonstrated against a running validator when this was written; it has been since — see below.

---

## What was built

Signed off and implemented in `openfiat-core` (task #125). A and C shipped. B and D did not, as recommended.

### A — the pool floor, as two separable pieces

The proposal was one thing; it turned out to be two, and only one of them can be built without an input the program cannot obtain. Both shipped, and keeping them apart is the whole of the honesty here.

**A1. Every terminal even split now records why.** `DisputeCase.terminal_reason` and a distinct `DisputeTerminalSplit` event, carrying one of four reasons:

| Reason | Meaning |
|---|---|
| `NoConsensus` | Enough arbitrators served and revealed; the weighted tally tied. Arbitration worked; the case was genuinely undecidable. |
| `QuorumNotReached` | The final round seated ≥ `MIN_ARBITRATORS` but counted fewer reveals. Seats taken and abandoned — the seat-squatting shape, surviving to the round budget. |
| `RoundUnstaffed` | The final round could not seat `MIN_ARBITRATORS` at all. Quorum was unreachable before a vote was cast. |
| `PoolExhausted` | The case stopped **short of** its round budget because the pool could not staff another round. |

The first three are derived from the case's own arrays and **need no pool size at all**, so they are reported on every deployment including ones that publish nothing. That is most of what §A asked for — "an operator seeing 'terminal split: pool exhausted' can act; one seeing three indecisive rounds cannot distinguish it from genuine disagreement" — delivered with no new trusted input.

**A2. Refusing to open a round the pool cannot staff.** `eligible pool >= MIN_ARBITRATORS + barred so far`, checked before re-opening; failing it resolves immediately with `PoolExhausted`. No payout changes — the split is identical in amount and destination, only earlier and recorded.

### The hard part, answered honestly

**A Solana program cannot count the arbitrator pool.** There is no way to enumerate `openfiat-staking`'s `StakeAccount`s from inside `execute_dispute_outcome`, and §A's inequality needs an *upper* bound on the pool to refuse a round. Three candidate sources, and why only one of them is usable:

1. **Derive it from participation across the case's own rounds** — the alternative this annex suggested. It does not work *for this purpose*, and the reason is structural rather than fixable: wallets that have taken a seat are a **lower** bound on the pool, and a lower bound is the wrong side of the inequality. Using it to refuse a round would end cases that were still decidable. It is used, but only to *raise* the estimate — see below.
2. **A counter on `staking::StakingConfig`**, moved as arbitrator stakes cross the minimum. This is the correct long-run source: exact, self-maintaining, no human in the loop. It was **not** built. It requires a layout migration on a live `StakingConfig` singleton and a change to every stake-lifecycle path in another program, which belongs to `openfiat-staking` rather than to this task.
3. **A governance attestation.** What shipped: a new singleton `ArbitrationPolicy` account, written by an admin-gated `publish_arbitrator_pool_size`.

So the number the floor uses is **asserted, not measured**, and that is a genuine weakening of §A rather than an implementation detail. Four things bound the damage, and they were the design:

- **Zero means unpublished and disables the floor entirely.** No cluster has the account today, so the shipped behaviour is byte-for-byte the old behaviour. Governance opts in; it does not arrive at upgrade time.
- **The published figure can never fall below what the case has witnessed.** The floor uses `max(published, seats taken in this case)`, so a stale-low number cannot end a case that is visibly attracting arbitrators. This is where the case's own participation is used — as a floor on the estimate, which is the direction a lower bound is sound in.
- **A wrong figure cannot block a case, only shorten it.** The outcome is the same even split either way; the failure mode is that it arrives sooner than the round budget would have delivered it. That is a real harm — sooner is what the griefing party wants — and it is why a deployment that cannot keep the figure current should leave it at zero.
- **The account is optional.** `execute_dispute_outcome` reads it as a trailing `remainingAccounts` entry, so a permissionless caller on a cluster that never created it resolves disputes exactly as before. The cost, stated plainly in the code: a caller can withhold it and suppress the floor. That buys them extra rounds of a frozen escrow, which is the status quo rather than a regression.

`ArbitrationPolicy` is shaped so that a staking-maintained counter can replace the attestation later without the floor arithmetic changing.

### C — the enabling order, written down

Recorded in two places rather than one, because whoever drafts the governance proposal and whoever reads the code are different people: beside the constants in `escrow/src/constants.rs` (on `RECOMMENDED_MIN_ARBITRATOR_STAKE_AGE_SECS` and `RECOMMENDED_ARBITRATOR_SORTITION_BPS`), and as a section of `programs/README.md`. Age gate first, 30 days after the first arbitrators stake rather than after genesis; sortition second, and only once the pool is comfortably above 17.

### Not built, as recommended

**B** (scaling the round budget to the pool) and **D** (a fractional quorum floor) were not implemented. Nothing in the build changed the argument against either.

### Two bugs found on the way

Both were found by checking whether this task's own change was safe, rather than by anyone looking for them — which is the argument for reviewing parameters together rather than one at a time.

#### The `DisputeCase` decoder read a verdict 8 bytes early

Checking that the two fields appended for this work were layout-safe for off-chain readers turned up that `crates/rpc/src/onchain_dispute.rs` was **already** decoding `DisputeCase` at the wrong offset: it skipped five vectors where the layout has six (`barred` was missing) and stopped before `deposit_shortfall`. So it read the `outcome` tag `4 + 32×barred.len() + 8` bytes early.

It failed safe by luck. With `barred` empty and no shortfall, the misread byte is the fifth byte of the `deposit` u64 — zero for any realistic deposit — so it decoded as "still running". **The moment a case re-opens a round, `barred` is non-empty** and the decoder reads into the middle of arbitrator-chosen pubkeys: a `1` byte there yields `Some(...)` and the next byte becomes the verdict. A node adopting a fabricated outcome for a case the chain never decided.

Its unit tests passed throughout, because the test fixture encoded the same wrong layout — two hand-written layouts agreeing with each other and neither agreeing with the program. Verified empirically against real `DisputeCase` accounts on a validator rather than by inspection, and fixed in `openfiat-core` `1a33b86`.

#### The barring cap was checked against a length that never moved

`MAX_BARRED_ARBITRATORS` is an Anchor `#[max_len]`, so overflowing `barred` does not truncate — it makes the account fail to serialize, leaving the escrow frozen with no instruction able to move it. The barring loop checked the cap against the *stored* length, which does not advance as the loop pushes, so a round starting at 13 barred could reach 20. Unreachable while `MAX_DISPUTE_ROUNDS` is 3 and only two rounds ever bar anyone — and live the moment either fact changes, which is exactly what happened when the terminal round started barring too. The guard now counts what the round is about to add. This is the concrete cost of the coupling this annex is about: the safe value of one parameter was a consequence of another parameter's value, and nothing said so.

### Demonstrated against a validator

The proof this annex asked for exists now, and it did not need 17 funded stake accounts. The cheapest honest instance of the floor is a published pool *below the quorum floor itself*: with one eligible arbitrator, no round of any case can reach three counted reveals, so the case must stop at the end of round 0 with `PoolExhausted` rather than bouncing to its budget. `programs/tests/arbitrator-pool-floor.ts` runs that against a real `solana-test-validator`, asserts the recorded reason **and** that the payout is the ordinary even split, and pairs it with the two boundary cases that matter more than the positive one: a pool exactly at the floor must still get its rounds, and a caller who omits the policy account must see the old behaviour unchanged.
