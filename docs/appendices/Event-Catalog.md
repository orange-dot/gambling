# Event Catalog: е-Играч Event Types and Schemas

**Document Version**: 1.0
**Date**: 2025-10-26
**Technology Stack**: C# .NET 9, MongoDB C# Driver
**Purpose**: Comprehensive catalog of all event types in the event-sourced system
**Architecture**: Event Sourcing + CQRS

---

## Table of Contents

1. [Introduction](#introduction)
2. [Event Structure](#event-structure)
3. [Player Aggregate Events](#player-aggregate-events)
4. [Transaction Aggregate Events](#transaction-aggregate-events)
5. [Limit Aggregate Events](#limit-aggregate-events)
6. [Session Aggregate Events](#session-aggregate-events)
7. [Alert Aggregate Events](#alert-aggregate-events)
8. [Operator Aggregate Events](#operator-aggregate-events)
9. [System Events](#system-events)
10. [Event Versioning Guidelines](#event-versioning-guidelines)
11. [Event Publishing Rules](#event-publishing-rules)

---

## Introduction

This document catalogs all event types in the е-Играч system. Events are the source of truth in our event-sourced architecture and represent facts that have occurred in the past.

### Design Principles

1. **Events are immutable**: Once written, never modified
2. **Events are versioned**: `schemaVersion` field tracks evolution
3. **Events are self-contained**: All necessary data in `eventData`
4. **Events use past tense**: "PlayerRegistered", not "RegisterPlayer"
5. **Events are business-focused**: Represent domain concepts

### Event Categories

| Aggregate Type | Event Count | Purpose |
|----------------|-------------|---------|
| Player | 12 | Player lifecycle and state changes |
| Transaction | 6 | Gambling transactions |
| Limit | 5 | Responsible gambling limits |
| Session | 4 | Gambling sessions |
| Alert | 4 | Fraud and problem gambling alerts |
| Operator | 4 | Operator management |
| System | 3 | System-level events |

---

## Event Structure

### Standard Event Envelope

Every event follows this structure:

```csharp
using MongoDB.Bson;
using MongoDB.Bson.Serialization.Attributes;

/// <summary>
/// Standard event envelope for all events in the system.
/// </summary>
public class Event
{
    [BsonId]
    public ObjectId Id { get; set; }

    /// <summary>
    /// Aggregate instance ID (e.g., player ID, transaction ID).
    /// </summary>
    [BsonElement("aggregateId")]
    public required string AggregateId { get; set; }

    /// <summary>
    /// Aggregate type (Player, Transaction, Limit, Session, etc.).
    /// </summary>
    [BsonElement("aggregateType")]
    public required string AggregateType { get; set; }

    /// <summary>
    /// Event type name (past tense, e.g., "PlayerRegistered").
    /// </summary>
    [BsonElement("eventType")]
    public required string EventType { get; set; }

    /// <summary>
    /// Aggregate version for optimistic locking (increments with each event).
    /// </summary>
    [BsonElement("eventVersion")]
    public int EventVersion { get; set; }

    /// <summary>
    /// Schema version of this event type (for evolution).
    /// </summary>
    [BsonElement("schemaVersion")]
    public int SchemaVersion { get; set; }

    /// <summary>
    /// Event-specific payload (stored as BsonDocument for flexibility).
    /// </summary>
    [BsonElement("eventData")]
    public required BsonDocument EventData { get; set; }

    /// <summary>
    /// Metadata about the event (timestamp, user, correlation ID, etc.).
    /// </summary>
    [BsonElement("metadata")]
    public required EventMetadata Metadata { get; set; }
}

/// <summary>
/// Metadata attached to every event.
/// </summary>
public class EventMetadata
{
    [BsonElement("timestamp")]
    public DateTime Timestamp { get; set; }

    [BsonElement("userId")]
    public required string UserId { get; set; }

    [BsonElement("correlationId")]
    public string? CorrelationId { get; set; }

    [BsonElement("causationId")]
    public string? CausationId { get; set; }

    [BsonElement("ipAddress")]
    public string? IpAddress { get; set; }

    [BsonElement("userAgent")]
    public string? UserAgent { get; set; }
}
```

---

## Player Aggregate Events

### Event Summary

| Event Type | Schema Version | Description | Phase |
|------------|----------------|-------------|-------|
| `PlayerRegistered` | 2 | Player completes registration via ConsentID | 1 |
| `PlayerProfileUpdated` | 1 | Player updates profile information | 2 |
| `PlayerVerified` | 1 | Player identity verified by ConsentID | 1 |
| `PlayerSuspended` | 2 | Player account suspended (manual or automated) | 2 |
| `PlayerUnsuspended` | 1 | Player account reinstated | 2 |
| `SelfExclusionActivated` | 2 | Player activates self-exclusion | 2 |
| `SelfExclusionExpired` | 1 | Self-exclusion period ends | 2 |
| `SelfExclusionViolationDetected` | 1 | Attempt to gamble during self-exclusion | 2 |
| `PlayerDeleted` | 1 | Player requests account deletion (GDPR) | 3 |
| `PlayerAnonymized` | 1 | Player data anonymized after retention period | 3 |
| `ConsentGiven` | 1 | Player gives consent for data processing | 1 |
| `ConsentRevoked` | 1 | Player revokes consent | 3 |

---

### PlayerRegistered

**Description**: Player successfully registers using ConsentID authentication.

**Aggregate**: Player
**Schema Version**: 2 (added `dateOfBirth` in v2)
**Trigger**: Successful ConsentID OAuth flow completion

**Schema (C#)**:

```csharp
/// <summary>
/// Player registration completed via ConsentID.
/// </summary>
public record PlayerRegisteredV2(
    string PlayerId,
    string ConsentIdSubjectId,
    string VerifiedName,
    DateTime? DateOfBirth,  // Added in v2, nullable for upcasted v1 events
    string AddressHash,     // SHA-256 hash of address
    RegistrationChannel Channel,  // Added in v3
    DateTime RegistrationDate
);

public enum RegistrationChannel
{
    MobileApp,
    Web,
    Kiosk
}
```

**Example JSON**:
```json
{
  "aggregateId": "player-a1b2c3d4",
  "aggregateType": "Player",
  "eventType": "PlayerRegistered",
  "eventVersion": 1,
  "schemaVersion": 2,
  "eventData": {
    "playerId": "player-a1b2c3d4",
    "consentIdSubjectId": "consent-xyz123",
    "verifiedName": "Петар Петровић",
    "dateOfBirth": "1990-05-15",
    "addressHash": "sha256:a3f2b9...",
    "channel": "MobileApp",
    "registrationDate": "2025-10-26T14:30:00Z"
  },
  "metadata": {
    "timestamp": "2025-10-26T14:30:00Z",
    "userId": "player-a1b2c3d4",
    "correlationId": "trace-12345",
    "ipAddress": "192.168.1.100",
    "userAgent": "Mozilla/5.0..."
  }
}
```

**Evolution History**:
- **v1** (2025-01-01): Initial version without `dateOfBirth`
- **v2** (2025-07-01): Added `dateOfBirth` for age verification
- **v3** (2025-11-01): Added `channel` to track registration source

---

### PlayerSuspended

**Description**: Player account suspended due to fraud, regulatory violation, or manual intervention.

**Aggregate**: Player
**Schema Version**: 2 (added `autoReinstateDate` in v2)
**Trigger**: Fraud detection, regulatory order, or operator action

**Schema (C#)**:

```csharp
/// <summary>
/// Player account suspended.
/// </summary>
public record PlayerSuspendedV2(
    string PlayerId,
    SuspensionReason Reason,
    string ReasonDetails,
    DateTime SuspendedAt,
    DateTime? AutoReinstateDate,  // Added in v2, null for permanent suspension
    string SuspendedBy  // "SYSTEM", "OPERATOR", or operator user ID
);

public enum SuspensionReason
{
    FraudDetected,
    ProblemGamblingDetected,
    RegulatoryViolation,
    ManualReview,
    SelfRequested,
    PaymentIssue
}
```

**Example JSON**:
```json
{
  "aggregateId": "player-a1b2c3d4",
  "aggregateType": "Player",
  "eventType": "PlayerSuspended",
  "eventVersion": 5,
  "schemaVersion": 2,
  "eventData": {
    "playerId": "player-a1b2c3d4",
    "reason": "FraudDetected",
    "reasonDetails": "Multiple transactions from different locations within 5 minutes",
    "suspendedAt": "2025-10-26T15:00:00Z",
    "autoReinstateDate": "2025-11-26T15:00:00Z",
    "suspendedBy": "SYSTEM"
  },
  "metadata": {
    "timestamp": "2025-10-26T15:00:00Z",
    "userId": "SYSTEM",
    "correlationId": "fraud-alert-98765"
  }
}
```

**Evolution History**:
- **v1** (2025-01-01): Initial version
- **v2** (2025-08-01): Added `autoReinstateDate` for temporary suspensions

---

### SelfExclusionActivated

**Description**: Player activates self-exclusion for responsible gambling.

**Aggregate**: Player
**Schema Version**: 2 (added `exclusionType` in v2)
**Trigger**: Player request via mobile app or operator

**Schema (C#)**:

```csharp
/// <summary>
/// Player activates self-exclusion.
/// </summary>
public record SelfExclusionActivatedV2(
    string PlayerId,
    SelfExclusionType ExclusionType,  // Added in v2
    int DurationMonths,  // 1, 3, 6, 12, or 0 for permanent
    DateTime StartDate,
    DateTime? EndDate,  // Null for permanent
    string ActivatedVia  // "MobileApp", "Web", "Operator", "Phone"
);

public enum SelfExclusionType
{
    Temporary,   // Fixed duration
    Permanent,   // Indefinite
    Cooldown     // Short break (24-72 hours)
}
```

**Example JSON**:
```json
{
  "aggregateId": "player-a1b2c3d4",
  "aggregateType": "Player",
  "eventType": "SelfExclusionActivated",
  "eventVersion": 8,
  "schemaVersion": 2,
  "eventData": {
    "playerId": "player-a1b2c3d4",
    "exclusionType": "Temporary",
    "durationMonths": 6,
    "startDate": "2025-10-26T16:00:00Z",
    "endDate": "2026-04-26T16:00:00Z",
    "activatedVia": "MobileApp"
  },
  "metadata": {
    "timestamp": "2025-10-26T16:00:00Z",
    "userId": "player-a1b2c3d4"
  }
}
```

**Evolution History**:
- **v1** (2025-01-01): Initial version
- **v2** (2025-09-01): Added `exclusionType` to differentiate temporary vs permanent

---

### ConsentGiven

**Description**: Player gives explicit consent for data processing (GDPR compliance).

**Aggregate**: Player
**Schema Version**: 1
**Trigger**: Registration or consent update

**Schema (C#)**:

```csharp
/// <summary>
/// Player gives consent for data processing.
/// </summary>
public record ConsentGivenV1(
    string PlayerId,
    ConsentType ConsentType,
    string ConsentVersion,  // Version of consent document shown
    DateTime ConsentDate,
    string IpAddress,
    bool ElectronicSignature
);

public enum ConsentType
{
    DataProcessing,
    MarketingCommunications,
    DataSharing,
    Profiling
}
```

**Example JSON**:
```json
{
  "aggregateId": "player-a1b2c3d4",
  "aggregateType": "Player",
  "eventType": "ConsentGiven",
  "eventVersion": 2,
  "schemaVersion": 1,
  "eventData": {
    "playerId": "player-a1b2c3d4",
    "consentType": "DataProcessing",
    "consentVersion": "1.2",
    "consentDate": "2025-10-26T14:30:00Z",
    "ipAddress": "192.168.1.100",
    "electronicSignature": true
  },
  "metadata": {
    "timestamp": "2025-10-26T14:30:00Z",
    "userId": "player-a1b2c3d4"
  }
}
```

---

## Transaction Aggregate Events

### Event Summary

| Event Type | Schema Version | Description | Phase |
|------------|----------------|-------------|-------|
| `TransactionAuthorized` | 3 | Transaction approved and authorized | 2 |
| `TransactionRejected` | 2 | Transaction rejected (limit exceeded, fraud, etc.) | 2 |
| `TransactionCompleted` | 1 | Transaction finalized by operator | 2 |
| `TransactionCancelled` | 1 | Transaction cancelled by player or operator | 2 |
| `TransactionRefunded` | 1 | Transaction refunded to player | 3 |
| `TransactionFlagged` | 1 | Transaction flagged for manual review | 2 |

---

### TransactionAuthorized

**Description**: Transaction authorized after limit checks and fraud detection.

**Aggregate**: Transaction
**Schema Version**: 3 (added device and location info in v3)
**Trigger**: Successful transaction validation

**Schema (C#)**:

```csharp
/// <summary>
/// Transaction authorized.
/// </summary>
public record TransactionAuthorizedV3(
    string TransactionId,
    string PlayerId,
    string OperatorId,
    long Amount,  // In smallest currency unit (dinars)
    TransactionType Type,
    DateTime AuthorizedAt,
    string QrCodeToken,  // For operator verification
    DateTime QrCodeExpiresAt,
    // Added in v3
    string DeviceId,
    string Location,  // Geolocation or venue ID
    string OperatorVenueId
);

public enum TransactionType
{
    Deposit,
    Withdrawal,
    Bet,
    Win,
    Refund
}
```

**Example JSON**:
```json
{
  "aggregateId": "tx-f9e8d7c6",
  "aggregateType": "Transaction",
  "eventType": "TransactionAuthorized",
  "eventVersion": 1,
  "schemaVersion": 3,
  "eventData": {
    "transactionId": "tx-f9e8d7c6",
    "playerId": "player-a1b2c3d4",
    "operatorId": "operator-123",
    "amount": 5000,
    "type": "Bet",
    "authorizedAt": "2025-10-26T17:00:00Z",
    "qrCodeToken": "qr-token-abc123",
    "qrCodeExpiresAt": "2025-10-26T17:01:00Z",
    "deviceId": "device-fingerprint-xyz",
    "location": "venue-belgrade-001",
    "operatorVenueId": "venue-belgrade-001"
  },
  "metadata": {
    "timestamp": "2025-10-26T17:00:00Z",
    "userId": "player-a1b2c3d4",
    "correlationId": "tx-flow-12345"
  }
}
```

**Evolution History**:
- **v1** (2025-01-01): Initial version
- **v2** (2025-06-01): Added QR code fields
- **v3** (2025-10-01): Added device ID, location, venue ID for fraud detection

---

### TransactionRejected

**Description**: Transaction rejected due to limit exceeded, fraud detection, or other reasons.

**Aggregate**: Transaction
**Schema Version**: 2 (added `fraudScore` in v2)
**Trigger**: Failed transaction validation

**Schema (C#)**:

```csharp
/// <summary>
/// Transaction rejected.
/// </summary>
public record TransactionRejectedV2(
    string TransactionId,
    string PlayerId,
    string OperatorId,
    long Amount,
    TransactionType Type,
    RejectionReason Reason,
    string ReasonDetails,
    DateTime RejectedAt,
    float? FraudScore  // Added in v2, ML fraud score if applicable
);

public enum RejectionReason
{
    DailyLimitExceeded,
    MonthlyLimitExceeded,
    InsufficientBalance,
    FraudSuspicion,
    SelfExcluded,
    AccountSuspended,
    OperatorError,
    TechnicalError
}
```

**Example JSON**:
```json
{
  "aggregateId": "tx-f9e8d7c6",
  "aggregateType": "Transaction",
  "eventType": "TransactionRejected",
  "eventVersion": 1,
  "schemaVersion": 2,
  "eventData": {
    "transactionId": "tx-f9e8d7c6",
    "playerId": "player-a1b2c3d4",
    "operatorId": "operator-123",
    "amount": 15000,
    "type": "Bet",
    "reason": "DailyLimitExceeded",
    "reasonDetails": "Daily limit: 10,000 RSD, current spent: 8,000 RSD, requested: 15,000 RSD",
    "rejectedAt": "2025-10-26T17:05:00Z",
    "fraudScore": null
  },
  "metadata": {
    "timestamp": "2025-10-26T17:05:00Z",
    "userId": "player-a1b2c3d4"
  }
}
```

**Evolution History**:
- **v1** (2025-01-01): Initial version
- **v2** (2025-10-01): Added `fraudScore` from ML model

---

## Limit Aggregate Events

### Event Summary

| Event Type | Schema Version | Description | Phase |
|------------|----------------|-------------|-------|
| `LimitSet` | 2 | Player sets or updates limits | 2 |
| `LimitIncreaseRequested` | 1 | Player requests limit increase | 2 |
| `LimitIncreaseApproved` | 1 | Limit increase approved after cooldown | 2 |
| `LimitDecreaseApplied` | 1 | Limit decrease applied immediately | 2 |
| `LimitExceeded` | 1 | Player attempts to exceed limit | 2 |

---

### LimitSet

**Description**: Player sets or updates daily/monthly limits.

**Aggregate**: Limit
**Schema Version**: 2 (added limit types in v2)
**Trigger**: Player action via mobile app

**Schema (C#)**:

```csharp
/// <summary>
/// Player sets or updates limits.
/// </summary>
public record LimitSetV2(
    string LimitId,
    string PlayerId,
    LimitType LimitType,  // Added in v2
    long? DailyLimit,     // Nullable to support partial updates
    long? MonthlyLimit,
    DateTime EffectiveDate,  // Immediate for decreases, delayed for increases
    DateTime SetAt,
    string SetVia  // "MobileApp", "Web", "Operator"
);

public enum LimitType
{
    Spending,   // Total spending limit
    Deposit,    // Deposit limit
    Loss,       // Loss limit
    Session     // Session duration limit (minutes)
}
```

**Example JSON**:
```json
{
  "aggregateId": "limit-player-a1b2c3d4",
  "aggregateType": "Limit",
  "eventType": "LimitSet",
  "eventVersion": 3,
  "schemaVersion": 2,
  "eventData": {
    "limitId": "limit-player-a1b2c3d4",
    "playerId": "player-a1b2c3d4",
    "limitType": "Spending",
    "dailyLimit": 10000,
    "monthlyLimit": 50000,
    "effectiveDate": "2025-10-26T18:00:00Z",
    "setAt": "2025-10-26T18:00:00Z",
    "setVia": "MobileApp"
  },
  "metadata": {
    "timestamp": "2025-10-26T18:00:00Z",
    "userId": "player-a1b2c3d4"
  }
}
```

**Evolution History**:
- **v1** (2025-01-01): Initial version with only spending limits
- **v2** (2025-08-01): Added `limitType` to support deposit, loss, session limits

---

### LimitExceeded

**Description**: Player attempts transaction that would exceed their limit.

**Aggregate**: Limit
**Schema Version**: 1
**Trigger**: Transaction validation failure

**Schema (C#)**:

```csharp
/// <summary>
/// Player attempts to exceed limit.
/// </summary>
public record LimitExceededV1(
    string LimitId,
    string PlayerId,
    LimitType LimitType,
    long CurrentLimit,
    long CurrentSpent,
    long AttemptedAmount,
    DateTime AttemptedAt
);
```

**Example JSON**:
```json
{
  "aggregateId": "limit-player-a1b2c3d4",
  "aggregateType": "Limit",
  "eventType": "LimitExceeded",
  "eventVersion": 5,
  "schemaVersion": 1,
  "eventData": {
    "limitId": "limit-player-a1b2c3d4",
    "playerId": "player-a1b2c3d4",
    "limitType": "Spending",
    "currentLimit": 10000,
    "currentSpent": 8000,
    "attemptedAmount": 5000,
    "attemptedAt": "2025-10-26T18:30:00Z"
  },
  "metadata": {
    "timestamp": "2025-10-26T18:30:00Z",
    "userId": "player-a1b2c3d4"
  }
}
```

---

## Session Aggregate Events

### Event Summary

| Event Type | Schema Version | Description | Phase |
|------------|----------------|-------------|-------|
| `SessionStarted` | 2 | Player starts gambling session | 2 |
| `SessionEnded` | 2 | Player ends gambling session | 2 |
| `SessionTimeoutWarning` | 1 | Warning about session timeout | 2 |
| `SessionForcedBreak` | 1 | Mandatory break triggered | 2 |

---

### SessionStarted

**Description**: Player starts a gambling session.

**Aggregate**: Session
**Schema Version**: 2 (added venue info in v2)
**Trigger**: First transaction or explicit session start

**Schema (C#)**:

```csharp
/// <summary>
/// Player starts gambling session.
/// </summary>
public record SessionStartedV2(
    string SessionId,
    string PlayerId,
    DateTime StartTime,
    string DeviceId,
    string DeviceType,  // "Mobile", "Web", "Kiosk"
    // Added in v2
    string? OperatorVenueId,
    string? Location
);
```

**Example JSON**:
```json
{
  "aggregateId": "session-xyz789",
  "aggregateType": "Session",
  "eventType": "SessionStarted",
  "eventVersion": 1,
  "schemaVersion": 2,
  "eventData": {
    "sessionId": "session-xyz789",
    "playerId": "player-a1b2c3d4",
    "startTime": "2025-10-26T19:00:00Z",
    "deviceId": "device-fingerprint-xyz",
    "deviceType": "Mobile",
    "operatorVenueId": "venue-belgrade-001",
    "location": "Belgrade, Serbia"
  },
  "metadata": {
    "timestamp": "2025-10-26T19:00:00Z",
    "userId": "player-a1b2c3d4"
  }
}
```

**Evolution History**:
- **v1** (2025-01-01): Initial version
- **v2** (2025-09-01): Added venue and location info

---

### SessionEnded

**Description**: Player ends gambling session (explicit or timeout).

**Aggregate**: Session
**Schema Version**: 2 (added statistics in v2)
**Trigger**: Player action, timeout, or forced break

**Schema (C#)**:

```csharp
/// <summary>
/// Player ends gambling session.
/// </summary>
public record SessionEndedV2(
    string SessionId,
    string PlayerId,
    DateTime EndTime,
    TimeSpan Duration,
    SessionEndReason EndReason,
    // Added in v2: Session statistics
    SessionStatistics? Statistics
);

public enum SessionEndReason
{
    PlayerEnded,
    Timeout,
    ForcedBreak,
    SystemError
}

public record SessionStatistics(
    int TransactionCount,
    long TotalWagered,
    long TotalWon,
    long NetProfit  // Positive for win, negative for loss
);
```

**Example JSON**:
```json
{
  "aggregateId": "session-xyz789",
  "aggregateType": "Session",
  "eventType": "SessionEnded",
  "eventVersion": 2,
  "schemaVersion": 2,
  "eventData": {
    "sessionId": "session-xyz789",
    "playerId": "player-a1b2c3d4",
    "endTime": "2025-10-26T20:30:00Z",
    "duration": "01:30:00",
    "endReason": "PlayerEnded",
    "statistics": {
      "transactionCount": 15,
      "totalWagered": 25000,
      "totalWon": 18000,
      "netProfit": -7000
    }
  },
  "metadata": {
    "timestamp": "2025-10-26T20:30:00Z",
    "userId": "player-a1b2c3d4"
  }
}
```

**Evolution History**:
- **v1** (2025-01-01): Initial version without statistics
- **v2** (2025-10-01): Added session statistics for analytics

---

## Alert Aggregate Events

### Event Summary

| Event Type | Schema Version | Description | Phase |
|------------|----------------|-------------|-------|
| `FraudAlertRaised` | 1 | Fraud detected by ML model | 2 |
| `ProblemGamblingAlertRaised` | 1 | Problem gambling detected | 2 |
| `AlertAcknowledged` | 1 | Alert reviewed by operator | 2 |
| `AlertResolved` | 1 | Alert resolved with action taken | 2 |

---

### FraudAlertRaised

**Description**: ML model detects suspicious fraud pattern.

**Aggregate**: Alert
**Schema Version**: 1
**Trigger**: Flink fraud detection job

**Schema (C#)**:

```csharp
/// <summary>
/// Fraud alert raised by ML model.
/// </summary>
public record FraudAlertRaisedV1(
    string AlertId,
    string PlayerId,
    string? TransactionId,  // Null if pattern-based alert
    float FraudScore,  // 0.0 to 1.0
    FraudRiskLevel RiskLevel,
    string[] DetectedPatterns,  // e.g., ["VelocityAnomaly", "LocationChange"]
    DateTime RaisedAt,
    AlertPriority Priority
);

public enum FraudRiskLevel
{
    Low,
    Medium,
    High,
    Critical
}

public enum AlertPriority
{
    Info,
    Low,
    Medium,
    High,
    Critical
}
```

**Example JSON**:
```json
{
  "aggregateId": "alert-fraud-12345",
  "aggregateType": "Alert",
  "eventType": "FraudAlertRaised",
  "eventVersion": 1,
  "schemaVersion": 1,
  "eventData": {
    "alertId": "alert-fraud-12345",
    "playerId": "player-a1b2c3d4",
    "transactionId": "tx-f9e8d7c6",
    "fraudScore": 0.87,
    "riskLevel": "High",
    "detectedPatterns": [
      "VelocityAnomaly",
      "LocationChange",
      "DeviceFingerprint Mismatch"
    ],
    "raisedAt": "2025-10-26T21:00:00Z",
    "priority": "High"
  },
  "metadata": {
    "timestamp": "2025-10-26T21:00:00Z",
    "userId": "SYSTEM",
    "correlationId": "flink-job-fraud-detection"
  }
}
```

---

### ProblemGamblingAlertRaised

**Description**: ML model detects problem gambling behavior.

**Aggregate**: Alert
**Schema Version**: 1
**Trigger**: Flink problem gambling detection job

**Schema (C#)**:

```csharp
/// <summary>
/// Problem gambling alert raised by ML model.
/// </summary>
public record ProblemGamblingAlertRaisedV1(
    string AlertId,
    string PlayerId,
    float RiskScore,  // 0.0 to 1.0
    ProblemGamblingRiskLevel RiskLevel,
    string[] DetectedBehaviors,  // e.g., ["LossChasing", "ExtendedSessions"]
    DateTime RaisedAt,
    AlertPriority Priority,
    string RecommendedAction  // "MandatoryBreak", "LimitReduction", "SelfExclusionSuggestion"
);

public enum ProblemGamblingRiskLevel
{
    Low,
    Moderate,
    High,
    Severe
}
```

**Example JSON**:
```json
{
  "aggregateId": "alert-pg-67890",
  "aggregateType": "Alert",
  "eventType": "ProblemGamblingAlertRaised",
  "eventVersion": 1,
  "schemaVersion": 1,
  "eventData": {
    "alertId": "alert-pg-67890",
    "playerId": "player-a1b2c3d4",
    "riskScore": 0.82,
    "riskLevel": "High",
    "detectedBehaviors": [
      "LossChasing",
      "ExtendedSessions",
      "IncreasedFrequency",
      "LimitIncreaseRequests"
    ],
    "raisedAt": "2025-10-26T21:30:00Z",
    "priority": "High",
    "recommendedAction": "MandatoryBreak"
  },
  "metadata": {
    "timestamp": "2025-10-26T21:30:00Z",
    "userId": "SYSTEM"
  }
}
```

---

## Operator Aggregate Events

### Event Summary

| Event Type | Schema Version | Description | Phase |
|------------|----------------|-------------|-------|
| `OperatorRegistered` | 1 | Operator registered in system | 1 |
| `OperatorSuspended` | 1 | Operator license suspended | 2 |
| `OperatorVenueAdded` | 1 | New gambling venue registered | 2 |
| `OperatorVenueRemoved` | 1 | Venue deregistered | 3 |

---

### OperatorRegistered

**Description**: Gambling operator registered in the system.

**Aggregate**: Operator
**Schema Version**: 1
**Trigger**: Regulatory approval and onboarding

**Schema (C#)**:

```csharp
/// <summary>
/// Operator registered in system.
/// </summary>
public record OperatorRegisteredV1(
    string OperatorId,
    string CompanyName,
    string TaxId,
    string LicenseNumber,
    DateTime LicenseExpiryDate,
    OperatorType OperatorType,
    string ContactEmail,
    string ContactPhone,
    DateTime RegisteredAt
);

public enum OperatorType
{
    OnlineCasino,
    PhysicalVenue,
    Bookmaker,
    Lottery,
    PokerRoom
}
```

**Example JSON**:
```json
{
  "aggregateId": "operator-123",
  "aggregateType": "Operator",
  "eventType": "OperatorRegistered",
  "eventVersion": 1,
  "schemaVersion": 1,
  "eventData": {
    "operatorId": "operator-123",
    "companyName": "Belgrade Casino DOO",
    "taxId": "12345678",
    "licenseNumber": "GLA-2025-001",
    "licenseExpiryDate": "2030-12-31",
    "operatorType": "PhysicalVenue",
    "contactEmail": "contact@belgradecasino.rs",
    "contactPhone": "+381111234567",
    "registeredAt": "2025-10-26T22:00:00Z"
  },
  "metadata": {
    "timestamp": "2025-10-26T22:00:00Z",
    "userId": "SYSTEM"
  }
}
```

---

## System Events

### Event Summary

| Event Type | Schema Version | Description | Phase |
|------------|----------------|-------------|-------|
| `SystemMaintenanceScheduled` | 1 | System maintenance scheduled | All |
| `SystemMaintenanceCompleted` | 1 | System maintenance completed | All |
| `DataRetentionExecuted` | 1 | Data retention policy executed | 3 |

---

### SystemMaintenanceScheduled

**Description**: System maintenance window scheduled.

**Aggregate**: System
**Schema Version**: 1
**Trigger**: Operations team

**Schema (C#)**:

```csharp
/// <summary>
/// System maintenance scheduled.
/// </summary>
public record SystemMaintenanceScheduledV1(
    string MaintenanceId,
    DateTime ScheduledStart,
    DateTime ScheduledEnd,
    MaintenanceType Type,
    string Description,
    bool ServiceInterruption,  // Whether services will be unavailable
    string ScheduledBy
);

public enum MaintenanceType
{
    DatabaseMigration,
    SoftwareUpgrade,
    HardwareMaintenance,
    SecurityPatch,
    PerformanceTuning
}
```

---

## Event Versioning Guidelines

### When to Increment Schema Version

**Increment `schemaVersion`** when:
- ✅ Adding new fields to `eventData`
- ✅ Removing fields from `eventData`
- ✅ Renaming fields
- ✅ Changing field types
- ✅ Changing field semantics (e.g., amount now includes tax)

**Do NOT increment** when:
- ❌ Adding fields to `metadata` (not part of domain)
- ❌ Fixing typos in documentation
- ❌ Changing event handler logic (code change, not schema)

### Upcasting Strategy

All old event versions must be supported indefinitely. Use upcasters to transform old events to new schema when loading from event store.

**Example Upcaster** (v1 → v2):
```csharp
public class PlayerRegisteredUpcaster
{
    public PlayerRegisteredV2 Upcast(Event @event)
    {
        if (@event.SchemaVersion == 1)
        {
            var v1Data = @event.EventData.ToObject<PlayerRegisteredV1>();
            return new PlayerRegisteredV2(
                PlayerId: v1Data.PlayerId,
                ConsentIdSubjectId: v1Data.ConsentIdSubjectId,
                VerifiedName: v1Data.VerifiedName,
                DateOfBirth: null,  // Default for old events
                AddressHash: v1Data.AddressHash,
                Channel: RegistrationChannel.MobileApp,  // Default
                RegistrationDate: v1Data.RegistrationDate
            );
        }
        return @event.EventData.ToObject<PlayerRegisteredV2>();
    }
}
```

---

## Event Publishing Rules

### Kafka Topic Routing

Events are published to Kafka topics based on aggregate type:

| Aggregate Type | Kafka Topic |
|----------------|-------------|
| Player | `eigrac.events.player` |
| Transaction | `eigrac.events.transaction` |
| Limit | `eigrac.events.limit` |
| Session | `eigrac.events.session` |
| Alert | `eigrac.events.alert` |
| Operator | `eigrac.events.operator` |
| System | `eigrac.events.system` |

### Partitioning Strategy

Events are partitioned by `aggregateId` to ensure ordering per aggregate:

```csharp
// Kafka producer configuration
var config = new ProducerConfig
{
    BootstrapServers = "kafka-cluster:9092",
    // Use aggregateId as partition key
    Partitioner = Partitioner.Murmur2
};

// Produce event
await producer.ProduceAsync(
    topic: "eigrac.events.player",
    message: new Message<string, string>
    {
        Key = @event.AggregateId,  // Ensures ordering per player
        Value = JsonSerializer.Serialize(@event)
    }
);
```

### Ordering Guarantees

- ✅ **Within aggregate**: Events for same aggregate ID are strictly ordered
- ✅ **Within partition**: Events in same partition are ordered
- ❌ **Across aggregates**: No ordering guarantees between different players

---

## Document Control

**Version**: 1.0
**Author**: е-Играч Architecture Team
**Last Updated**: 2025-10-26
**Status**: Ready for Implementation
**Tech Stack**: C# .NET 9, MongoDB C# Driver

**Related Documents**:
- [Event Versioning Playbook](./Event-Versioning-Playbook.md)
- [MongoDB Schema Design](./MongoDB-Schema-Design.md)
- [CQRS Implementation Guide](./CQRS-Implementation-Guide.md) (to be created)

**Next Steps**:
1. Review event types with domain experts
2. Validate event schemas with C# team
3. Implement upcasters for existing events
4. Generate code from event catalog
5. Set up Kafka topics and partitioning

---

**END OF DOCUMENT**
