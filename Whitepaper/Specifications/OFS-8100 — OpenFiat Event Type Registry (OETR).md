# OFS-8100 — OpenFiat Event Type Registry (OETR)

**Document ID:** OFS-8100

**Title:** OpenFiat Event Type Registry

**Version:** 1.0.0 (Draft)

**Status:** Draft Standard

**Category:** Core Protocol Registry

**Depends On:** All OpenFiat Protocol Specifications

---

## Abstract

The OpenFiat Event Type Registry (OETR) defines the canonical set of protocol events exchanged, persisted, indexed, and consumed throughout the OpenFiat ecosystem.

Every protocol state transition SHALL emit one or more standardized events from this registry.

These events provide the foundation for:

* Network gossip
* Node synchronization
* State reconstruction
* Audit logs
* Explorer indexing
* SDK subscriptions
* Wallet notifications
* Merchant dashboards
* Analytics
* Monitoring
* Event sourcing
* Future protocol extensions

---

## 1. Event Model

Every event SHALL contain a common envelope.

```json
{
  "id": "evt_01JABC...",
  "type": "AdvertisementCreated",
  "version": 1,
  "timestamp": "2026-01-01T00:00:00Z",
  "node": "node_xxx",
  "actor": "merchant_xxx",
  "sequence": 1452912,
  "payload": {}
}
```

Every implementation SHALL preserve the event type exactly as defined in this registry.

---

## 2. Event Naming Rules

Events SHALL use PascalCase.

They SHALL represent completed state transitions.

Examples:

AdvertisementCreated

ReservationOpened

SettlementCompleted

IdentityClaimPublished

ProposalCreated

VoteCast

NotificationDelivered

OraclePriceUpdated

RiskAssessmentCompleted

---

## 3. Event Categories

The registry defines the following namespaces.

| Category      | Prefix |
| ------------- | ------ |
| Network       | NET    |
| Storage       | STO    |
| Identity      | ID     |
| Marketplace   | MKT    |
| Advertisement | ADV    |
| Reservation   | RSV    |
| Settlement    | SET    |
| Vault         | VLT    |
| Reputation    | REP    |
| Governance    | GOV    |
| Oracle        | ORA    |
| Risk          | RSK    |
| Notification  | NOT    |
| Wallet        | WAL    |
| Chain Bridge  | CHN    |
| SDK           | SDK    |
| System        | SYS    |

---

## 4. Network Events (NET)

PeerDiscovered

PeerConnected

PeerDisconnected

PeerAuthenticated

PeerRejected

NodeStarted

NodeStopped

NodeRestarted

NodeSynchronized

NodeSynchronizationStarted

NodeSynchronizationCompleted

NodeSynchronizationFailed

SnapshotRequested

SnapshotReceived

SnapshotVerified

SnapshotRejected

SessionEstablished

SessionExpired

HeartbeatReceived

HeartbeatTimeout

ServiceRegistered

ServiceUnregistered

ServiceUpdated

BandwidthLimitReached

RateLimitExceeded

ProtocolVersionMismatch

ReplayAttackDetected

MalformedMessageReceived

NetworkPartitionDetected

CheckpointCreated

CheckpointVerified

CheckpointRejected

---

## 5. Storage Events (STO)

StorageInitialized

StorageRecovered

StorageCorruptionDetected

SnapshotPersisted

SnapshotLoaded

WriteBatchCommitted

WriteBatchRolledBack

CompactionStarted

CompactionCompleted

PruningStarted

PruningCompleted

MigrationStarted

MigrationCompleted

BackupCreated

BackupRestored

DatabaseIntegrityVerified

---

## 6. Identity Events (ID)

IdentityCreated

IdentityClaimPublished

IdentityClaimUpdated

IdentityClaimRevoked

IdentityVerified

IdentityRejected

VerificationRequested

VerificationCompleted

VerificationExpired

IdentityMerged

IdentitySplit

---

## 7. Marketplace Events (MKT)

MarketplaceInitialized

MarketplaceOpened

MarketplacePaused

MarketplaceResumed

MarketplaceClosed

LiquidityAvailable

LiquidityExhausted

MarketplaceHealthUpdated

---

## 8. Advertisement Events (ADV)

AdvertisementCreated

AdvertisementUpdated

AdvertisementActivated

AdvertisementPaused

AdvertisementResumed

AdvertisementDisabled

AdvertisementDeleted

AdvertisementExpired

AdvertisementMatched

AdvertisementVisibilityChanged

AdvertisementLimitUpdated

AdvertisementPricingUpdated

AdvertisementLiquidityUpdated

AdvertisementPaymentMethodsUpdated

AdvertisementAutoPaused

AdvertisementAutoResumed

AdvertisementValidationFailed

---

## 9. Reservation Events (RSV)

ReservationRequested

ReservationOpened

ReservationAccepted

ReservationRejected

ReservationExpired

ReservationCancelled

ReservationReleased

ReservationExtended

ReservationTimedOut

ReservationDisputed

ReservationCompleted

ReservationFailed

---

## 10. Settlement Events (SET)

SettlementInitiated

SettlementDepositDetected

SettlementDepositConfirmed

SettlementVerificationStarted

SettlementVerificationCompleted

SettlementApproved

SettlementRejected

SettlementCompleted

SettlementFailed

SettlementRefundInitiated

SettlementRefundCompleted

SettlementCancelled

SettlementExpired

SettlementReversed

---

## 11. Liquidity Vault Events (VLT)

VaultCreated

VaultActivated

VaultBalanceLocked

VaultBalanceUnlocked

VaultBalanceDeposited

VaultBalanceWithdrawn

VaultBalanceAdjusted

VaultBalanceReserved

VaultBalanceReleased

VaultThresholdReached

VaultFrozen

VaultUnfrozen

VaultClosed

---

## 12. Reputation Events (REP)

ReputationCreated

ReputationUpdated

ReputationScoreChanged

MerchantRated

BuyerRated

DisputePenaltyApplied

PenaltyExpired

NodeScoreUpdated

TrustScoreCalculated

---

## 13. Governance Events (GOV)

ProposalCreated

ProposalUpdated

ProposalCancelled

ProposalActivated

ProposalExpired

VotingOpened

VoteCast

VoteChanged

VoteRevoked

VotingClosed

ProposalPassed

ProposalRejected

ProposalExecuted

TreasuryPaymentApproved

TreasuryPaymentExecuted

---

## 14. Oracle Events (ORA)

OracleProviderRegistered

OracleProviderRemoved

OracleFeedStarted

OracleFeedStopped

OraclePriceUpdated

OracleHealthUpdated

OracleConsensusReached

OracleConsensusFailed

ExchangeRatePublished

StablecoinStatusChanged

---

## 15. Risk Events (RSK)

RiskProviderRegistered

RiskProviderUnavailable

WalletScreeningStarted

WalletScreeningCompleted

WalletFlagged

WalletCleared

DepositRejected

HighRiskAddressDetected

SanctionsMatchDetected

FraudPatternDetected

RiskScoreUpdated

---

## 16. Notification Events (NOT)

NotificationQueued

NotificationSent

NotificationDelivered

NotificationRead

NotificationFailed

NotificationRetried

NotificationExpired

SubscriptionCreated

SubscriptionUpdated

SubscriptionRemoved

---

## 17. Wallet Events (WAL)

WalletConnected

WalletDisconnected

WalletAuthorized

WalletRevoked

WalletDelegated

WalletDelegationRevoked

WalletBalanceChanged

WalletBackupCompleted

WalletRestored

---

## 18. Chain Bridge Events (CHN)

BlockhashAnnounced

TransactionRelayRequested

TransactionRelayed

---

## 19. SDK Events (SDK)

SDKConnected

SDKDisconnected

SubscriptionStarted

SubscriptionStopped

CacheInvalidated

CacheRefreshed

RPCConnected

RPCDisconnected

---

## 20. System Events (SYS)

ConfigurationLoaded

ConfigurationUpdated

MaintenanceStarted

MaintenanceCompleted

UpgradeStarted

UpgradeCompleted

RollbackStarted

RollbackCompleted

FeatureEnabled

FeatureDisabled

NodeHealthChanged

---

## 21. Event Versioning

Every event SHALL include a schema version.

Schema evolution SHALL preserve backward compatibility whenever practical.

Breaking changes SHALL require a new event version.

---

## 22. Event Ordering

Events SHALL be processed in deterministic sequence order.

Implementations MUST reject duplicate events.

Replay protection SHALL follow OFS-1800.

---

## 23. Extensibility

Future specifications MAY register additional event types.

Existing event names SHALL NEVER be reused for different semantics.

Deprecated event types SHALL remain reserved permanently.

---

## 24. Conformance

A compliant implementation MUST:

* Emit standardized event names.
* Preserve event ordering.
* Preserve event versions.
* Preserve event identifiers.
* Reject unknown mandatory event versions.
* Support event replay.

---

## 25. Summary

The OpenFiat Event Type Registry defines the canonical event vocabulary for the OpenFiat ecosystem.

Every protocol action, state transition, synchronization operation, governance decision, marketplace update, notification, oracle publication, and risk assessment is represented by a standardized event from this registry, enabling deterministic synchronization, interoperable SDKs, explorer indexing, auditability, and event-driven application development across the entire network.
