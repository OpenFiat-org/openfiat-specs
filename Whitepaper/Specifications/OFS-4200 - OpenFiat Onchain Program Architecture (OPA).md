# OFS-4200 — OpenFiat On-Chain Program Architecture (OPA)

**Document ID:** OFS-4200

**Title:** OpenFiat On-Chain Program Architecture

**Version:** 0.1.0 (**DRAFT — design-level, implementation may refine details**)

**Status:** Draft

**Category:** Infrastructure

**Depends On:** OFS-4100, OFS-2300, OFS-2400, OFS-4000, OFS-1600

---

## Status Banner

The whitepaper and protocol specifications describe on-chain *behavior* (vault types, state machines, PDA custody) in detail but define **no PDA seed scheme, account layout, instruction set, or program boundary** anywhere. This document supplies that missing design layer. It is implementation guidance, not a byte-level IDL — exact account field ordering/discriminators are decided in code, not here.

**Devnet-only.** Every program ID, mint address, and cluster reference produced under this design is a devnet artifact. Mainnet deployment is a separate future phase gated on an external security audit and final (non-draft) sign-off of OFS-4100.

## 1. Program Boundary Decision

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
- `openfiat-staking`'s `slash` instruction is **not** called via direct CPI from `openfiat-escrow`'s dispute-resolution path. It is invoked as a **separate, off-chain-relayed instruction**, triggered when the deterministic outcome of a dispute's decentralized commit-reveal arbitrator vote (OFS-2400 §16, Chapter 11 §11.12-11.16) identifies an arbitrator whose revealed vote fell outside case consensus. This deliberately avoids tight on-chain coupling between escrow and staking before either has been audited — a bug in one program's CPI-calling code cannot directly corrupt the other's state.

**Note on v1 arbitration model:** dispute resolution in OpenFiat v1 is decentralized from launch — any sufficiently staked, eligible participant may voluntarily join a case and vote via commit-reveal (OFS-2400 §16). It is not a small governance-appointed committee; the programs below are designed for many independent arbitrators per case, not a fixed authority list.

## 2. Shared Types (`programs/shared`)

A small crate consumed by all four programs to avoid duplicated definitions:

```rust
pub enum Role { Merchant, Arbitrator, NodeOperator, NotificationProvider, OracleProvider, RiskIntelligenceProvider, SnapshotProvider }
pub enum ProposalCategory { Informational, Standards, Parameter, Treasury, ProtocolUpgrade, Constitutional }
pub enum VaultState { Available, Reserved, AwaitingFiatSettlement, Released, Cancelled, Frozen }
pub enum DisputeOutcome { BuyerWins, MerchantWins, MutualSettlement, InvalidDispute }
```

## 3. `openfiat-presale` (Phase 3 — priority)

### PDA seeds

- Sale config: `[b"sale_config", sale_nonce.to_le_bytes()]` — `sale_nonce: u64` namespaces independent sale rounds under one deployed program. v1 production usage is a single sale at `sale_nonce = 0`; this is not a hard global singleton so a future round doesn't require a redeploy. (Implementation note, added during Phase 3: the original design here specified a fixed singleton seed with no nonce — that turned out to make even test-suite coverage of the soft-cap-missed/refund branch impossible without a second deployment, since `SaleState` transitions out of `Active` are one-way. The nonce is the fix.)
- USDC escrow vault: `[b"sale_usdc_vault", sale_nonce.to_le_bytes()]`.
- Contribution record: `[b"contribution", sale_config, buyer_pubkey]` — one per buyer per sale, prevents double-counting. (Already disambiguated per-sale since `sale_config`'s own address differs per nonce.)

### Account layouts (field-level)

- `SaleConfig { admin: Pubkey, open_mint: Pubkey, usdc_mint: Pubkey, presale_vault: Pubkey, usdc_vault: Pubkey, treasury: Pubkey, swap_program: Pubkey, hard_cap: u64, soft_cap: u64, min_contribution: u64, max_contribution: u64, max_slippage_bps: u16, open_decimals: u8, usdc_decimals: u8, start_time: i64, end_time: i64, stablecoin_whitelist: Vec<Pubkey>, total_raised: u64, state: SaleState, bump: u8, usdc_vault_bump: u8 }`
- `SaleState { Active, Finalized, SoftCapMissed }` — no separate `Pending` state; a sale is `Active` from `initialize_sale` onward and simply rejects contributions outside `[start_time, end_time]`.
- `swap_program` is admin-set at `initialize_sale`, not hardcoded to Jupiter's program id in code — production devnet/mainnet deployments must set it to Jupiter's real, independently verified aggregator program id (`JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4` as of Phase 3); test/CI deployments point it at a deterministic mock instead (`programs/mock-jupiter`). See `contribute_with_swap`'s doc comment for why this is safe: the instruction never trusts anything about the configured program's internal account layout, only that (a) the pubkey matches and (b) the destination vault's balance increased by at least the required amount after the CPI returns.
- `Contribution { buyer: Pubkey, usdc_amount: u64, open_entitlement: u64, claimed: bool, refunded: bool, bump: u8 }`

### Instructions

- `initialize_sale` — admin-only, sets `SaleConfig` from OFS-4100 §3's signed-off numbers.
- `contribute(input_mint, input_amount)` — validates `input_mint` is SOL, USDC, or on the whitelist and the sale is `Active`; if `input_mint != usdc_mint`, CPIs into Jupiter's aggregator with a `minimum_out` computed from `max_slippage_bps`, aborting the whole instruction (not just the swap) if the minimum isn't met; applies the 1:1 rate to the resulting USDC amount; enforces per-wallet min/max and the hard cap on the **post-swap** amount; updates or creates the buyer's `Contribution` PDA and `total_raised`.
- `finalize_sale` — admin-only, callable after `end_time` or `total_raised >= hard_cap`; if `total_raised < soft_cap`, sets state to `SoftCapMissed` instead of `Finalized`; otherwise sweeps USDC to the treasury multisig and sets `Finalized`.
- `claim` — callable only when `Finalized`; transfers `open_entitlement` to the buyer (immediate, no vesting per OFS-4100 §3); rejects if already claimed.
- `refund` — callable only when `SoftCapMissed`; returns the buyer's USDC-equivalent contribution; rejects if already refunded or already claimed.

### Non-negotiable invariants

- A `contribute` call either fully succeeds (swap + credit) or fully fails — no instruction may leave a contributor's input tokens in an intermediate, program-controlled-but-uncredited state.
- `total_raised` must never exceed `hard_cap`, even under concurrent contributions racing near the cap (Solana's per-account write-lock on `SaleConfig` inside one instruction makes this straightforward — every `contribute` call takes a write-lock on the shared config, so concurrent attempts serialize rather than race).

## 4. `openfiat-escrow`

### PDA seeds

- Liquidity Vault: `[b"liquidity_vault", merchant_pubkey, stablecoin_mint]`
- Trade Escrow Vault: `[b"trade_escrow", reservation_id]` (reservation_id: u64, assigned by the off-chain `reservations` crate and passed in)
- Fee config: `[b"fee_config"]` — singleton, governance-updatable.

### Account layouts

- `LiquidityVault { merchant: Pubkey, mint: Pubkey, total: u64, reserved: u64, available: u64, settled: u64, pending_settlement: u64, bump: u8 }`
- `TradeEscrowVault { reservation_id: u64, buyer: Pubkey, seller: Pubkey, mint: Pubkey, amount: u64, state: VaultState, dispute_authority: Pubkey, created_at: i64, timeout_at: i64, bump: u8 }`
- `FeeConfig { ad_listing_fee: u64, dispute_filing_fee: u64, dev_treasury: Pubkey, ecosystem_treasury: Pubkey, infra_treasury: Pubkey, emergency_reserve: Pubkey, bump: u8 }`

### Instructions

`create_liquidity_vault`, `deposit_liquidity`, `reserve_liquidity` (atomic marking, no transfer), `withdraw_liquidity`, `create_trade_escrow`, `fund_trade_escrow`, `approve_settlement`, `release_escrow` (only instruction that moves settlement funds; computes and routes fee splits from `FeeConfig` atomically), `cancel_reservation`, `expire_reservation` (permissionless, callable once `timeout_at` has passed — default 30 minutes per OFS-2300 §8a's timeout matrix, itself a `FeeConfig`-adjacent config value), `freeze_on_dispute` (settable only by the configured `dispute_authority` — a PDA controlled by `openfiat-disputes`'s case-resolution logic, not a human-held key; Phase 5 wires this to the real commit-reveal consensus outcome per OFS-2400 §16), `execute_dispute_outcome(outcome: DisputeOutcome)` (callable only once a case's commit-reveal consensus has been reached and tallied).

### Non-negotiable invariant

Every instruction that moves vault funds signs via `invoke_signed` using the vault's own PDA seeds — there is no code path where a human keypair authorizes a vault fund transfer. A dedicated test must attempt (and fail) a direct transfer from a vault token account outside the program's own instructions.

## 5. `openfiat-staking`

### PDA seeds

- Stake account: `[b"stake", owner_pubkey, role_as_u8]`
- Staking config: `[b"staking_config"]` — singleton, governance-updatable (per-role minimums, unbonding period, slash %).

### Account layouts

- `StakeAccount { owner: Pubkey, role: Role, amount: u64, unbonding_amount: u64, unbonding_release_at: i64, slashed_total: u64, bump: u8 }`
- `StakingConfig { min_stake: u64, min_stake_arbitrator: u64, unbonding_period_secs: i64, slash_bps: u16, slashing_authority: Pubkey, bump: u8 }`

### Instructions

`initialize_stake_account(role)`, `stake(amount)`, `request_unstake(amount)` (immediately reduces the amount counted as "effective" for `get_effective_stake`, per OFS-4100 §4's timing decision, while the tokens themselves remain locked until `unbonding_release_at`), `withdraw_unstaked`, `slash(stake_account, misconduct_code)` (callable only by `slashing_authority`; for arbitrator misconduct specifically, this is invoked as a separate signed instruction relayed by the off-chain `disputes` crate once a case's commit-reveal vote has been tallied and an arbitrator's revealed vote is found outside consensus — a partial, moderate slash per Chapter 11 §11.16, not a discretionary committee decision, and not a direct escrow→staking CPI), `get_effective_stake` (read-only, called via CPI by `openfiat-governance`; also determines arbitrator eligibility to join a dispute case per OFS-2400 §16).

## 6. `openfiat-governance`

### PDA seeds

- Proposal: `[b"proposal", proposal_id_le_bytes]`
- Vote record: `[b"vote", proposal_pubkey, voter_pubkey]` — existence of this PDA is itself the double-vote guard.
- Governance config: `[b"governance_config"]` — singleton (quorum %, thresholds per category, deposit amount, vote-lock duration).

### Account layouts

- `Proposal { id: u64, category: ProposalCategory, proposer: Pubkey, stake_deposit: u64, votes_for: u64, votes_against: u64, quorum_snapshot: u64, created_at: i64, voting_ends_at: i64, state: ProposalState, bump: u8 }`
- `ProposalState { Draft, Voting, Accepted, Rejected }`
- `VoteRecord { proposal: Pubkey, voter: Pubkey, weight: u64, in_favor: bool, locked_until: i64, bump: u8 }`
- `GovernanceConfig { quorum_bps: u16, threshold_simple_bps: u16, threshold_treasury_bps: u16, threshold_upgrade_bps: u16, quorum_upgrade_bps: u16, deposit_amount: u64, vote_lock_secs: i64, bump: u8 }`

### Instructions

`create_proposal(category, title_hash, summary_hash)` (full title/summary text lives off-chain per the OpenFiat network's existing gossip/session-sync machinery — only content hashes are recorded on-chain, matching the whitepaper's off-chain-messaging/on-chain-anchoring split used elsewhere), `cast_vote(in_favor)` (reads effective stake from `openfiat-staking` via CPI at cast time), `tally_and_finalize` (permissionless after `voting_ends_at`), `refund_or_forfeit_deposit`, `update_config_parameter(target_program, parameter_key, new_value)` (only callable when the calling proposal is `Accepted` and `category == Parameter`), `authorize_treasury_spend` (only when `Accepted` and `category == Treasury`).

### Forbidden-action checklist (design-review gate, not a single test)

Confirm no instruction, in any combination, can: confiscate a user's funds outside a resolved dispute or a proven-misconduct slash; reverse a `Released` trade; alter a finalized `VoteRecord` or a closed dispute's recorded outcome; move funds from any vault/stake account to an arbitrary destination not enumerated in this document's instruction set.

## 7. Config Accounts Naming Token Destinations

A configuration field that names somewhere tokens will be sent MUST be written by an
instruction that takes it as a **validated token account**, not as a bare `Pubkey`
parameter.

This is not a style preference. Every instruction that later *spends* to such a field
constrains it as `InterfaceAccount<'info, TokenAccount>` and requires the stored
pubkey to equal that account's address. If the field was populated from a raw
`Pubkey`, nothing stopped a wallet address being stored there — and a wallet is not a
token account, so the constraint can never be satisfied and the spending instruction
becomes permanently unexecutable.

The failure is invisible in three ways that matter:

* **Tests pass.** Integration tests create their own correctly-typed accounts, so the
  code path is exercised and correct. Only the deployed configuration is broken.
* **The write succeeds.** Storing the wrong kind of address is a valid transaction;
  nothing fails until something first tries to spend.
* **It presents as a missing feature.** The symptom is "this never happens", which
  reads as unimplemented rather than misconfigured.

Four fields in this workspace were populated this way and had to be corrected after
deployment: `FeeConfig`'s four treasury destinations, `StakingConfig`'s
`slash_destination` and `rewards_authority`, and `GovernanceConfig`'s
`forfeit_destination`. In each case the settlement fee, arbitrator slashing, reward
distribution and deposit forfeiture respectively could not execute at all.

`openfiat-presale` is the counter-example and the pattern to copy: `initialize_sale`
takes `presale_vault`, `usdc_vault` and `treasury` as `InterfaceAccount<TokenAccount>`
with mint and owner constraints applied at write time, so storing a wallet is not
expressible. `update_fee_config` was written the same way when the escrow fee path was
repaired, and later update instructions follow it.

**An authority field has the same hazard in a different form.** `rewards_authority`
must *sign*, so a `Pubkey` is the correct type — but an address for which no keypair
exists is equally dead, and equally silent. Such fields MUST be rejected if zero, and
SHOULD be verified as controllable before a deployment is considered complete.

### 7.1 How to decode a deployed account

Verification that catches this class is reading the deployed account back and decoding
it. A green test suite does not. But decoding has its own failure mode, and the weaker
form of this rule is not enough.

Decode by **walking the struct definition field by field**, and assert that the walk
consumes the whole account minus its trailing bumps. Do not read at hand-computed
offsets.

A hand-computed offset that skips a field produces a decode that *looks* correct: every
subsequent field is read from the wrong place, but each still yields a well-formed value
of the right type, so nothing appears wrong. Two 32-byte pubkeys adjacent in a struct
will happily swap identities and both still decode as valid addresses. The byte count is
what catches it — an omitted field leaves exactly its own width unexplained at the end.

This is not hypothetical. `StakingConfig`'s `slashing_authority` was omitted from such a
walk, shifting `slash_destination` and `rewards_authority` each one field along. The
result was a confident report that reward distribution was permanently broken — it was
not — and that the slash destination was a controllable key — it was not. Both
conclusions were wrong, in opposite directions, from one missing field. It was caught
only because someone re-derived the values independently rather than trusting the
report.

Applying the byte-count assertion afterwards confirmed the project's other three config
decodes were sound: `FeeConfig` consumed 202 of 203 bytes, `StakingConfig` 234 of 237,
`GovernanceConfig` 138 of 140, each remainder being exactly the declared bumps. The
check takes one line and is the difference between a decode you can cite and one you
merely believe.

## 8. Testing Environment Note

`anchor test` runs against `solana-test-validator`. Since `openfiat-presale` CPIs into Jupiter's real aggregator program, the test environment needs either a cloned copy of Jupiter's on-chain program (via `solana-test-validator --clone <jupiter_program_id> --url devnet`) or a purpose-built mock with the same instruction interface for deterministic CI runs. This should be verified as the first task of Phase 3, since it determines how automatable this phase's testing actually is.
