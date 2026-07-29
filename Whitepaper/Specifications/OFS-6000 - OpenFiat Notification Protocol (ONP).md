# OFS-6000 — OpenFiat Notification Protocol (ONP)

**Document ID:** OFS-6000

**Title:** OpenFiat Notification Protocol

**Version:** 1.1.0 (Draft)

**Status:** Draft

**Category:** Infrastructure

**Depends On:** OFS-1000, OFS-1200, OFS-1400, OFS-1500

---

## Abstract

The OpenFiat Notification Protocol (ONP) defines how applications, merchants, users, infrastructure providers, and third-party notification services deliver real-time protocol events across the OpenFiat ecosystem.

Unlike centralized platforms that own and operate notification infrastructure, OpenFiat allows anyone to become a Notification Provider by implementing a standard protocol and registering through the decentralized Service Registry.

This architecture creates a permissionless, fault-tolerant notification network that can deliver events through multiple communication channels without depending on a single company or service.

---

## 1. Introduction

Trading cannot rely on users constantly watching their screens.

Participants need immediate notification when:

* A reservation is received.
* Fiat payment is submitted.
* A dispute is opened.
* Escrow is released.
* Governance voting begins.
* A wallet receives important alerts.

The Notification Protocol standardizes how these events are delivered while remaining completely decentralized.

---

## 2. Scope

This specification defines:

* Notification providers
* Event subscriptions
* Notification routing
* Delivery channels
* Delivery confirmation
* Retry behavior
* Provider registration
* Provider redundancy
* Notification security

This specification does **not** define:

* User interface design
* Email formatting
* SMS formatting
* Push notification content
* Third-party provider APIs

---

## 3. Design Goals

The Notification Protocol SHALL:

* Be permissionless.
* Support multiple providers.
* Avoid centralized notification infrastructure.
* Support offline merchants.
* Guarantee eventual delivery where practical.
* Support future communication channels.
* Preserve user privacy.

---

## 4. Design Philosophy

Notifications are infrastructure.

They are **not** protocol consensus.

A delayed notification MUST NOT change marketplace state.

The protocol continues operating even if every notification provider becomes unavailable.

Notifications improve usability.

They never determine protocol correctness.

---

## 5. Notification Providers

Any organization or individual may operate a Notification Provider.

Examples include:

* SMS Gateway
* Email Gateway
* Telegram Bot
* Discord Bot
* Mobile Push Service
* Browser Push Service
* WhatsApp Gateway (where permitted)
* Signal Gateway
* Webhook Provider
* Enterprise Messaging Gateway

Providers register through OFS-1500.

---

## 6. Decentralized Provider Model

There is no official notification provider.

Applications may choose:

* One provider
* Multiple providers
* Their own self-hosted provider

Competition improves reliability and reduces centralization.

---

## 7. Provider Registration

Notification Providers publish:

* Provider ID
* Supported channels
* Geographic coverage
* Rate limits
* Pricing (if applicable)
* Service endpoints
* Protocol version
* Public signing key

Registrations are digitally signed.

---

## 8. Supported Delivery Channels

Initial protocol channels include:

* Email
* SMS
* Telegram
* Discord
* Web Push
* Mobile Push
* Webhooks

Future governance proposals may introduce additional delivery channels.

---

## 9. Community Notification Providers

OpenFiat intentionally allows community-operated notification services.

For example:

A local entrepreneur may deploy an SMS gateway connected to a domestic telecommunications provider.

After registering through the Service Registry, that gateway becomes available to any OpenFiat application.

This encourages regional innovation while improving delivery quality in local markets.

---

## 10. Event Types

Examples include:

Trading

* Reservation Created
* Reservation Expiring
* Payment Submitted
* Settlement Approved
* Escrow Released
* Trade Completed

Marketplace

* Advertisement Disabled
* Reputation Updated

Disputes

* Evidence Requested
* Resolution Issued

Governance

* Proposal Published
* Voting Started
* Proposal Activated

Infrastructure

* Snapshot Available
* Node Maintenance
* Provider Offline

---

## 11. Event Subscription

Users subscribe to the notifications they wish to receive.

Examples:

```text id="subscriptions"
Wallet

↓

Trade Notifications

✓

Governance

✓

Marketing

✗

Infrastructure

✗

Security Alerts

✓
```

Subscriptions belong to the wallet and synchronize across compatible applications.

### 11.1 Subscription Destinations

A subscription also carries the destinations the wallet wants notifications delivered to. Each destination binds three things together:

* The Notification Provider it is addressed to
* The delivery channel
* The destination address itself

Subscriptions synchronize by replicating to every node. A destination address is a plaintext contact detail — an email address, a phone number, a webhook URL — so replicating one directly would write every user's contact details into permanent state that the entire network can read.

Destination addresses MUST NOT be replicated in plaintext.

Each destination address MUST be sealed to the public key of the Notification Provider it is bound to, as registered in the Service Registry (OFS-1500). Only that provider can open it. Every other participant — including every node that relays, stores, and re-serves the subscription — holds opaque ciphertext.

The sealing construction MUST be authenticated. An implementation MUST reject a sealed destination whose ciphertext or ephemeral key material has been altered, rather than yielding unusable plaintext to a delivery attempt.

A subscription carrying no destinations is valid. It produces no deliveries.

---

## 12. Routing

Applications select an appropriate Notification Provider.

Selection MAY consider:

* Reputation
* Delivery latency
* Geographic proximity
* Supported channels
* Cost
* Availability

The protocol intentionally does not mandate routing algorithms.

### 12.1 Destination Eligibility

The freedom above applies to which providers an application *offers* a user. Once a subscription exists, routing selects among that subscription's own destinations — the wallet has already chosen which provider may hold each address by sealing to it, and routing cannot widen that choice.

A destination MUST NOT be delivered to unless all of the following hold:

* The bound service resolves in the Service Registry
* The service is a Notification service, and its registered channel matches the destination's channel
* The service is currently healthy

Channel matching is a correctness and privacy requirement, not an optimization. A destination sealed for an SMS provider is a phone number; routing it to an email provider both discloses it to a party the wallet never selected and produces a delivery that cannot succeed.

A destination failing any check MUST be skipped. Skipping SHOULD be observable to the node operator rather than silent, since a subscription that silently delivers nothing is indistinguishable from one that was never created.

---

## 13. Redundant Delivery

Critical notifications MAY be delivered through multiple providers.

Example:

```text id="redundant-delivery"
Settlement Approved

↓

Telegram

+

SMS

+

Email
```

Redundancy improves reliability during provider outages.

### 13.1 Unintended Redundancy and Notification Identity

The redundancy above is chosen: a wallet subscribes on several channels and accepts several deliveries.

Unintended redundancy is a separate problem created by the network itself. Protocol events are gossip-replicated, so **every** node observes every event. If each node dispatches independently, one logical notification becomes one delivery per node: a recipient receives the same alert as many times as the network has nodes, and load on providers scales with network size rather than with actual activity. At any meaningful node count this is indistinguishable from an attack on both the user and the provider.

Electing a single dispatching node would suppress it, but reintroduces exactly the single point of failure §6 exists to eliminate.

Notification identity MUST therefore be deterministic. A notification's identifier MUST be derived from the identity of the triggering event, the trigger type, and the recipient — and from nothing node-local, such as wall-clock time, arrival order, or the observing node's own identity. Every node independently computing the identifier for the same logical notification MUST arrive at the same value, with no coordination.

Notification Providers MUST deduplicate on this identifier, delivering at most once per identifier regardless of how many nodes present it.

Nodes therefore dispatch independently and do not coordinate. Redundant dispatch is expected and desirable: it is the mechanism by which delivery survives an individual node failing, being partitioned, or declining to dispatch.

---

## 14. Delivery Confirmation

Providers SHOULD return delivery status.

Examples:

* Accepted
* Delivered
* Failed
* Expired
* Rejected

Delivery confirmations help applications determine whether retries are necessary.

### 14.1 Witnessed Dispatch and Claimed Delivery

Delivery status has two sources of very different strength, and implementations MUST NOT conflate them.

A **dispatch record** is what a node witnessed first-hand: that it handed a notification to a provider's registered endpoint, and the endpoint accepted it. The node performed the act and observed the outcome.

A **delivery confirmation** is what the provider reports happened afterwards on the channel — delivered, read, bounced, expired. Only the provider can observe the last mile. No node can, because the recipient is reachable only off-protocol.

Both are retained, and a provider's confirmation MUST NOT overwrite a node's own dispatch record.

A confirmation is self-reported by the party whose reputation (§18) and compensation depend on its content, so it MUST be verified before entering replicated state:

* Signed by the provider it names
* Naming a provider registered in the Service Registry
* Corresponding to a dispatch the receiving node itself witnessed, matching on service, recipient, and trigger

A confirmation naming an identifier the node never dispatched MUST be rejected and MUST NOT create a record. Without this, a registered provider can report on traffic it was never routed, or manufacture evidence of work that was never requested of it.

This strictness has a real cost, accepted deliberately: a node that legitimately never routed a given notification — one that joined late, restored from a snapshot, or holds no replica of the relevant subscription — will reject an otherwise honest confirmation. That loss is recoverable, because the nodes that did dispatch it accept the confirmation and gossip it onward, and dispatch is deterministic (§13.1), so in steady state those are all nodes holding the subscription. An accepted but unverifiable confirmation is not recoverable: it writes an unfalsifiable claim into replicated state permanently.

---

## 15. Retry Policy

Providers SHOULD retry transient failures.

Examples include:

* Temporary network failure
* Provider unavailable
* Mail server timeout
* SMS gateway congestion

Retry strategies remain implementation-specific.

This section governs the provider's own retries on the delivery channel, after it has accepted a notification. It does not require a node to re-attempt a dispatch the provider refused.

Node-side retry is discouraged. Because every node dispatches the same notification independently (§13.1), a refused dispatch is already being retried concurrently by every other node holding the subscription — retrying locally multiplies load on a provider that is likely refusing precisely because it is under strain. A notification is also time-sensitive: re-delivering a stale trade alert minutes later can be worse for the recipient than not delivering it. A node SHOULD therefore record a failed dispatch and stop.

---

## 16. Offline Merchants

Notification Providers enable merchants to remain reachable even when their trading application is closed.

Example:

```text id="offline-merchant"
Buyer Creates Reservation

↓

Protocol Event

↓

Notification Provider

↓

SMS

↓

Merchant Opens App

↓

Session Restored

↓

Trade Continues
```

This significantly improves merchant availability without requiring permanent online sessions.

---

## 17. Notification Authentication

Every notification originates from a signed protocol event.

Providers MUST verify:

* Event signature
* Event integrity
* Authorization
* Subscription validity

Notification providers never create protocol events.

They only deliver them.

---

## 18. Provider Reputation

Notification Providers accumulate operational reputation.

Metrics MAY include:

* Delivery success
* Delivery latency
* Uptime
* Failure rate
* Regional reliability

These metrics may influence future provider selection.

---

## 19. Privacy

Notification Providers SHOULD receive only the minimum information required to perform delivery.

Examples:

SMS Provider

* Phone number
* Notification content

Email Provider

* Email address
* Notification content

Providers SHOULD NOT receive unrelated trade information whenever possible.

End-to-end encryption MAY be supported by compatible channels.

Sealing of destination addresses (§11.1) is not covered by that "MAY" and is not optional. The minimum-information principle above governs what a provider receives at delivery time, but a subscription is replicated network-wide before any delivery occurs. Left in plaintext, a destination would disclose every user's contact details to every node regardless of how little any provider is later sent — defeating the principle at the point of subscription rather than at the point of delivery.

---

## 20. Security Considerations

Implementations MUST protect against:

* Fake notifications
* Message tampering
* Replay attacks
* Unauthorized subscriptions
* Provider impersonation
* Spam delivery
* Duplicate delivery amplification (§13.1)
* Fabricated delivery confirmations (§14.1)
* Disclosure of destination addresses through replicated state (§11.1)

All notification events MUST remain cryptographically verifiable.

The last three are properties of a replicated, permissionless network rather than of any one channel, and are not addressed by securing the transport to a provider. Each is enabled by the same underlying fact — that every node holds every subscription and observes every event — and each therefore has to be closed in the protocol rather than left to provider implementations.

---

## 21. Performance Considerations

The Notification Protocol is expected to support millions of daily events.

Implementations SHOULD optimize:

* Event batching
* Queue management
* Parallel delivery
* Regional routing
* Delivery retries

Notification delivery SHOULD not block marketplace operations.

---

## 22. Conformance

A compliant implementation MUST:

* Support decentralized Notification Providers.
* Verify signed protocol events.
* Support wallet subscriptions.
* Support multiple delivery channels.
* Support provider registration.
* Support delivery confirmations.
* Preserve notification privacy.
* Prevent notification forgery.
* Seal destination addresses to their bound provider, and never replicate one in plaintext (§11.1).
* Deliver only to destinations whose bound service resolves, matches the destination's channel, and is healthy (§12.1).
* Derive notification identifiers deterministically, from the triggering event and recipient alone, so that independent nodes agree without coordination (§13.1).
* Deduplicate on the notification identifier, delivering at most once per identifier (§13.1). Required of Notification Providers.
* Keep node-witnessed dispatch records distinct from provider-claimed delivery confirmations, and never let the latter overwrite the former (§14.1).
* Reject a delivery confirmation that is unsigned, names an unregistered provider, or names a notification the verifying node never dispatched (§14.1).

---

## 23. Relationship to Other Specifications

The Notification Protocol connects OpenFiat protocol events with real-world communication channels.

```text id="notification-architecture"
            Protocol Events
                  │
                  ▼
             OFS-6000
     Notification Protocol
                  │
      ┌───────────┼────────────┬────────────┐
      ▼           ▼            ▼            ▼
   Email        SMS       Telegram     Webhooks
                  │
                  ▼
            End Users &
          Infrastructure
```

---

## 24. Summary

The OpenFiat Notification Protocol transforms protocol events into real-world communication without introducing centralized infrastructure.

By allowing any individual, company, or community organization to operate standardized Notification Providers, OpenFiat creates a resilient, permissionless messaging ecosystem capable of serving users across every region and communication preference.

Whether notifications are delivered through email, SMS, Telegram, Discord, webhooks, or future communication technologies, the protocol ensures that delivery infrastructure remains decentralized, interoperable, and independent of any single provider.

The Notification Protocol answers one fundamental question:

**"How can participants be informed about critical marketplace events in real time without relying on centralized notification services?"**
