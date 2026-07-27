# OFS-4200 — OpenFiat On-Chain Program Architecture (OPA)

**Document ID:** OFS-4200

**Title:** OpenFiat On-Chain Program Architecture

**Version:** 0.1.0 (**DRAFT — design-level, implementation may refine details**)

**Status:** Draft

**Category:** Infrastructure

**Depends On:** OFS-4100, OFS-2300, OFS-2400, OFS-4000, OFS-1600

---

# Status Banner

The whitepaper and protocol specifications describe on-chain *behavior* (vault types, state machines, PDA custody) in detail but define **no PDA seed scheme, account layout, instruction set, or program boundary** anywhere. This document supplies that missing design layer. It is implementation guidance, not a byte-level IDL — exact account field ordering/discriminators are decided in code, not here.

**Devnet-only.** Every program ID, mint address, and cluster reference produced under this design is a devnet artifact. Mainnet deployment is a separate future phase gated on an external security audit and final (non-draft) sign-off of OFS-4100.

# 1. Program Boundary Decision

Four devnet Anchor programs, plus the OPEN token itself as a plain SPL Token-2022 mint with no custom program:

| Program | Custody responsibility | Rationale |
|---|---|---|
| `openfiat-presale` | Contributor USDC/SOL/stablecoin funds during the sale window | Time-boxed, first to ship (priority), isolated so a presale-specific bug can't touch escrow or staking funds |
| `openfiat-escrow` | In-flight trade funds (Liquidity Vaults, Trade Escrow Vaults) | Holds third-party stablecoins in flight — highest transaction volume, most frequently exercised |
| `openfiat-staking` | Long-held OPEN stake per role | Different custody duration/risk profile than escrow (months vs. minutes) |
| `openfiat-governance` | No asset custody beyond small proposal-stake deposits | Lowest asset risk, highest "what can this program authorize" risk (parameter changes, treasury spend authorization) |

**Why not one monolithic program:** collapsing all four into one program means a single bug's blast radius covers presale contributor funds, in-flight trade funds, staked capital, and governance-authorized treasury spend simultaneously. Splitting them means each is independently auditable and a bug in one cannot directly corrupt another's state.

**Why not more granular** (e.g. separate programs per vault type, per role): the whitepaper's own separation of concerns is escrow-vs-staking-vs-governance-vs-presale, not finer than that; more programs means more cross-program calls, more PDA-derivation surface, and more deployment/upgrade coordination overhead without a corresponding security benefit at this scale.

**Cross-program coupling — decoupled where risk-relevant, coupled where safe:**
- `openfiat-governance`'s `cast_vote` reads `openfiat-staking`'s effective-stake **via CPI** (read-only, low risk).
- `openfiat-staking`'s `slash` instruction is **not** called via direct CPI from `openfiat-escrow`'s dispute-resolution path. It is invoked as a **separate, off-chain-relayed instruction** signed by the same governance-controlled authority that resolved the dispute. This deliberately avoids tight on-chain coupling between escrow and staking before either has been audited — a bug in one program's CPI-calling code cannot directly corrupt the other's state.

# 2. Shared Types (`programs/shared`)

A small crate consumed by all four programs to avoid duplicated definitions:

```rust
pub enum Role { Merchant, Arbitrator, NodeOperator, NotificationProvider, OracleProvider, RiskIntelligenceProvider, SnapshotProvider }
pub enum ProposalCategory { Informational, Standards, Parameter, Treasury, ProtocolUpgrade, Constitutional }
pub enum VaultState { Available, Reserved, AwaitingFiatSettlement, Released, Cancelled, Frozen }
pub enum DisputeOutcome { BuyerWins, MerchantWins, MutualSettlement, InvalidDispute }
```

# 3. `openfiat-presale` (Phase 3 — priority)

## PDA seeds
- Sale config: `[b"sale_config"]` — singleton.
- Contribution record: `[b"contribution", sale_config, buyer_pubkey]` — one per buyer, prevents double-counting.

## Account layouts (field-level)
- `SaleConfig { admin: Pubkey, usdc_mint: Pubkey, open_mint: Pubkey, hard_cap: u64, soft_cap: u64, min_contribution: u64, max_contribution: u64, start_time: i64, end_time: i64, max_slippage_bps: u16, stablecoin_whitelist: Vec<Pubkey>, total_raised: u64, state: SaleState, bump: u8 }`
- `SaleState { Pending, Active, Finalized, SoftCapMissed }`
- `Contribution { buyer: Pubkey, usdc_amount: u64, open_entitlement: u64, claimed: bool, refunded: bool, bump: u8 }`

## Instructions
- `initialize_sale` — admin-only, sets `SaleConfig` from OFS-4100 §3's signed-off numbers.
- `contribute(input_mint, input_amount)` — validates `input_mint` is SOL, USDC, or on the whitelist and the sale is `Active`; if `input_mint != usdc_mint`, CPIs into Jupiter's aggregator with a `minimum_out` computed from `max_slippage_bps`, aborting the whole instruction (not just the swap) if the minimum isn't met; applies the 1:1 rate to the resulting USDC amount; enforces per-wallet min/max and the hard cap on the **post-swap** amount; updates or creates the buyer's `Contribution` PDA and `total_raised`.
- `finalize_sale` — admin-only, callable after `end_time` or `total_raised >= hard_cap`; if `total_raised < soft_cap`, sets state to `SoftCapMissed` instead of `Finalized`; otherwise sweeps USDC to the treasury multisig and sets `Finalized`.
- `claim` — callable only when `Finalized`; transfers `open_entitlement` to the buyer (immediate, no vesting per OFS-4100 §3); rejects if already claimed.
- `refund` — callable only when `SoftCapMissed`; returns the buyer's USDC-equivalent contribution; rejects if already refunded or already claimed.

## Non-negotiable invariants
- A `contribute` call either fully succeeds (swap + credit) or fully fails — no instruction may leave a contributor's input tokens in an intermediate, program-controlled-but-uncredited state.
- `total_raised` must never exceed `hard_cap`, even under concurrent contributions racing near the cap (Solana's per-account write-lock on `SaleConfig` inside one instruction makes this straightforward — every `contribute` call takes a write-lock on the shared config, so concurrent attempts serialize rather than race).

# 4. `openfiat-escrow`

## PDA seeds
- Liquidity Vault: `[b"liquidity_vault", merchant_pubkey, stablecoin_mint]`
- Trade Escrow Vault: `[b"trade_escrow", reservation_id]` (reservation_id: u64, assigned by the off-chain `reservations` crate and passed in)
- Fee config: `[b"fee_config"]` — singleton, governance-updatable.

## Account layouts
- `LiquidityVault { merchant: Pubkey, mint: Pubkey, total: u64, reserved: u64, available: u64, settled: u64, pending_settlement: u64, bump: u8 }`
- `TradeEscrowVault { reservation_id: u64, buyer: Pubkey, seller: Pubkey, mint: Pubkey, amount: u64, state: VaultState, dispute_authority: Pubkey, created_at: i64, timeout_at: i64, bump: u8 }`
- `FeeConfig { ad_listing_fee: u64, dispute_filing_fee: u64, dev_treasury: Pubkey, ecosystem_treasury: Pubkey, infra_treasury: Pubkey, emergency_reserve: Pubkey, bump: u8 }`

## Instructions
`create_liquidity_vault`, `deposit_liquidity`, `reserve_liquidity` (atomic marking, no transfer), `withdraw_liquidity`, `create_trade_escrow`, `fund_trade_escrow`, `approve_settlement`, `release_escrow` (only instruction that moves settlement funds; computes and routes fee splits from `FeeConfig` atomically), `cancel_reservation`, `expire_reservation` (permissionless, callable once `timeout_at` has passed — default 20 minutes per OFS-2300's example, itself a `FeeConfig`-adjacent config value), `freeze_on_dispute` (settable only by the configured `dispute_authority`, a placeholder pubkey until Phase 5 wires the real arbitrator-set config), `execute_dispute_outcome(outcome: DisputeOutcome)`.

## Non-negotiable invariant
Every instruction that moves vault funds signs via `invoke_signed` using the vault's own PDA seeds — there is no code path where a human keypair authorizes a vault fund transfer. A dedicated test must attempt (and fail) a direct transfer from a vault token account outside the program's own instructions.

# 5. `openfiat-staking`

## PDA seeds
- Stake account: `[b"stake", owner_pubkey, role_as_u8]`
- Staking config: `[b"staking_config"]` — singleton, governance-updatable (per-role minimums, unbonding period, slash %).

## Account layouts
- `StakeAccount { owner: Pubkey, role: Role, amount: u64, unbonding_amount: u64, unbonding_release_at: i64, slashed_total: u64, bump: u8 }`
- `StakingConfig { min_stake: u64, min_stake_arbitrator: u64, unbonding_period_secs: i64, slash_bps: u16, slashing_authority: Pubkey, bump: u8 }`

## Instructions
`initialize_stake_account(role)`, `stake(amount)`, `request_unstake(amount)` (immediately reduces the amount counted as "effective" for `get_effective_stake`, per OFS-4100 §4's timing decision, while the tokens themselves remain locked until `unbonding_release_at`), `withdraw_unstaked`, `slash(stake_account, misconduct_code)` (callable only by `slashing_authority` — v1: a governance-controlled multisig representing the trusted arbitrator committee, invoked via a separate signed instruction relayed by the off-chain `disputes`/`governance` crates, not a direct escrow→staking CPI), `get_effective_stake` (read-only, called via CPI by `openfiat-governance`).

# 6. `openfiat-governance`

## PDA seeds
- Proposal: `[b"proposal", proposal_id_le_bytes]`
- Vote record: `[b"vote", proposal_pubkey, voter_pubkey]` — existence of this PDA is itself the double-vote guard.
- Governance config: `[b"governance_config"]` — singleton (quorum %, thresholds per category, deposit amount, vote-lock duration).

## Account layouts
- `Proposal { id: u64, category: ProposalCategory, proposer: Pubkey, stake_deposit: u64, votes_for: u64, votes_against: u64, quorum_snapshot: u64, created_at: i64, voting_ends_at: i64, state: ProposalState, bump: u8 }`
- `ProposalState { Draft, Voting, Accepted, Rejected }`
- `VoteRecord { proposal: Pubkey, voter: Pubkey, weight: u64, in_favor: bool, locked_until: i64, bump: u8 }`
- `GovernanceConfig { quorum_bps: u16, threshold_simple_bps: u16, threshold_treasury_bps: u16, threshold_upgrade_bps: u16, quorum_upgrade_bps: u16, deposit_amount: u64, vote_lock_secs: i64, bump: u8 }`

## Instructions
`create_proposal(category, title_hash, summary_hash)` (full title/summary text lives off-chain per the OpenFiat network's existing gossip/session-sync machinery — only content hashes are recorded on-chain, matching the whitepaper's off-chain-messaging/on-chain-anchoring split used elsewhere), `cast_vote(in_favor)` (reads effective stake from `openfiat-staking` via CPI at cast time), `tally_and_finalize` (permissionless after `voting_ends_at`), `refund_or_forfeit_deposit`, `update_config_parameter(target_program, parameter_key, new_value)` (only callable when the calling proposal is `Accepted` and `category == Parameter`), `authorize_treasury_spend` (only when `Accepted` and `category == Treasury`).

## Forbidden-action checklist (design-review gate, not a single test)
Confirm no instruction, in any combination, can: confiscate a user's funds outside a resolved dispute or a proven-misconduct slash; reverse a `Released` trade; alter a finalized `VoteRecord` or a closed dispute's recorded outcome; move funds from any vault/stake account to an arbitrary destination not enumerated in this document's instruction set.

# 7. Testing Environment Note

`anchor test` runs against `solana-test-validator`. Since `openfiat-presale` CPIs into Jupiter's real aggregator program, the test environment needs either a cloned copy of Jupiter's on-chain program (via `solana-test-validator --clone <jupiter_program_id> --url devnet`) or a purpose-built mock with the same instruction interface for deterministic CI runs. This should be verified as the first task of Phase 3, since it determines how automatable this phase's testing actually is.
