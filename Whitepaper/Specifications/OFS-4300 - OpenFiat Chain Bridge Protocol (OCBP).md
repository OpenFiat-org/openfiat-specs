# OFS-4300 — OpenFiat Chain Bridge Protocol (OCBP)

**Document ID:** OFS-4300

**Title:** OpenFiat Chain Bridge Protocol

**Version:** 0.1.0 (Draft)

**Status:** Draft

**Category:** Governance Layer

**Depends On:** OFS-1200, OFS-1800, OFS-2300, OFS-8100, OFS-8200

---

## Abstract

The OpenFiat Chain Bridge Protocol (OCBP) defines how OpenFiat P2P nodes reach the Solana execution layer that OFS-4200's on-chain programs run on. A node that maintains its own Solana RPC connection can supply a fresh blockhash, submit transactions, and query account state directly. A node with no RPC connection still participates: it obtains the current blockhash from peers over the existing gossip network (OFS-1200) and gets its already-signed transactions relayed to the chain by whichever RPC-connected peers pick them up.

OCBP does not introduce custody. A node never holds or signs with a user's trading or staking keys — it supplies blockhashes and relays already-signed transactions, nothing more. Signing happens client-side, in the caller's own wallet.

---

## 1. Introduction

OFS-4200 defines the on-chain programs (presale, escrow, staking, governance) that hold and move funds. Those programs are Solana programs — reaching them requires a Solana blockhash (to construct a valid transaction) and a path to submit a signed transaction to the cluster. Neither exists anywhere in the OpenFiat P2P network today: `crates/settlement`'s own escrow-release path is explicitly deferred pending exactly this bridge.

Not every node operator wants to run Solana RPC infrastructure alongside their OpenFiat node. This specification lets both kinds of node participate fully: one supplies chain connectivity to the network, the other borrows it — without either needing custody of anyone's keys.

---

## 2. Scope

This specification defines:

* Two node connectivity modes with respect to the Solana chain.
* How a fresh blockhash reaches a node with no direct RPC connection.
* How an already-signed transaction reaches the Solana cluster from a node with no direct RPC connection.
* Deduplication of blockhash announcements across many independently RPC-connected nodes.
* The RPC methods a client (an SDK, a wallet, a trading bot) uses regardless of which mode the node it's talking to is in.

This specification does not define:

* Transaction construction or instruction encoding (OFS-4200 and the Anchor programs it describes own that).
* Custody of any kind — no node signs a user transaction on the user's behalf.
* Consensus, finality, or anything Solana's own runtime already guarantees.

---

## 3. Design Goals

The protocol SHALL:

* Let a node with no Solana RPC connection still submit and observe transactions.
* Never require a node to hold or sign with a user's trading or staking key.
* Bound gossip amplification when many independent nodes observe and announce the same blockhash.
* Let a client submit a transaction identically regardless of which mode the node it's talking to is in.
* Reuse OFS-1200's existing gossip transport and OFS-1800's existing rate-limiting overlay rather than inventing new ones.

---

## 4. Node Connectivity Modes

Every node operates in exactly one of two modes, configured by its operator:

**RPC-Connected.** The node maintains a live connection (HTTP and, where available, WebSocket) to one or more Solana RPC endpoints. It can fetch the latest blockhash, submit transactions, and query arbitrary account state directly.

**Gossip-Only.** The node has no direct Solana RPC connection. It obtains the current blockhash from gossip (§6) and submits transactions by requesting relay from an RPC-Connected peer (§7). It cannot answer a query that requires a live account read — that is a real, documented capability boundary, not a gap to work around.

A node MAY change mode at any time by reconfiguring; the protocol places no restriction on how many RPC endpoints an RPC-Connected node uses.

---

## 5. Non-Custodial Boundary

A node — in either mode — MUST NOT hold, generate, or sign with a Solana keypair on behalf of a user's trading, staking, or governance activity. The transaction a node relays arrives already signed by its owner's own wallet; the node's only actions on it are (a) supplying the blockhash used to construct it and (b) forwarding its already-signed bytes toward the Solana cluster.

A node's own node-identity keypair (OFS-1000 §6) is unrelated to this boundary — it signs gossip envelopes and RPC session material, never a Solana transaction on a user's behalf.

---

## 6. Blockhash Announcement

An RPC-Connected node periodically observes the current blockhash from its own RPC connection and originates a `BlockhashAnnounced` event (OFS-8100) onto gossip's dedicated Chain channel, carrying the blockhash, its slot, and the observation time.

Origination is open to any RPC-Connected node — this is not a role-scoped event the way `AdvertisementCreated` is scoped to Merchant Gateways, since chain connectivity is an operator choice, not a registered role.

**Announcement rate.** A node MUST NOT originate a new announcement for every slot it observes. It SHOULD announce only after both a minimum time interval and a minimum slot-advance have elapsed since its last announcement. `[PROPOSED — NEEDS SIGN-OFF]` Concrete defaults are an implementation parameter, following the same pattern as OGP's TTL default and this workspace's other `[PROPOSED]` protocol numbers — Solana produces a new slot roughly every 400ms, so a per-slot announcement policy would flood the Chain channel.

**Amplification control.** Because the underlying fact (a given blockhash at a given slot) is the same across every RPC-Connected node that observes it, gossip's ordinary duplicate suppression — keyed by event id — does not collapse these announcements; each origin produces a distinct, separately signed event for the same content. A node MUST therefore additionally deduplicate `BlockhashAnnounced` events by their content, `(blockhash, slot)`, independent of event id: it stores and rebroadcasts only the first such event it sees for a given `(blockhash, slot)` pair; later announcements of an already-seen pair are accepted as valid but MUST NOT be rebroadcast. This is what bounds fan-out to a constant cost regardless of how many independent nodes are announcing the same real-world blockhash — including at the scale of thousands of RPC-Connected nodes across the network.

**Recency selection.** Separately from the amplification control above, a node MUST track the highest-slot blockhash it has seen and use that — not simply the first one it ever saw — as its current view for constructing or forwarding transactions, evicting a blockhash once it exceeds Solana's own validity window (approximately 150 slots, on the order of 60-90 seconds). Content-addressed first-seen suppression bounds gossip traffic; it is not, by itself, a safe way to pick which blockhash to build a transaction against, since the first one a node saw may since have expired. Both mechanisms operate together: first-seen-per-content controls what gets rebroadcast, highest-slot-not-yet-expired controls what gets used.

---

## 7. Transaction Relay

A Gossip-Only node that needs to submit an already-signed Solana transaction originates a `TransactionRelayRequested` event (OFS-8100) on the Chain channel, carrying the transaction's signed wire bytes and the request time. An RPC-Connected node relays by submitting those bytes, unmodified, to its own RPC connection(s).

Origination of `TransactionRelayRequested` is open to any node, not role-scoped — any peer may need relay regardless of registered role. A node relaying on another's behalf MUST reject a payload that fails to deserialize as a well-formed Solana transaction before submitting it, but MUST NOT otherwise inspect, alter, or condition submission on the transaction's contents. Rate limiting on origination follows OFS-1800's existing per-peer overlay, since this event type is open-origination rather than role-scoped.

**Multiple relay is expected, not an error.** Any number of RPC-Connected peers may independently receive and submit the same relay request to any number of RPC endpoints. Solana's own signature-based deduplication handles the resulting duplicate submissions; the protocol does not attempt to coordinate which peer "should" relay a given request.

A relaying node MAY, after observing confirmation from its own RPC connection, originate a `TransactionRelayed` event (OFS-8100) carrying the signature, the slot it landed in, and its own identity, so the requesting node can learn of success without needing an RPC connection of its own to poll. This is best-effort — a Gossip-Only node's own transaction succeeding does not depend on receiving this event.

---

## 8. Client-Facing RPC Surface

OFS-8200 registers three methods under this specification, usable identically against a node in either mode:

* `getChainStatus` — the node's mode, its current blockhash and slot, and that blockhash's remaining validity.
* `getLatestBlockhash` — the node's current blockhash (from its own RPC connection if RPC-Connected, from gossip's recency-selected value per §6 if Gossip-Only).
* `sendTransaction` — submit an already-signed Solana transaction. An RPC-Connected node submits it directly; a Gossip-Only node originates a `TransactionRelayRequested` event (§7). The caller does not need to know or care which happened.

`sendTransaction` is the one documented exception to OFS-8200 §5's general `sendX` contract: its `data` is a signed Solana transaction, not an OpenFiat `Signed*` JSON envelope, because Solana's own transaction-signing scheme — not this protocol suite's JSON convention — governs that payload.

---

## 9. What This Protocol Does NOT Do

OCBP SHALL NEVER:

* Hold, generate, or sign with a user's Solana trading, staking, or governance key.
* Guarantee that a relayed transaction lands on-chain — Solana's own runtime determines that.
* Treat blockhash announcement as a consensus or voting mechanism — it is a convergent observation of a shared external fact, not a decision the network makes.
* Require every node to run Solana RPC infrastructure.

---

## 10. Security Considerations

Implementations MUST:

* Reject a `TransactionRelayRequested` payload that fails to deserialize as a well-formed Solana transaction before submitting it, without otherwise inspecting or conditioning on its contents.
* Apply OFS-1800's rate-limiting overlay to both `BlockhashAnnounced` and `TransactionRelayRequested` origination, since both are open to any node rather than role-scoped.
* Never derive trust or priority from a peer's claimed chain mode — a node claiming to be RPC-Connected is not thereby granted authority beyond what OFS-1600's existing reputation system already governs.

---

## 11. Performance Considerations

Blockhash amplification control (§6) is the primary cost concern: without content-addressed deduplication, a network of N RPC-Connected nodes each observing and announcing the same blockhash produces O(N) redundant gossip traffic per blockhash change. With it, cost is bounded regardless of N. Implementations SHOULD apply the same content-addressed approach to any future high-frequency, independently-observed-fact event type this protocol suite introduces.

---

## 12. Conformance

A compliant implementation MUST:

* Support both node connectivity modes.
* Never sign a Solana transaction on a user's behalf.
* Deduplicate blockhash announcements by content, not event id.
* Track and use the highest-slot, not-yet-expired blockhash for its own current view.
* Expose `getChainStatus`, `getLatestBlockhash`, and `sendTransaction` with identical caller-facing behavior regardless of its own mode.
* Reject malformed relay-requested transactions before submission.

---

## 13. Relationship to Other Specifications

```text id="chain-bridge-architecture"
          OFS-1200                    OFS-4200
       Gossip Protocol           On-Chain Programs
             │                    (Solana, off-network)
             ▼                          ▲
        OFS-4300                        │
     Chain Bridge Protocol ─────────────┘
             │
   ┌─────────┴─────────┐
   ▼                    ▼
RPC-Connected        Gossip-Only
   node                 node
   │                    │
   │ direct RPC   gossip-sourced blockhash +
   │              relay via RPC-Connected peers
   ▼                    ▼
        OFS-2300 Settlement
     (first real consumer: escrow release)
```

OCBP is the missing link between the P2P network OFS-1000 through OFS-1800 describe and the on-chain programs OFS-4200 describes. `crates/settlement`'s escrow release, and any future staking or governance instruction submitted from a node, reach Solana through this protocol rather than through a direct, per-consumer RPC integration.
