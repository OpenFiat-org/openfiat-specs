# OFS-4100 Annex A — The arbitration parameters, reviewed as one system

**Status:** `[PROPOSED — NEEDS SIGN-OFF]`
**Scope:** the quorum floor, the seat count, the round budget, the barring rule and the sortition threshold — reviewed together, because reviewed apart they are individually defensible and jointly broken.

This annex exists because task #125 asked for the 3-of-7 quorum and the round budget to be revisited *together with* the sortition threshold. That framing turned out to be the whole finding: each parameter is sound on its own terms, and their interaction has a failure mode none of them shows in isolation.

## The parameters as they stand

| Parameter | Value | Where | Status |
|---|---|---|---|
| `MAX_ARBITRATORS` (seats per round) | 7 | `escrow/state.rs` | shipped |
| `MIN_ARBITRATORS` (quorum floor, counted reveals) | 3 | `escrow/state.rs` | shipped |
| `MAX_DISPUTE_ROUNDS` | 3 | `escrow/constants.rs` | `[PROPOSED]` |
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

Four options, not mutually exclusive. My recommendation is **A + C**.

### A. Make the pool floor explicit and enforced `[PROPOSED — NEEDS SIGN-OFF]`

A case should not open a round it cannot possibly resolve. Before opening round *n*, require

> eligible pool ≥ `MIN_ARBITRATORS` + (barred so far)

and if that fails, stop re-opening and go to the terminal split **immediately**, recording *why*. This changes no outcome — the split happens either way — but it stops the protocol pretending a decision was attempted, and it makes the condition observable rather than inferred from a case that quietly bounced three times. An operator seeing "terminal split: pool exhausted" can act; one seeing three indecisive rounds cannot tell this from genuine disagreement.

### B. Scale the round budget to the pool `[PROPOSED — NEEDS SIGN-OFF]`

Replace the constant 3 with `min(3, floor((pool − MIN_ARBITRATORS) / MAX_ARBITRATORS) + 1)`. A large pool keeps all three rounds; a small one gets fewer, which is honest — a second round it cannot staff is theatre. Downside: `MAX_BARRED_ARBITRATORS` is an Anchor `#[max_len]`, so the account layout must still be sized for the maximum. Sizing stays at 14; only the behaviour adapts.

### C. Enable the stake age gate before the sortition threshold `[PROPOSED — NEEDS SIGN-OFF]`

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

The pool-floor arithmetic above is read directly from the shipped constants: `MAX_BARRED_ARBITRATORS = MAX_ARBITRATORS * (MAX_DISPUTE_ROUNDS - 1)` in `escrow/state.rs`, and the `counted < MIN_ARBITRATORS` check in `execute_dispute_outcome.rs`. It has **not** been demonstrated against a running validator — doing so needs 17+ funded arbitrator stake accounts, which is a fixture worth building if option A or B is signed off, and is the test that would prove the fix rather than the analysis.
