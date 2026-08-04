# OFS-8000 — OpenFiat Error Code Registry (OECR)

**Document ID:** OFS-8000

**Title:** OpenFiat Error Code Registry

**Version:** 1.0.0 (Draft)

**Status:** Draft Standard

**Category:** Core Protocol

**Depends On:** All OpenFiat Protocol Specifications

---

## Abstract

The OpenFiat Error Code Registry (OECR) defines the canonical set of protocol error codes used throughout the OpenFiat ecosystem.

Every OpenFiat implementation SHALL return standardized error codes for identical failure conditions, regardless of implementation language, operating system, or API transport.

This specification guarantees consistent behavior across:

* OpenFiat Nodes
* SDKs
* Wallets
* Merchant Applications
* JSON-RPC
* REST APIs
* WebSocket APIs
* CLI Tools
* Governance Services
* Oracle Providers
* Risk Providers
* Notification Providers

---

## 1. Introduction

OpenFiat consists of many independent implementations.

Without standardized error codes:

* Applications cannot reliably recover from failures.
* SDK behavior becomes inconsistent.
* Monitoring becomes fragmented.
* Client applications require implementation-specific logic.

This specification defines one canonical error registry for the entire protocol.

---

## 2. Design Goals

The error system SHALL be:

* Deterministic
* Language independent
* Transport independent
* Human readable
* Machine readable
* Extensible
* Backward compatible

---

## 3. Error Object

Every protocol API SHALL expose the following logical structure.

```json
{
  "error": {
    "code": 4004,
    "name": "INSUFFICIENT_AVAILABLE_LIQUIDITY",
    "message": "Advertisement cannot satisfy requested amount.",
    "retryable": true,
    "details": {}
  }
}
```

`retryable` is `true` in this example because 4004 is: a merchant's available liquidity returns on its own when a competing reservation expires or is cancelled, without the caller changing anything. §16 gives the rule the rest of the registry is classified by.

---

## 4. Error Fields

| Field     | Description                               |
| --------- | ----------------------------------------- |
| code      | Numeric protocol error                    |
| name      | Stable symbolic identifier                |
| message   | Human-readable description                |
| retryable | Indicates whether retrying may succeed    |
| details   | Optional implementation-specific metadata |

Implementations MAY add additional fields but SHALL NOT alter the meaning of the standard fields.

---

## 5. Error Code Ranges

| Range     | Category                   |
| --------- | -------------------------- |
| 0000–0999 | General Protocol           |
| 1000–1999 | Network                    |
| 2000–2999 | Identity                   |
| 3000–3999 | Advertisements             |
| 4000–4999 | Reservations & Marketplace |
| 5000–5999 | Settlement & Liquidity     |
| 6000–6999 | Disputes                   |
| 7000–7999 | Governance                 |
| 8000–8999 | Notifications              |
| 9000–9999 | Internal & Implementation  |

Future specifications SHALL allocate codes only within their assigned range.

---

## 6. General Errors (0000)

| Code | Name                    |
| ---- | ----------------------- |
| 0000 | UNKNOWN_ERROR           |
| 0001 | INTERNAL_ERROR          |
| 0002 | INVALID_REQUEST         |
| 0003 | INVALID_PARAMETER       |
| 0004 | UNSUPPORTED_OPERATION   |
| 0005 | NOT_IMPLEMENTED         |
| 0006 | RESOURCE_NOT_FOUND      |
| 0007 | RESOURCE_ALREADY_EXISTS |
| 0008 | OPERATION_TIMEOUT       |
| 0009 | RATE_LIMIT_EXCEEDED     |
| 0010 | DECRYPTION_FAILED       |

`DECRYPTION_FAILED` (0010) is a sealed payload that did not open, and it MUST NOT be reported as `INVALID_SIGNATURE` (1003). The two are routinely reached in sequence — a record's signature is verified, and its encrypted payload is opened afterwards with a key the reader may not hold — so the second failing says nothing whatever about the first. A peer told 1003 signs again and re-sends the identical unreadable bytes, while the remedy for 0010 is to obtain the key. 0010 is deliberately silent on *why* the box did not open: an implementation SHALL NOT distinguish a wrong key from a tampered ciphertext or a payload lifted from another slot, because a caller able to tell them apart holds a decryption oracle. It lives in the general range because no domain owns it; any protocol carrying a sealed payload reports it.

---

## 7. Network Errors (1000)

| Code | Name                         |
| ---- | ---------------------------- |
| 1000 | NETWORK_ERROR                |
| 1001 | PEER_NOT_FOUND               |
| 1002 | PROTOCOL_VERSION_MISMATCH    |
| 1003 | INVALID_SIGNATURE            |
| 1004 | REPLAY_ATTACK_DETECTED       |
| 1005 | SNAPSHOT_VERIFICATION_FAILED |
| 1006 | SESSION_EXPIRED              |
| 1007 | MESSAGE_OUT_OF_ORDER         |
| 1008 | NODE_NOT_SYNCHRONIZED        |
| 1009 | NETWORK_UNAVAILABLE          |
| 1010 | CHAIN_UNAVAILABLE            |
| 1011 | BLOCKHASH_EXPIRED            |
| 1012 | MALFORMED_TRANSACTION        |
| 1013 | TRANSACTION_SUBMISSION_FAILED |
| 1014 | SESSION_REVOKED              |
| 1015 | REQUEST_EXPIRED              |

`SESSION_EXPIRED` (1006) means one thing: a session reached the end of its validity window. It SHALL NOT be used for either of the conditions below, both of which were reaching clients as 1006.

`SESSION_REVOKED` (1014) is a session deliberately ended before its window closed. Revocation is permanent (OFS-1400 §16) and expiry is not, so the two demand opposite responses: a client answering 1006 renews, and renewing a revoked session is the one request that will never be granted.

`REQUEST_EXPIRED` (1015) is a signed artifact whose own validity window has passed — a signed request outside its anti-replay freshness window, a fee-settlement quote past its validity bound (OFS-1500). No session is involved, and re-authenticating leaves the expired value exactly where it is, inside the bytes that were signed. The remedy is to rebuild the artifact from current values — a fresh timestamp, a fresh quote at the current rate — and sign it again. 1015 is therefore non-retryable in §16's sense: the identical request carries the identical expired window.

---

## 8. Identity Errors (2000)

| Code | Name                      |
| ---- | ------------------------- |
| 2000 | IDENTITY_NOT_FOUND        |
| 2001 | INVALID_IDENTITY_CLAIM    |
| 2002 | IDENTITY_ALREADY_EXISTS   |
| 2003 | IDENTITY_REVOKED          |
| 2004 | CLAIM_VERIFICATION_FAILED |
| 2005 | INVALID_SIGNATURE_CHAIN   |
| 2006 | IDENTITY_IN_USE_ELSEWHERE |

`IDENTITY_IN_USE_ELSEWHERE` (2006) is a node observing an event signed by its own key that it did not emit — proof that a second process holds the same identity. It MUST NOT be reported as `INVALID_SIGNATURE` (1003): the signature verified, and that is the entire finding. Everyone who acts on 1003 acts wrongly here — a peer re-signs, an operator audits their signing path — while the actual remedy is to stop the duplicate and rotate the key. Non-retryable, and permanently so until a human intervenes.

---

## 9. Advertisement Errors (3000)

| Code | Name                       |
| ---- | -------------------------- |
| 3000 | ADVERTISEMENT_NOT_FOUND    |
| 3001 | ADVERTISEMENT_DISABLED     |
| 3002 | ADVERTISEMENT_EXPIRED      |
| 3003 | INVALID_ADVERTISEMENT      |
| 3004 | DUPLICATE_ADVERTISEMENT    |
| 3005 | UNSUPPORTED_PAYMENT_METHOD |
| 3006 | PAYMENT_METHOD_LIMIT_REACHED |

---

## 10. Reservation Errors (4000)

| Code | Name                             |
| ---- | -------------------------------- |
| 4000 | RESERVATION_NOT_FOUND            |
| 4001 | RESERVATION_ALREADY_EXISTS       |
| 4002 | RESERVATION_EXPIRED              |
| 4003 | RESERVATION_CANCELLED            |
| 4004 | INSUFFICIENT_AVAILABLE_LIQUIDITY |
| 4005 | MERCHANT_OFFLINE                 |
| 4006 | INVALID_RESERVATION_STATE        |
| 4007 | PRICE_DISAGREEMENT               |

---

## 11. Settlement Errors (5000)

| Code | Name                            |
| ---- | ------------------------------- |
| 5000 | SETTLEMENT_FAILED               |
| 5001 | VAULT_INSUFFICIENT_BALANCE      |
| 5002 | INVALID_DEPOSIT                 |
| 5003 | UNSUPPORTED_STABLECOIN          |
| 5004 | BLOCKCHAIN_CONFIRMATION_TIMEOUT |
| 5005 | SETTLEMENT_ALREADY_COMPLETED    |
| 5006 | SETTLEMENT_ALREADY_CANCELLED    |
| 5007 | FLAGGED_DEPOSIT_ADDRESS         |
| 5008 | SETTLEMENT_NOT_FOUND            |
| 5009 | INVALID_SETTLEMENT_STATE        |
| 5010 | SETTLEMENT_ALREADY_EXISTS       |

`SETTLEMENT_ALREADY_EXISTS` (5010) and `SETTLEMENT_ALREADY_COMPLETED` (5005) are different statements and MUST NOT be substituted for one another. 5010 says only that the settlement id in the request is already held by this node; it makes no claim about that settlement's state or its parties. 5005 says a settlement finished. An initiate re-sent after a dropped connection is the ordinary way to reach 5010, and answering it with 5005 tells a client its trade is complete when the settlement holding that id may still be at `AwaitingPayment` — a client that believes it stops waiting for a payment it is still owed.

Likewise `SETTLEMENT_NOT_FOUND` (5008) and `INVALID_SETTLEMENT_STATE` (5009) are distinct from `SETTLEMENT_FAILED` (5000) and from each other: an unknown settlement and an illegal transition are both permanent, while 5000 is retryable, and a client cannot act on any of the three if they arrive as one.

---

## 12. Dispute Errors (6000)

| Code | Name                 |
| ---- | -------------------- |
| 6000 | DISPUTE_NOT_FOUND    |
| 6001 | DISPUTE_ALREADY_OPEN |
| 6002 | DISPUTE_CLOSED       |
| 6003 | INVALID_EVIDENCE     |
| 6004 | DISPUTE_TIMEOUT      |
| 6005 | INVALID_DISPUTE_STATE |

`INVALID_DISPUTE_STATE` (6005) is this range's counterpart to `INVALID_RESERVATION_STATE` (4006) and `INVALID_SETTLEMENT_STATE` (5009): an action that is not a legal transition from the dispute's current state. It was absent, and in its absence `DISPUTE_CLOSED` (6002) absorbed the role — an arbitrator joining a panel that has just filled, a vote committed before the case locks, a reveal before the commit phase ends. None of those is a closed case. 6002 states an outcome that has not happened, and a participant who believes it stops acting on a dispute they are still entitled, and sometimes obliged, to act on.

6002 retains its meaning for a case that genuinely concluded. An implementation SHALL NOT report it for a dispute that is still live in any state.

---

## 13. Governance Errors (7000)

| Code | Name                      |
| ---- | ------------------------- |
| 7000 | PROPOSAL_NOT_FOUND        |
| 7001 | VOTING_CLOSED             |
| 7002 | DUPLICATE_VOTE            |
| 7003 | INSUFFICIENT_VOTING_POWER |
| 7004 | INVALID_PROPOSAL          |
| 7005 | PROPOSAL_ALREADY_EXISTS   |
| 7006 | INVALID_PROPOSAL_STATE    |

`INVALID_PROPOSAL` (7004) is a verdict on a proposal's content, and only on its content. Three conditions were reaching clients as 7004 that say nothing about the content, each sending an author to edit text that was never wrong.

`PROPOSAL_ALREADY_EXISTS` (7005) says only that the proposal id in the request is already held. It makes no claim about either proposal. As with `SETTLEMENT_ALREADY_EXISTS` (5010), the ordinary way to reach it is a resend after a dropped connection, and it takes its own code rather than the generic `RESOURCE_ALREADY_EXISTS` (0007) because governance has an allocated range and every domain with one names its own.

`INVALID_PROPOSAL_STATE` (7006) is this range's counterpart to 4006, 5009 and 6005: withdrawing a proposal that already executed, activating one already active. It is fixed by reading the proposal's status, never by editing the proposal.

An unauthorized action — one attempted by someone other than the proposal's author — is `INVALID_IDENTITY_CLAIM` (2001), the same answer every other range gives to a caller who is not who they would have to be. It is not a governance code, and it is not 7004.

---

## 14. Notification Errors (8000)

| Code | Name                              |
| ---- | --------------------------------- |
| 8000 | NOTIFICATION_PROVIDER_UNAVAILABLE |
| 8001 | DELIVERY_FAILED                   |
| 8002 | INVALID_DESTINATION               |
| 8003 | UNSUPPORTED_NOTIFICATION_TYPE     |
| 8004 | SUBSCRIPTION_NOT_FOUND            |

---

## 15. Internal Errors (9000)

These errors SHALL NOT expose sensitive implementation details.

| Code | Name                         |
| ---- | ---------------------------- |
| 9000 | DATABASE_ERROR               |
| 9001 | STORAGE_CORRUPTED            |
| 9002 | CONFIGURATION_ERROR          |
| 9003 | SERIALIZATION_ERROR          |
| 9004 | DESERIALIZATION_ERROR        |
| 9005 | UNKNOWN_IMPLEMENTATION_ERROR |

---

## 16. Retry Semantics

Retryability is a property of the code, not of the call site: for a given code the answer is the same whichever method returned it, and an implementation SHALL answer the same way every time it returns that code.

An implementation SHALL report it. §4 lists `retryable` as a standard field of the error object, and §17 names the field each transport carries it in. Withholding it does not leave a client without an opinion — it leaves the client to hardcode its own copy of this table, in every language it is written in, and to drift from this document the first time a code is added or reclassified. That duplication is what the field exists to prevent, and an implementation that computes the judgement internally without emitting it has kept the cost and given away none of the benefit.

The field is a statement about this code, not a promise about this request. A client MAY apply its own policy on top — a retry budget, a backoff schedule, or a decision not to retry a particular operation at all — and `retryable: false` means only that the identical request will reach the identical outcome, so a client that repeats it unchanged learns nothing.

Errors fall into two categories:

### Retryable

Examples:

* NETWORK_UNAVAILABLE
* OPERATION_TIMEOUT
* DELIVERY_FAILED
* BLOCKCHAIN_CONFIRMATION_TIMEOUT

Clients MAY retry automatically using an appropriate backoff strategy.

### Non-Retryable

Examples:

* INVALID_REQUEST
* INVALID_SIGNATURE
* ADVERTISEMENT_EXPIRED
* FLAGGED_DEPOSIT_ADDRESS

Clients SHOULD NOT retry without changing the request.

---

## 17. Transport Mapping

Implementations SHALL preserve the protocol error code regardless of transport.

Examples:

* JSON-RPC: returned in the `error` object. The reference node reports every domain failure as JSON-RPC code `-32000` carrying `{"ofsErrorCode": <number>, "ofsErrorName": <name>, "ofsRetryable": <boolean>}` in `error.data` — see OFS-8200 §10.
* REST: returned in the response body with an appropriate HTTP status.
* WebSocket: returned in the protocol message payload.
* CLI: displayed to the user while preserving the numeric code for scripting.

A transport SHALL carry the numeric code, the symbolic name and the retryability of every error it reports. A transport MAY name the fields to suit its own conventions; it SHALL NOT drop one of the three, and a client SHALL treat the numeric code as the authoritative identity where the two disagree.

---

## 18. Extensibility

Future protocol specifications MAY reserve additional codes within their allocated ranges.

Existing codes SHALL NOT be renumbered or reused.

Deprecated codes SHOULD remain reserved permanently to preserve compatibility.

---

## 19. Conformance

A compliant implementation MUST:

* Return standardized protocol error codes.
* Preserve numeric values across transports.
* Preserve symbolic names.
* Report each error's retryability alongside it (§16), rather than deciding it privately.
* Avoid exposing sensitive internal implementation details.
* Maintain backward compatibility with previously assigned codes.

---

## 20. Summary

The OpenFiat Error Code Registry provides a single, protocol-wide error vocabulary for every implementation.

By standardizing numeric error codes, symbolic identifiers, retry semantics, and transport mappings, it ensures that all OpenFiat clients and services behave consistently regardless of language or implementation.

The OpenFiat Error Code Registry answers one fundamental question:

**"How can every OpenFiat implementation report failures in a deterministic, interoperable, and machine-readable manner?"**

The next logical protocol after this would be **OFS-8100 — OpenFiat Event Type Registry**, which standardizes every protocol event (e.g., `AdvertisementCreated`, `ReservationOpened`, `SettlementCompleted`, `DisputeOpened`) exchanged across the network. That complements the error registry and gives every implementation a shared event vocabulary.
