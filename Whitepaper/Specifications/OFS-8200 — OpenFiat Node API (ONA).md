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

* **`getX`** — a read. MUST NOT mutate node state. MUST be safe to retry and safe to call without any signature.
* **`sendX`** — a write. MUST take exactly one param, `data`: an opaque, base64-encoded, already-signed JSON payload the caller's own client produced locally — the domain's own `Signed*` envelope, JSON-serialized. The node's only job is to decode it and apply it through the identical validate → store → broadcast path OFS-1200 §8 defines for a gossip-received event — it never constructs, completes, or signs a payload for the caller. JSON is deliberately chosen here over the postcard-based wire format OFS-1200's gossip envelope carries internally: `sendX` is the one boundary a non-Rust SDK (TypeScript, Python, ...) has to cross, and a signature computed over a compact binary encoding only Rust's `serde` ecosystem produces byte-identically would make every other language re-implement that encoding just to sign anything. Every domain event's signature is computed over its own JSON encoding for exactly this reason — see the domain's own specification for the exact `Signed*` shape.

```json
{ "method": "getReservation", "params": { "id": "res-1" } }
{ "method": "sendReservationRequest", "params": { "data": "<base64 JSON bytes>" } }
```

No third method-name family is defined. A future specification that needs one (for example, a batched submission method) MUST NOT repurpose `getX`/`sendX` semantics for a different meaning.

## 6. Method Categories

Each domain specification already listed in "Depends On" contributes its own methods to one shared namespace. The canonical category list, by underlying specification:

| Domain | Governing spec | Representative methods |
| --- | --- | --- |
| Advertisements | OFS-2100 | `getAdvertisement`, `getAdvertisements`, `sendAdvertisementCreate` |
| Reservations | OFS-2200 | `getReservation`, `getReservations`, `sendReservationRequest` |
| Settlement | OFS-2300 | `getSettlement`, `sendSettlementInitiate`, `sendPaymentSubmitted`, `sendSettlementApproved` |
| Trade (read-only join) | OFS-2000 | `getTrade`, `getTrades` |
| Disputes | OFS-2400 | `getDispute`, `sendDisputeOpen`, `sendArbitratorJoin`, `sendVoteCommit`, `sendVoteReveal` |
| Identity | OFS-5000 | `getIdentityClaim`, `getIdentityClaimsByWallet`, `sendClaimPublish` |
| Reputation (read-only) | OFS-3000 | `getReputation` |
| Governance | OFS-4000 | `getProposal`, `getProposals`, `sendProposalCreate`, `sendVoteCast` |
| Service providers | OFS-1500 | `getProvider`, `getProviders`, `sendProviderRegister` |
| Notifications | OFS-6000 | `getSubscription`, `sendSubscriptionUpdate`, `sendDeliveryReport` |
| Oracles | OFS-7000 | `getOracleRecord`, `getMedianExchangeRate`, `sendOraclePublish` |
| Risk intelligence | OFS-7100 | `getWalletScreening`, `sendRiskPublish` |
| Snapshots | OFS-1300 | `getLatestSnapshot`, `getCheckpointHeight`, `sendSnapshotAnnounce` |
| Sessions | OFS-1400 | `getSession`, `sendSessionEstablish`, `sendSessionRenew`, `sendSessionRevoke`, `sendSessionMigrate` |
| Node | — | `getVersion`, `getHealth` |

This table is illustrative, not normative on its own — the normative list for a given node is whatever its self-description (§9) reports, since that is generated from the node's own dispatch table.

A future domain specification (a new OFS-2xxx/6xxx/7xxx document) that wants an API surface MUST follow §5's naming convention and add itself to this table's spirit; this specification does not need to be revised for every new domain, the same way OFS-8100 (Event Type Registry) doesn't need revision for every new event name.

## 7. Authentication

There is no API-key, OAuth, or session-token authentication layer distinct from the rest of the protocol. Three authentication tiers apply, matching how much a method actually needs:

1. **Unauthenticated reads.** Most `getX` methods require no proof of identity — they return already-public gossip-replicated state, identical to what any peer already has.
2. **Signed writes.** Every `sendX` method's payload is itself a signed event (§5) — authentication is embedded in the payload the caller already produced, not a separate header or token.
3. **Session-scoped reads.** A small number of methods (account-scoped data, e.g. a wallet's own notification subscriptions) require an established session per OFS-1400; the caller presents the session identifier issued by `sendSessionEstablish`.

Node operators MAY additionally require transport-level authentication (mTLS, an API gateway, an allowlist) for their own deployment — that is a deployment concern, not part of this specification's contract.

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

## 14. Performance Considerations

* Read methods (`getX`) SHOULD be servable from a node's local replicated state without a network round-trip, matching the design goal that gossip-replicated state is already locally authoritative.
* A node SHOULD bound the number of concurrent WebSocket subscribers and the backlog buffered per subscriber; this specification does not mandate a specific limit (see §8's rate-limiting note for the same reasoning).

## 15. Conformance

A compliant implementation MUST:

* Expose exactly one JSON-RPC 2.0 endpoint for both reads and writes.
* Name every read method with a `get` prefix and every write method with a `send` prefix.
* Accept only an opaque, pre-signed payload for every `sendX` method — never construct or sign a payload on the caller's behalf.
* Verify a `sendX` payload's signature through the same path a gossip-received event uses.
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
