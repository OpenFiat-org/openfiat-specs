# OFS-8200 — OpenFiat Node API (ONA)

**Document ID:** OFS-8200

**Title:** OpenFiat Node API

**Version:** 1.0.0 (Draft)

**Status:** Draft Standard

**Category:** Interface Specification

**Depends On:** OFS-1000, OFS-1200, OFS-1400, OFS-8000, OFS-8100

---

## Abstract

The OpenFiat Node API (ONA) defines the third-party-facing interface every OpenFiat node exposes: one JSON-RPC 2.0 endpoint, modeled directly on Solana's own JSON-RPC API rather than a REST resource hierarchy, plus one WebSocket endpoint streaming successful mutations as they happen.

This is the interface external developers, SDKs, trading bots, and service-provider integrations are meant to build against — the stable contract every other node-facing specification's internal gossip/registry mechanics are deliberately hidden behind.

ONA does not define new marketplace or protocol behavior. It defines how already-specified behavior (OFS-2100 through OFS-7100) is exposed for a caller to read and to submit already-signed mutations.

---

## 1. Introduction

Every prior specification in this suite (OFS-1000 through OFS-8100) defines how nodes behave among themselves: transport, gossip, registries, and the protocols that ride on top of them. None of them define how a caller outside the peer-to-peer network — a wallet application, a trading bot, a block explorer, a notification or oracle provider's own tooling — is meant to interact with a single node.

ONA closes that gap. It is intentionally the last specification in this suite to be written, because it has nothing of its own to specify beyond *access*: every data shape and state transition it exposes is already defined elsewhere.

## 2. Scope

This specification covers:

* The transport and envelope format for requests and responses.
* Method naming conventions and the distinction between read and write methods.
* How a write method's caller-signed payload relates to the same signed event format OFS-1200 (Gossip Protocol) already defines.
* The error model, built directly on OFS-8000 (Error Code Registry).
* The subscription (WebSocket) model for reacting to mutations without polling.
* Self-description: how a node publishes its own method catalog.

This specification does not cover:

* Any marketplace or protocol state machine — see OFS-2000 through OFS-7100.
* Peer-to-peer transport between nodes — see OFS-1000.
* Session state for authenticated clients — see OFS-1400, which this API's session-scoped methods delegate to directly.

## 3. Design Goals

* **Familiarity.** A developer who has integrated against Solana's JSON-RPC API should recognize this API's shape immediately: `getX`/`sendX` naming, one POST endpoint, opaque signed payloads for mutations.
* **No custom trust model.** The node never signs anything on a caller's behalf and never introduces a parallel credential system (API keys, OAuth). Every mutation is authenticated the same way a gossip-originated event is: the caller's own wallet signature, verified through the same path OFS-1200 §8 already specifies.
* **Self-description that cannot drift.** The method catalog a node publishes about itself MUST be generated from the same dispatch table the node actually runs, not maintained by hand alongside it.
* **Transport-agnostic method design.** Every method's shape is defined independently of whether a given deployment fronts it with a load balancer, a rate limiter, or additional authentication — this specification constrains the method contract, not node operators' own deployment topology.

## 4. Transport

```text
POST /rpc
Content-Type: application/json
```

The request and response bodies are JSON-RPC 2.0 envelopes exactly as defined by the JSON-RPC 2.0 specification: a request carries `jsonrpc`, `id`, `method`, and `params`; a response carries `jsonrpc`, `id`, and exactly one of `result` or `error`.

```json
{ "jsonrpc": "2.0", "id": 1, "method": "getAdvertisement", "params": { "id": "ad-1" } }
```

```json
{ "jsonrpc": "2.0", "id": 1, "result": { "id": "ad-1", "merchant": "...", "status": "Active" } }
```

A node MAY additionally expose `GET /health` (liveness only, no protocol semantics) and `GET /metrics` (operator-facing Prometheus text format, out of scope for this specification).

## 5. Method Naming

Method names are camelCase and fall into exactly two families, distinguished by their prefix:

* **`getX`** — a read. MUST NOT mutate node state that any other caller can observe, and MUST be safe to retry. Most reads are callable without any signature; the wallet-proof reads of §7.2 are the exception, and they are still reads — they mutate nothing but the single-use nonce they spend, which is why retrying one requires a fresh challenge rather than being idempotent. A read MUST NOT be given a `sendX` name because it requires a proof.
* **`sendX`** — a write. MUST take exactly one param, `data`: an opaque, base64-encoded, already-signed JSON payload the caller's own client produced locally — the domain's own `Signed*` envelope, JSON-serialized. The node's only job is to decode it and apply it through the identical validate → store → broadcast path OFS-1200 §8 defines for a gossip-received event — it never constructs, completes, or signs a payload for the caller. JSON is deliberately chosen here over the postcard-based wire format OFS-1200's gossip envelope carries internally: `sendX` is the one boundary a non-Rust SDK (TypeScript, Python, ...) has to cross, and a signature computed over a compact binary encoding only Rust's `serde` ecosystem produces byte-identically would make every other language re-implement that encoding just to sign anything. Every domain event's signature is computed over its own JSON encoding for exactly this reason — see the domain's own specification for the exact `Signed*` shape.

```json
{ "method": "getReservation", "params": { "id": "res-1" } }
{ "method": "sendReservationRequest", "params": { "data": "<base64 JSON bytes>" } }
```

No third method-name family is defined. A future specification that needs one (for example, a batched submission method) MUST NOT repurpose `getX`/`sendX` semantics for a different meaning.

**One documented exception:** `sendTransaction` (OFS-4300) is a `sendX` write whose `data` is a base64-encoded, already-signed **Solana** transaction, not an OpenFiat `Signed*` JSON envelope — Solana's own transaction-message signing scheme governs that payload, not this specification's JSON convention. It still satisfies `sendX`'s core contract (opaque, already-signed by the caller, the node never constructs or completes it) — only the payload's own signing scheme differs, because it's chain-native rather than protocol-native data.

## 6. Method Categories

Each domain specification already listed in "Depends On" contributes its own methods to one shared namespace. The canonical category list, by underlying specification:

| Domain | Governing spec | Representative methods |
| --- | --- | --- |
| Advertisements | OFS-2100 | `getAdvertisement`, `getAdvertisements` (filtered and paged, OFS-2100 §20.1), `sendAdvertisementCreate` |
| Reservations | OFS-2200 | `getReservation`, `getReservations`, `getMyReservations`, `sendReservationRequest` |
| Settlement | OFS-2300 | `getSettlement`, `getSettlements`, `getMySettlements`, `sendSettlementInitiate`, `sendPaymentSubmitted`, `sendSettlementApproved` |
| Trade (read-only join) | OFS-2000 | `getTrade`, `getTrades`, `getMyTrades` |
| Disputes | OFS-2400 | `getDispute`, `getDisputes`, `getMyDisputes`, `sendDisputeOpen`, `sendArbitratorJoin`, `sendVoteCommit`, `sendVoteReveal` |
| Identity | OFS-5000 | `getIdentityClaim`, `getIdentityClaimsByWallet`, `sendClaimPublish` |
| Reputation (read-only) | OFS-3000 | `getReputation` |
| Governance | OFS-4000 | `getProposal`, `getProposals`, `sendProposalCreate`, `sendVoteCast` |
| Service providers | OFS-1500 | `getProvider`, `getProviders`, `sendProviderRegister` |
| Notifications | OFS-6000 | `getSubscription`, `sendSubscriptionUpdate`, `sendDeliveryReport` |
| Oracles | OFS-7000 | `getOracleRecord`, `getExchangeRate`, `getMedianExchangeRate`, `sendOraclePublish` |
| Risk intelligence | OFS-7100 | `getWalletScreening`, `sendRiskPublish` |
| Snapshots | OFS-1300 | `getLatestSnapshot`, `getCheckpointHeight`, `sendSnapshotAnnounce` |
| Sessions | OFS-1400 | `getSession`, `sendSessionEstablish`, `sendSessionRenew`, `sendSessionRevoke`, `sendSessionMigrate` |
| Chain bridge | OFS-4300 | `getChainStatus`, `getLatestBlockhash`, `sendTransaction` |
| Node | OFS-1100 | `getVersion`, `getHealth`, `getPeers` |
| Wallet proof | — | `getWalletChallenge` (§7.2) |
| Network aggregates | OFS-2300 | `getSettledVolume` (§7.3) |

This table is illustrative, not normative on its own — the normative list for a given node is whatever its self-description (§9) reports, since that is generated from the node's own dispatch table.

A future domain specification (a new OFS-2xxx/6xxx/7xxx document) that wants an API surface MUST follow §5's naming convention and add itself to this table's spirit; this specification does not need to be revised for every new domain, the same way OFS-8100 (Event Type Registry) doesn't need revision for every new event name.

## 7. Authentication

There is no API-key, OAuth, or session-token authentication layer distinct from the rest of the protocol. Four authentication tiers apply, matching how much a method actually needs:

1. **Unauthenticated reads.** Most `getX` methods require no proof of identity — they return already-public gossip-replicated state, identical to what any peer already has. Where such a read touches a trade, it is redacted (§7.1).
2. **Signed writes.** Every `sendX` method's payload is itself a signed event (§5) — authentication is embedded in the payload the caller already produced, not a separate header or token.
3. **Session-scoped reads.** A small number of methods (account-scoped data, e.g. a wallet's own notification subscriptions) require an established session per OFS-1400; the caller presents the session identifier issued by `sendSessionEstablish`.
4. **Wallet-proof reads.** A caller who proves control of a wallet, by signing a nonce this node issued, reads that wallet's own records in full (§7.2).

Node operators MAY additionally require transport-level authentication (mTLS, an API gateway, an allowlist) for their own deployment — that is a deployment concern, not part of this specification's contract.

### 7.1 Public trade reads are redacted

`[PROPOSED — NEEDS SIGN-OFF]`

| Read | Public form |
|---|---|
| `getSettlement`, `getSettlements` | Everything except the parties and `payment_reference` |
| `getReservation`, `getReservations` | Everything except the requester |
| `getDispute`, `getDisputes` | Everything except the parties, the opening `reason`, the arbitrator-to-vote pairing, and the mutual-settlement flags. Seat, commitment and reveal **counts** survive |
| `getTrade`, `getTrades` | The join of the two redacted records above, with the derived trade status |
| `getMySettlements`, `getMyReservations`, `getMyDisputes`, `getMyTrades` | The whole record, to a proven party (§7.2) |

**What is being protected is a graph, not a secret.** An endpoint that answers "who does this wallet trade with, and how often" hands anyone a map of real trading relationships: which merchant a wallet always returns to, who a busy merchant's regulars are, and therefore who is worth following home. In a peer-to-peer fiat market that is a physical-safety question rather than a preference, and it is the reason the aggregate view of counterparties is gated behind a signing handshake.

That argument was made once and enforced in one method, while the same graph stayed reachable through the enumerating reads, none of which took a parameter and all of which named both parties. The gate was not weak; it was walked around. A specification that lists the redacted shapes without saying this teaches implementers to route around it in exactly the same way — the next method that joins two records, or adds a party field "for convenience", reopens it.

**Redaction rather than authentication, for the public reads.** An explorer showing settlement volume, states and timing is a legitimate public view of a public network. Putting a signature in front of that would break it while pushing anyone determined back to raw gossip, which achieves nothing. What an explorer never needed is *who*. So the public read keeps everything except identity.

**Every read in a family must be redacted together.** The enumerating method hands out every identifier, so a by-id read left whole is bypassed by iterating the list the other one returns. The same applies to any read that joins two records: a join of two redacted records is redacted, and composing it from the same redaction means a field added to either half cannot appear in the join without appearing in the half.

**A price is not an identity.** The number a reservation was struck at, its oracle mid, the amount, the state, the timings, and an on-chain signature anyone can already read on Solana all stay public: they say something about the trade rather than about the people. `payment_reference` is the clearest case in the other direction — it is free text a buyer fills in with their own bank reference, so it routinely carries a real name or an account number.

**The rule for adding a field.** A field belongs in a public view only if it describes the *trade* rather than the *people*. When in doubt it stays out: adding one later is a release note, and removing one is a disclosure that has already happened.

**What this is honestly worth.** These records are gossiped to every node, so anyone running one reads them all, and no amount of API gating changes that. What is protected is the *ease* of the query — the difference between `curl`-ing a stranger's public access node and standing up a node to index the network. That difference is most of what casual harvesting is made of. An implementation MUST NOT present redaction as confidentiality.

### 7.2 Wallet-proof reads

`[PROPOSED — NEEDS SIGN-OFF]`

| Parameter | Value |
|---|---|
| Challenge issuer | `getWalletChallenge`, taking the wallet and returning a subject, a nonce and an expiry |
| Nonce | 32 random bytes, single-use |
| Challenge lifetime | 300 seconds |
| Signed bytes | `<domain>:<subject>:<nonce>` |
| Domain separator | Per method — one gated surface's signature is never valid on another |
| Failure mode | Refusal |

A caller asks for a challenge, signs the exact bytes it names, and presents the wallet, its public key, the nonce and the signature. The node checks that the public key really derives the claimed wallet, consumes the nonce, and then verifies the signature.

**Issuing a challenge is deliberately open.** A nonce is worthless without the private key that signs it, and demanding a signature to obtain the thing you sign would be circular. Issuance confirms nothing about the wallet, not even that it exists.

**The order of checks is not arbitrary.** The derivation check comes first, so asking for a wallet you do not hold the key for is refused outright — and refusing early also means a stranger's failed attempt cannot spend the nonce its real owner is part-way through signing. The nonce is then consumed *before* the signature is verified, so presenting a captured signature burns the nonce rather than replaying it.

**Outstanding challenges MUST be keyed by nonce, not by subject.** Because issuance is an anonymous write, keying by subject lets a stranger request challenges for someone else's wallet in a loop and invalidate the one that wallet is part-way through signing, locking them out for as long as the attacker cares to keep it up — a denial of service requiring no credentials, no stake and no relationship to the victim. Keyed by nonce, an outstanding challenge is reachable only by someone who already knows its random bytes. The cost is that unanswered challenges accumulate rather than replacing each other, so an implementation MUST expire them and SHOULD bound how many may be outstanding.

**A refused challenge MUST NOT say which way it failed.** "Never existed" and "already spent" are reported identically; distinguishing them confirms that some other party is mid-handshake for that subject.

**One nonce answers exactly one method, once.** The nonce itself carries no domain — the separation is in what the caller signs — so the same nonce can answer any one gated surface depending on the domain the signature was made under, and can answer only one of them. Every wallet-proof method MUST therefore define its own domain separator, and MUST NOT share one with another method.

**Refusal, never narrowing.** A caller who cannot prove the wallet gets an error, not a filtered answer. An implementation that silently narrows looks identical in every passing test right up until a refactor drops the filter; a refusal fails loudly and immediately.

**A proven party learns nothing new about anyone else.** These methods return the caller's own records — the trades they are party to, and, for an arbitrator, the cases they are seated on. A party already knows who they traded with.

### 7.3 Reads that must state what they are not

`[PROPOSED — NEEDS SIGN-OFF]`

Three reads are worth specifying because their honest answer is a shape rather than a number.

**`getExchangeRate` is three-state.** A pair is either currently priced, or published-but-lapsed, or unpublished. `getMedianExchangeRate` returns a price or nothing and collapses the last two into the same empty answer, which is not enough to act on: a lapsed feed means a provider does publish this corridor and waiting is sensible, while an unpublished pair means nobody prices it and waiting is pointless. Neither is a number, and a caller MUST show neither as one. `getMedianExchangeRate` remains, because it is the right shape when all a caller wants is a price or nothing.

**`getSettledVolume` is per asset, and never a single total.** These are different tokens at different scales; one "total volume" figure adds one asset to another, and does it silently. Each entry therefore carries its own mint, its own decimals taken from that mint rather than assumed, and its own count. A settlement counts only once the node has independently observed its on-chain release confirm — everything looser measures intent rather than volume, and a sum over advertisements measures what merchants say they are willing to trade.

An asset is two hops from a settlement (settlement → reservation → advertisement → mint), so a settlement whose advertisement has since been removed (OFS-2100 §21) cannot be attributed at all. Those MUST be counted and reported rather than dropped: a figure that quietly omits what it could not classify looks complete and is not, and the count of unattributed records is exactly what tells a reader how far to trust the rest.

The response MUST carry an explicit statement of its own scope. One node reports what it has replicated, which is not necessarily the network's whole history — a node that joined last week, or one running a rolling retention window, honestly holds less. Stated in the response rather than in a footnote elsewhere, because a number presented without its scope reads as a global total.

**`getPeers` reports measurements, not a verdict.** It returns the peers this node has discovered with the fields a peer record genuinely carries, plus this node's own peer identity and the addresses it announces — an operator publishing an entrypoint needs both and has nowhere else to get them, and assembling them from a log line is how they get typed wrong. Success and failure counts are this node's own tally of its own exchanges; two honest nodes can disagree about both. An implementation SHOULD NOT fold them into an uptime percentage or a health score, which would present one node's local experience as a network-wide judgement.

## 8. Rate Limiting

`[PROPOSED — NEEDS SIGN-OFF]` This specification does not mandate a specific rate-limiting scheme or numeric threshold — matching this workspace's existing pattern for protocol parameters left to implementations (see `docs/architecture.md`'s other `[PROPOSED]` defaults). A node implementation MAY apply per-caller rate limiting at its own discretion; a future revision of this specification MAY adopt a normative default once operational experience across multiple independent node operators exists to inform one.

## 9. Self-Description

A node MUST publish a machine-readable description of every method it currently serves, generated directly from its own dispatch table — never maintained by hand alongside it, so the description cannot silently drift from what the node actually runs.

This specification does not mandate a specific description format; the reference implementation publishes an [OpenRPC](https://open-rpc.org) 1.2.6 document at `GET /openrpc.json` (the JSON-RPC equivalent of an OpenAPI/Swagger document for a REST API), plus a self-contained interactive reference page at `GET /docs` for browsing every method's parameter and result shape without external tooling.

## 10. Errors

Transport-level failures (malformed JSON, an unknown method, malformed parameters, an internal fault) use the standard JSON-RPC 2.0 error codes: `-32700` (parse error), `-32601` (method not found), `-32602` (invalid params), `-32603` (internal error).

Every domain-level failure — insufficient liquidity, a duplicate event, an unauthorized signer, any failure OFS-8000 (Error Code Registry) already assigns a numeric code — is reported as a single JSON-RPC error code, `-32000` ("Application error"), carrying OFS-8000's own numeric code and symbolic name in the error's `data` field:

```json
{
  "error": {
    "code": -32000,
    "message": "INSUFFICIENT_AVAILABLE_LIQUIDITY",
    "data": { "ofsErrorCode": 4004, "ofsErrorName": "INSUFFICIENT_AVAILABLE_LIQUIDITY" }
  }
}
```

A client MUST treat `data.ofsErrorCode` as the authoritative error identity for programmatic handling and MAY treat `message`/`data.ofsErrorName` as a human-readable label. This API introduces no error codes of its own outside the standard JSON-RPC transport codes — every application-level failure is already named by OFS-8000.

## 11. Subscriptions

```text
GET /ws
```

A node MAY expose a WebSocket endpoint streaming every successful `sendX` mutation as it is applied, as a JSON object carrying the method name and its result:

```json
{ "method": "sendReservationRequest", "result": "res-1" }
```

This is a single, unfiltered firehose, not a per-topic subscription protocol — a client that only cares about a subset of activity (a specific reservation, a specific oracle pair) filters client-side. A future revision of this specification MAY define server-side topic filtering if operational experience shows the firehose doesn't scale for high-volume consumers; inventing that filtering syntax speculatively, before a real consumer needs it, is out of scope here.

## 12. Versioning

This specification's own version (§ header) tracks breaking changes to the *envelope*: the JSON-RPC transport, the `getX`/`sendX` naming convention, the error model, and the self-description mechanism. It does not version individual methods — a new method appearing, or an existing method gaining an optional parameter, is not a breaking change to this specification and does not require a version bump here.

Breaking changes to an individual domain's method (a required parameter added, a result shape changed) are the concern of that domain's own governing specification (§6's table), not this one.

## 13. Security Considerations

* Every `sendX` payload's signature is verified through the identical path a gossip-received event goes through (OFS-1200 §8) — there is no separate, weaker verification path for API-submitted events.
* Because the node never constructs or signs anything on a caller's behalf, a compromised node cannot forge a caller's mutation; it can only refuse to relay one it has already received (an availability concern, not an authenticity one).
* A node exposing this API publicly SHOULD apply standard web-facing protections (TLS termination, request size limits, connection limits) at its own deployment layer; this specification's transport (§4) does not preclude any of them.
* The redaction of §7.1 is a property of the *API surface*, not of the data. Every redacted record is gossip-replicated, so it protects the ease of a query rather than the contents of a record, and it MUST NOT be described to users as confidentiality. Its value is real and bounded: it is the difference between querying somebody else's access node and running one.
* An unbounded read is a denial-of-service surface as well as a privacy one. A read that enumerates a growing collection MUST be bounded — see OFS-2100 §20.1 for the paging rule and why its page ceiling is not advisory.
* A wallet-proof read's nonce ledger is itself attackable if it is keyed by the subject rather than by the nonce (§7.2). An implementation adding a new gated surface MUST reuse the same issuer and the same ledger, with its own domain separator, rather than implementing the handshake again.

## 14. Performance Considerations

* Read methods (`getX`) SHOULD be servable from a node's local replicated state without a network round-trip, matching the design goal that gossip-replicated state is already locally authoritative.
* A node SHOULD bound the number of concurrent WebSocket subscribers and the backlog buffered per subscriber; this specification does not mandate a specific limit (see §8's rate-limiting note for the same reasoning).

## 15. Conformance

A compliant implementation MUST:

* Expose exactly one JSON-RPC 2.0 endpoint for both reads and writes.
* Name every read method with a `get` prefix and every write method with a `send` prefix.
* Accept only an opaque, pre-signed payload for every `sendX` method — never construct or sign a payload on the caller's behalf.
* Verify a `sendX` payload's signature through the same path a gossip-received event uses.
* Redact party identity, payment references, a dispute's reason, and the arbitrator-to-vote pairing from every public trade read, including any read that joins two of them (§7.1).
* Refuse, rather than narrow, a wallet-proof read whose proof does not verify, and separate every gated surface by its own domain (§7.2).
* Report every application-level failure using OFS-8000's numeric error codes, carried in a `-32000` JSON-RPC error's `data` field.
* Publish a self-description of its own current method catalog, generated from its own dispatch table.
* Never introduce a parallel authentication system distinct from wallet signatures and OFS-1400 sessions.

## 16. Relationship to Other Specifications

```text id="ona-architecture"
        OFS-2000 .. OFS-7100
   (every marketplace / protocol spec)
                  │
                  ▼
             OFS-8000
        Error Code Registry
                  │
                  ▼
             OFS-8200
          Node API (ONA)
                  │
      ┌───────────┼────────────┐
      ▼           ▼            ▼
   SDKs      Trading Bots   Explorers /
                             Dashboards
```

ONA is the single point where every other specification in this suite becomes reachable by something outside the peer-to-peer network. It defines no new state and no new behavior — only how already-specified state and behavior are read and mutated by a caller who is not itself a full gossiping peer.
