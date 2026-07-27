# OFS-6000 — OpenFiat Notification Protocol (ONP)

**Document ID:** OFS-6000

**Title:** OpenFiat Notification Protocol

**Version:** 1.0.0 (Draft)

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

---

## 15. Retry Policy

Providers SHOULD retry transient failures.

Examples include:

* Temporary network failure
* Provider unavailable
* Mail server timeout
* SMS gateway congestion

Retry strategies remain implementation-specific.

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

---

## 20. Security Considerations

Implementations MUST protect against:

* Fake notifications
* Message tampering
* Replay attacks
* Unauthorized subscriptions
* Provider impersonation
* Spam delivery

All notification events MUST remain cryptographically verifiable.

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
