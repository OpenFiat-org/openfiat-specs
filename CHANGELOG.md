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
- OFS-2400 §16.2: the chain arbitrates and the off-chain registry only
  collects, replicates and records what it observed executed.
- OFS-8200 §7.1–§7.3: public trade reads are redacted, wallet-proof reads
  return a party's own records, and the three reads whose honest answer is
  a shape rather than a number.
- OFS-7000 §12.1: a lapsed feed and an unpriced pair are different answers.
- OFS-1100 §18.1 and OFS-2300 §24.1: reading a node's own discovery state,
  and settlement reads that do not name the parties.
- Initial repository scaffold: directory layout, CI, developer tooling,
  and community health files.

[Unreleased]: https://github.com/OpenFiat-org/openfiat-specs/commits/main
