# Changelog

All notable changes to `openfiat-specs` will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- OFS-2100 §6.1, §12.1, §20.1, §24.1: an advertisement names a mint rather
  than a ticker; the floating-price formula, its precision and its rounding;
  filtered, cursor-paged listing; and why a node must not reject a mint it
  does not itself recognize.
- OFS-2200 §7.1: a reservation records the price it was made at, validated
  as arithmetic against the advertisement's own terms rather than against
  the validating node's oracle view.
- OFS-2400 §14, §16.2 and §17: the chain arbitrates and the off-chain
  registry only collects, replicates and records what it observed executed
  — with no exception for a mutual settlement, which is recorded as agreed
  and still waits for the escrow to move. The state is named
  `Awaiting Chain Execution`, since it is reached both by a completed
  reveal phase and by party agreement, and only one of those is a verdict.
- OFS-8200 §7.1–§7.3: public trade reads are redacted, wallet-proof reads
  return a party's own records, and the three reads whose honest answer is
  a shape rather than a number.
- OFS-1300 §9 and OFS-1700 §10: a snapshot is tagged with the Solana slot
  its state is current as of, not a locally-invented height. A per-producer
  counter is not comparable between producers, so ordering by it ordered
  nothing and the anti-rollback check compared two different quantities. A
  borrowed clock is also checkable, which a self-reported one never is.
  States plainly what a slot does not assert: recency, not containment.
- OFS-2100 §12: drops two fields no implementation has and none should.
  A Discount is a negative Premium — the field is signed — and two ways to
  say one thing can contradict. A Refresh Frequency is a claim a merchant
  is not in a position to make: the price resolves at read time and the
  oracle record carries its own expiry. Adds Price Decimals, which is real
  and was missing.
- OFS-2400 §8.1: the dispute statuses, enumerated. §16.2 had been
  describing `Awaiting Chain Execution` as an addition to a list that did
  not exist.
- OFS-2400 §21.1: `ResolutionIssued` and `EscrowReleased` were listed as
  gossip events beside a paragraph explaining that they are not. Moved to
  their own section as chain observations — listing them as messages
  invited the implementation the protocol forbids.
- OFS-8200 §7.2.1: the six wallet-proof surfaces and the domain each signs
  under. The mechanism was specified in full without naming a single method
  that uses it, and the subject is not always a wallet.
- OFS-7000 §12.1: a lapsed feed and an unpriced pair are different answers.
- OFS-1100 §18.1 and OFS-2300 §24.1: reading a node's own discovery state,
  and settlement reads that do not name the parties.
- OFS-8200 §4.1: every key, peer id, signature and event id is base58 in
  JSON, never an array of integers. An Ed25519 private key is also 32
  bytes, so the array form gave a reader no way to tell a published
  identity from a leaked secret; it was also the only form nobody could
  paste into an entrypoint. Records that the encoding is part of the
  signed transcript, and that non-identity byte fields stay arrays.
- OFS-1500 §8: a Service ID derived from an identity SHALL use the whole
  public key. A truncated peer id looks unique and is not — the Ed25519
  multihash preamble is shared by every provider, so an eight-byte prefix
  carries two varying bytes and collides at a few hundred providers,
  displacing one provider's record with another's.
- OFS-1300 §15.1 and §15.2: a compressed snapshot is a gzip-compressed
  tar named `.tar.gz`, with the state in a member read by name so a later
  member is a compatible addition. States that `None` must stay
  acceptable, that an implementation must not announce a method it does
  not apply, and that the size limit belongs on the decompressed length —
  the compressed size says nothing about what it expands to.
- Initial repository scaffold: directory layout, CI, developer tooling,
  and community health files.

[Unreleased]: https://github.com/OpenFiat-org/openfiat-specs/commits/main
