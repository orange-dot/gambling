# Event Versioning Playbook: е-Играч Event Schema Evolution

**Document Version**: 1.1
**Date**: 2025-10-25
**Technology Stack**: C# .NET 9, Dapper, MassTransit
**Purpose**: Guide for managing event schema changes in event-sourced systems
**Architecture**: Event Sourcing + CQRS

---

## Table of Contents

1. [Introduction](#introduction)
2. [Core Principles](#core-principles)
3. [Event Schema Structure](#event-schema-structure)
4. [Versioning Strategy](#versioning-strategy)
5. [Upcasting Patterns](#upcasting-patterns)
6. [Schema Evolution Workflows](#schema-evolution-workflows)
7. [Testing Strategy](#testing-strategy)
8. [Migration Procedures](#migration-procedures)
9. [Governance & Approval](#governance--approval)
10. [Tools & Utilities](#tools--utilities)

---

## Introduction

### Why Event Versioning Matters

In event-sourced systems, **events are immutable** and stored forever (7+ years for е-Играч). As business requirements evolve, event schemas must change. Unlike traditional databases where you can migrate data with SQL scripts, **you cannot modify historical events**.

**Challenge**: How do we evolve event schemas without breaking the ability to replay old events?

**Solution**: **Event Versioning** + **Upcasting**

### Key Concepts

| Concept | Definition | Example |
|---------|------------|---------|
| **Event** | Immutable record of state change | `PlayerRegistered`, `TransactionAuthorized` |
| **Schema Version** | Version number for event structure | `schemaVersion: 1`, `schemaVersion: 2` |
| **Upcasting** | Transform old event to new schema on read | Convert v1 event to v2 format when loading |
| **Downcasting** | Transform new event to old schema (rare) | Support legacy systems reading new events |
| **Breaking Change** | Change incompatible with old code | Rename field, change data type, remove field |
| **Non-Breaking Change** | Additive change, backward compatible | Add optional field with default value |

---

## Core Principles

### 1. **Events Are Immutable**

- Once written, events **NEVER** change
- No updates, no deletes (only appends)
- Historical events remain in their original schema version

### 2. **Schema Versions Are Explicit**

Every event has a `schemaVersion` field:

```javascript
{
  eventType: "PlayerRegistered",
  schemaVersion: 2,  // Explicit version
  eventData: { ... }
}
```

### 3. **Old Events Must Always Be Readable**

- Code must handle **all schema versions** ever created
- Upcasters transform old events to current schema on read
- Never assume events are in latest schema

### 4. **New Versions Are Additive**

**Prefer**:
- Add new fields (with defaults for old events)
- Add new event types

**Avoid**:
- Rename fields (breaks old code)
- Remove fields (loses data)
- Change field types (type mismatch)

### 5. **Test Event Replay with Mixed Versions**

- Integration tests must replay events with v1, v2, v3 mixed together
- Ensures upcasters work correctly
- Validates aggregate can be rebuilt from any combination of event versions

---

## Event Schema Structure

### Standard Event Format

```javascript
// MongoDB BSON representation
{
  _id: ObjectId(),
  aggregateId: "player-uuid",
  aggregateType: "Player",
  eventType: "PlayerRegistered",
  eventVersion: 1,  // Aggregate version (optimistic locking)

  // SCHEMA VERSION (critical for evolution)
  schemaVersion: 2,

  eventData: {
    // Event-specific payload
    playerId: "player-uuid",
    consentIdSubjectId: "consent-xyz123",
    verifiedName: "Petar Petrović",
    dateOfBirth: ISODate("1990-01-15"),  // Added in v2
    addressHash: "sha256:abc123..."
  },

  metadata: {
    timestamp: ISODate("2025-10-25T14:30:00Z"),
    userId: "player-uuid",
    // ...
  }
}
```

**C# Representation**:

```csharp
using MongoDB.Bson;
using MongoDB.Bson.Serialization.Attributes;

public class Event
{
    [BsonId]
    public ObjectId Id { get; set; }

    [BsonElement("aggregateId")]
    public required string AggregateId { get; set; }

    [BsonElement("aggregateType")]
    public required string AggregateType { get; set; }

    [BsonElement("eventType")]
    public required string EventType { get; set; }

    [BsonElement("eventVersion")]
    public int EventVersion { get; set; }

    [BsonElement("schemaVersion")]
    public int SchemaVersion { get; set; }

    [BsonElement("eventData")]
    public required BsonDocument EventData { get; set; }

    [BsonElement("metadata")]
    public required EventMetadata Metadata { get; set; }
}

public class EventMetadata
{
    [BsonElement("timestamp")]
    public DateTime Timestamp { get; set; }

    [BsonElement("userId")]
    public required string UserId { get; set; }

    [BsonElement("correlationId")]
    public string? CorrelationId { get; set; }
}
```

### Schema Version Field

**Location**: Top-level field in event document

**Type**: Integer (1, 2, 3, ...)

**Semantics**:
- **v1**: Original schema
- **v2**: First evolution (e.g., added `dateOfBirth` field)
- **v3**: Second evolution (e.g., added `addressHash` field)

**Increment When**:
- Adding/removing fields in `eventData`
- Changing field types
- Renaming fields
- Changing field semantics

**Do NOT Increment When**:
- Adding fields to `metadata` (not part of domain model)
- Fixing typos in comments/documentation
- Changing event handler logic (code change, not schema change)

---

## Versioning Strategy

### Semantic Versioning for Events

**Format**: `schemaVersion` = Integer (1, 2, 3, ...)

**When to Increment**:

| Change Type | Increment? | Example |
|-------------|------------|---------|
| Add optional field | Yes → v2 | Added `dateOfBirth` field |
| Add required field | Yes → v2 | Added `verifiedName` field |
| Remove field | Yes → v2 | Removed deprecated `nickname` field |
| Rename field | Yes → v2 | Renamed `name` to `verifiedName` |
| Change field type | Yes → v2 | Changed `dateOfBirth` from String to ISODate |
| Change field semantics | Yes → v2 | `amount` now includes tax (was excluding tax) |
| Add new event type | No | New event starts at v1 |
| Fix bug in event handler | No | Code change, not schema change |

### Version Registry

**Centralized Catalog**: `docs/appendices/Event-Catalog.md`

Example entry:

```markdown
## PlayerRegistered

### Version 1 (2025-01-01 to 2025-06-30)

**Fields**:
- `playerId`: String (UUID)
- `consentIdSubjectId`: String
- `verifiedName`: String
- `addressHash`: String (SHA256)

**Deprecated**: 2025-06-30 (replaced by v2)

### Version 2 (2025-07-01 onwards)

**Added Fields**:
- `dateOfBirth`: ISODate (age verification)

**Migration**: v1 events set `dateOfBirth: null` (fetched from ConsentID on first use)

**Status**: Current
```

---

## Upcasting Patterns

### What is Upcasting?

**Upcasting** = Transform old event schema to new schema when reading from event store.

**When**: During aggregate loading (replaying events to rebuild state)

**Where**: In **event handler** before applying event to aggregate

### Pattern 1: Add Field with Default Value

**Scenario**: v1 events missing `dateOfBirth` field (added in v2)

**v1 Event** (stored in DB):
```javascript
{
  eventType: "PlayerRegistered",
  schemaVersion: 1,
  eventData: {
    playerId: "player-123",
    verifiedName: "Petar Petrović",
    addressHash: "sha256:abc"
  }
}
```

**v2 Event** (current schema):
```javascript
{
  eventType: "PlayerRegistered",
  schemaVersion: 2,
  eventData: {
    playerId: "player-123",
    verifiedName: "Petar Petrović",
    dateOfBirth: ISODate("1990-01-15"),  // New field
    addressHash: "sha256:abc"
  }
}
```

**Upcaster Code** (C# .NET 9):

```csharp
// Event data models
public record PlayerRegisteredV1(
    string PlayerId,
    string VerifiedName,
    string AddressHash
);

public record PlayerRegisteredV2(
    string PlayerId,
    string VerifiedName,
    DateTime? DateOfBirth,  // Nullable for upcasted v1 events
    string AddressHash
);

// Upcaster service
public class PlayerRegisteredUpcaster
{
    public PlayerRegisteredV2 Upcast(Event @event)
    {
        if (@event.SchemaVersion == 1)
        {
            var v1Data = @event.EventData.ToObject<PlayerRegisteredV1>();

            // Upcast: add missing field with default
            return new PlayerRegisteredV2(
                PlayerId: v1Data.PlayerId,
                VerifiedName: v1Data.VerifiedName,
                DateOfBirth: null,  // Default: null (fetch from ConsentID later)
                AddressHash: v1Data.AddressHash
            );
        }

        // Already v2, deserialize as-is
        return @event.EventData.ToObject<PlayerRegisteredV2>();
    }
}
```

**Usage in Event Handler**:

```csharp
public class PlayerAggregate
{
    private readonly PlayerRegisteredUpcaster _upcaster = new();

    public string PlayerId { get; private set; }
    public string Name { get; private set; }
    public DateTime? DateOfBirth { get; private set; }
    public string AddressHash { get; private set; }

    public void Apply(Event @event)
    {
        switch (@event.EventType)
        {
            case "PlayerRegistered":
                var upcastedEvent = _upcaster.Upcast(@event);
                HandlePlayerRegistered(upcastedEvent);
                break;
            // ...
        }
    }

    private void HandlePlayerRegistered(PlayerRegisteredV2 data)
    {
        PlayerId = data.PlayerId;
        Name = data.VerifiedName;
        DateOfBirth = data.DateOfBirth;  // May be null for old events
        AddressHash = data.AddressHash;
    }
}
```

---

### Pattern 2: Rename Field

**Scenario**: Field renamed from `name` to `verifiedName` (v1 → v2)

**v1 Event**:
```javascript
{
  schemaVersion: 1,
  eventData: {
    name: "Petar Petrović"  // Old field name
  }
}
```

**v2 Event**:
```javascript
{
  schemaVersion: 2,
  eventData: {
    verifiedName: "Petar Petrović"  // New field name
  }
}
```

**Upcaster**:

```csharp
public PlayerRegisteredV2 UpcastPlayerRegistered(Event @event)
{
    if (@event.SchemaVersion == 1)
    {
        var v1Data = @event.EventData.ToObject<PlayerRegisteredV1>();

        return new PlayerRegisteredV2(
            PlayerId: v1Data.PlayerId,
            VerifiedName: v1Data.Name,  // Rename: Name → VerifiedName
            DateOfBirth: null,
            AddressHash: v1Data.AddressHash
        );
    }

    return @event.EventData.ToObject<PlayerRegisteredV2>();
}
```

---

### Pattern 3: Change Field Type

**Scenario**: `dateOfBirth` changed from String to ISODate (v2 → v3)

**v2 Event**:
```javascript
{
  schemaVersion: 2,
  eventData: {
    dateOfBirth: "1990-01-15"  // String (ISO 8601 format)
  }
}
```

**v3 Event**:
```javascript
{
  schemaVersion: 3,
  eventData: {
    dateOfBirth: ISODate("1990-01-15")  // MongoDB Date object
  }
}
```

**Upcaster**:

```csharp
public PlayerRegisteredV3 UpcastPlayerRegistered(Event @event)
{
    // Handle v1 → v2 (from previous upcaster)
    var data = UpcastToV2(@event);

    // Handle v2 → v3
    if (@event.SchemaVersion == 2)
    {
        return new PlayerRegisteredV3(
            PlayerId: data.PlayerId,
            VerifiedName: data.VerifiedName,
            DateOfBirth: data.DateOfBirth != null
                ? DateTime.Parse(data.DateOfBirth)  // Parse string to DateTime
                : null,
            AddressHash: data.AddressHash
        );
    }

    // Already v3
    return @event.EventData.ToObject<PlayerRegisteredV3>();
}
```

**Chaining Upcasters**:

```csharp
public PlayerRegisteredV3 UpcastPlayerRegistered(Event @event)
{
    var currentVersion = @event.SchemaVersion;
    object data = @event.EventData;

    // Chain: v1 → v2 → v3
    if (currentVersion == 1)
    {
        data = UpcastV1ToV2(data);
        currentVersion = 2;
    }

    if (currentVersion == 2)
    {
        data = UpcastV2ToV3(data);
        currentVersion = 3;
    }

    return (PlayerRegisteredV3)data;
}
```

---

### Pattern 4: Split Event into Multiple Events

**Scenario**: `PlayerRegistered` event did too much (registration + limit setting). Split into two events (v2).

**v1 Event** (monolithic):
```javascript
{
  eventType: "PlayerRegistered",
  schemaVersion: 1,
  eventData: {
    playerId: "player-123",
    verifiedName: "Petar",
    monthlyLimit: 20000  // Limit setting in registration event
  }
}
```

**v2 Events** (separated):
```javascript
// Event 1: Registration only
{
  eventType: "PlayerRegistered",
  schemaVersion: 2,
  eventData: {
    playerId: "player-123",
    verifiedName: "Petar"
    // monthlyLimit removed
  }
}

// Event 2: Separate limit event
{
  eventType: "LimitSet",
  schemaVersion: 1,
  eventData: {
    playerId: "player-123",
    monthlyLimit: 20000
  }
}
```

**Upcaster** (synthesize missing event):

```csharp
public IEnumerable<Event> UpcastPlayerRegistered(Event @event)
{
    if (@event.SchemaVersion == 1)
    {
        var v1Data = @event.EventData.ToObject<PlayerRegisteredV1>();

        // Return TWO events to replace one v1 event
        yield return new Event
        {
            EventType = "PlayerRegistered",
            SchemaVersion = 2,
            EventData = new PlayerRegisteredV2(
                PlayerId: v1Data.PlayerId,
                VerifiedName: v1Data.VerifiedName,
                DateOfBirth: null,
                AddressHash: v1Data.AddressHash
            )
        };

        yield return new Event
        {
            EventType = "LimitSet",
            SchemaVersion = 1,
            EventData = new LimitSet(
                PlayerId: v1Data.PlayerId,
                MonthlyLimit: v1Data.MonthlyLimit
            )
        };
    }
    else
    {
        // v2: already split, return as-is
        yield return @event;
    }
}
```

**Warning**: Returning multiple events from upcaster is complex. Prefer adding new fields instead.

---

## Schema Evolution Workflows

### Workflow: Adding a New Field

**Scenario**: Add `registrationChannel` field to track how player registered (mobile app, web, kiosk).

**Step 1: Define New Schema (v2)**

```csharp
public record PlayerRegisteredV2(
    string PlayerId,
    string VerifiedName,
    DateTime? DateOfBirth,
    string AddressHash,
    RegistrationChannel Channel  // NEW FIELD
);

public enum RegistrationChannel
{
    MobileApp,
    Web,
    Kiosk
}
```

**Step 2: Update Event Catalog**

Add entry to `Event-Catalog.md`:

```markdown
### Version 3 (2025-11-01 onwards)

**Added Fields**:
- `registrationChannel`: String (enum: mobile_app, web, kiosk)

**Default for v1/v2**: `'mobile_app'` (assume mobile if not specified)

**Status**: Current
```

**Step 3: Write Upcaster**

```csharp
public class PlayerRegisteredUpcaster
{
    public PlayerRegisteredV3 Upcast(Event @event)
    {
        // Chain previous upcasters
        var data = UpcastToV2(@event);

        // v2 → v3: Add registrationChannel
        if (@event.SchemaVersion < 3)
        {
            return new PlayerRegisteredV3(
                PlayerId: data.PlayerId,
                VerifiedName: data.VerifiedName,
                DateOfBirth: data.DateOfBirth,
                AddressHash: data.AddressHash,
                Channel: RegistrationChannel.MobileApp  // Default for old events
            );
        }

        return @event.EventData.ToObject<PlayerRegisteredV3>();
    }

    private PlayerRegisteredV2 UpcastToV2(Event @event)
    {
        // Previous upcaster logic (v1 → v2)
        if (@event.SchemaVersion == 1)
        {
            var v1Data = @event.EventData.ToObject<PlayerRegisteredV1>();
            return new PlayerRegisteredV2(
                PlayerId: v1Data.PlayerId,
                VerifiedName: v1Data.VerifiedName,
                DateOfBirth: null,
                AddressHash: v1Data.AddressHash
            );
        }
        return @event.EventData.ToObject<PlayerRegisteredV2>();
    }
}
```

**Step 4: Update Event Emitters**

```csharp
using MongoDB.Driver;
using MongoDB.Bson;

// Command handler that emits v3 events
public class RegisterPlayerCommandHandler
{
    private readonly IMongoCollection<BsonDocument> _eventStore;

    public RegisterPlayerCommandHandler(IMongoDatabase database)
    {
        _eventStore = database.GetCollection<BsonDocument>("events");
    }

    public async Task HandleAsync(RegisterPlayerCommand command)
    {
        var @event = new BsonDocument
        {
            { "aggregateId", command.PlayerId },
            { "aggregateType", "Player" },
            { "eventType", "PlayerRegistered" },
            { "eventVersion", 1 },
            { "schemaVersion", 3 },  // New version
            { "eventData", new BsonDocument
                {
                    { "playerId", command.PlayerId },
                    { "verifiedName", command.VerifiedName },
                    { "dateOfBirth", command.DateOfBirth },
                    { "addressHash", HashAddress(command.Address) },
                    { "registrationChannel", command.Source.ToString() }  // NEW: capture channel
                }
            },
            { "metadata", new BsonDocument
                {
                    { "timestamp", DateTime.UtcNow },
                    { "userId", command.PlayerId },
                    { "correlationId", command.CorrelationId }
                }
            }
        };

        await _eventStore.InsertOneAsync(@event);
    }

    private string HashAddress(string address)
    {
        using var sha256 = System.Security.Cryptography.SHA256.Create();
        var hash = sha256.ComputeHash(System.Text.Encoding.UTF8.GetBytes(address));
        return $"sha256:{Convert.ToHexString(hash).ToLower()}";
    }
}
```

**Step 5: Test**

```csharp
using Xunit;
using FluentAssertions;
using MongoDB.Bson;

public class PlayerRegisteredUpcasterTests
{
    private readonly PlayerRegisteredUpcaster _upcaster = new();

    [Fact]
    public void Upcast_ShouldConvertV1ToV3()
    {
        // Arrange
        var v1Event = new Event
        {
            SchemaVersion = 1,
            EventType = "PlayerRegistered",
            EventData = BsonDocument.Parse(@"{
                ""playerId"": ""player-123"",
                ""verifiedName"": ""Petar"",
                ""addressHash"": ""sha256:abc""
            }")
        };

        // Act
        var upcastedEvent = _upcaster.Upcast(v1Event);

        // Assert
        upcastedEvent.PlayerId.Should().Be("player-123");
        upcastedEvent.VerifiedName.Should().Be("Petar");
        upcastedEvent.DateOfBirth.Should().BeNull();
        upcastedEvent.Channel.Should().Be(RegistrationChannel.MobileApp);
        upcastedEvent.AddressHash.Should().Be("sha256:abc");
    }

    [Fact]
    public void Upcast_ShouldHandleV3EventsUnchanged()
    {
        // Arrange
        var v3Event = new Event
        {
            SchemaVersion = 3,
            EventType = "PlayerRegistered",
            EventData = BsonDocument.Parse(@"{
                ""playerId"": ""player-456"",
                ""verifiedName"": ""Ana"",
                ""dateOfBirth"": ""1995-05-20"",
                ""addressHash"": ""sha256:def"",
                ""registrationChannel"": ""Kiosk""
            }")
        };

        // Act
        var upcastedEvent = _upcaster.Upcast(v3Event);

        // Assert
        upcastedEvent.PlayerId.Should().Be("player-456");
        upcastedEvent.VerifiedName.Should().Be("Ana");
        upcastedEvent.DateOfBirth.Should().Be(new DateTime(1995, 5, 20));
        upcastedEvent.Channel.Should().Be(RegistrationChannel.Kiosk);
    }
}
```

**Step 6: Deploy**

1. Deploy upcaster code (backward compatible, reads old and new events)
2. Wait 1 week (ensure no issues with old event replay)
3. Deploy event emitters (start writing v3 events)

**Rollback**: If issues arise, revert to emitting v2 events (upcaster remains, handles v2 and v3)

---

### Workflow: Removing a Deprecated Field

**Scenario**: Remove `nickname` field (was never used, deprecated in v2, removed in v3).

**Step 1: Mark as Deprecated (v2)**

In v2, add `[Obsolete]` attribute:

```csharp
public record PlayerRegisteredV2(
    string PlayerId,
    string VerifiedName,
    [property: Obsolete("Will be removed in v3, use VerifiedName")]
    string? Nickname
);
```

**Step 2: Wait 6-12 Months**

- Ensure no code depends on `nickname` field
- Monitor analytics: is field ever queried?
- Communicate deprecation to stakeholders

**Step 3: Remove Field (v3)**

```csharp
public record PlayerRegisteredV3(
    string PlayerId,
    string VerifiedName
    // Nickname removed
);
```

**Step 4: Update Upcaster**

```csharp
public class PlayerRegisteredUpcaster
{
    public PlayerRegisteredV3 Upcast(Event @event)
    {
        if (@event.SchemaVersion < 3)
        {
            var oldData = @event.EventData.ToObject<PlayerRegisteredV2>();

            // Strip deprecated field
            return new PlayerRegisteredV3(
                PlayerId: oldData.PlayerId,
                VerifiedName: oldData.VerifiedName
                // Nickname omitted (removed)
            );
        }

        return @event.EventData.ToObject<PlayerRegisteredV3>();
    }
}
```

**Step 5: Update Event Catalog**

```markdown
### Version 3 (2026-01-01 onwards)

**Removed Fields**:
- `nickname`: String (deprecated in v2, removed in v3)

**Impact**: No impact (field was unused, logged warning in v2)
```

---

## Testing Strategy

### Unit Tests: Upcasters

**Test Each Version Transition**:

```csharp
using Xunit;
using FluentAssertions;
using MongoDB.Bson;

public class PlayerRegisteredUpcasterTests
{
    private readonly PlayerRegisteredUpcaster _upcaster = new();

    // v1 → v2 Tests
    [Fact]
    public void UpcastV1ToV2_ShouldAddDateOfBirthWithNullDefault()
    {
        // Arrange
        var v1Event = CreateV1Event();

        // Act
        var v2Event = _upcaster.UpcastV1ToV2(v1Event);

        // Assert
        v2Event.DateOfBirth.Should().BeNull();
    }

    [Fact]
    public void UpcastV1ToV2_ShouldPreserveExistingFields()
    {
        // Arrange
        var v1Event = CreateV1Event(verifiedName: "Petar");

        // Act
        var v2Event = _upcaster.UpcastV1ToV2(v1Event);

        // Assert
        v2Event.VerifiedName.Should().Be("Petar");
    }

    // v2 → v3 Tests
    [Fact]
    public void UpcastV2ToV3_ShouldAddRegistrationChannelWithDefault()
    {
        // Arrange
        var v2Event = CreateV2Event();

        // Act
        var v3Event = _upcaster.UpcastV2ToV3(v2Event);

        // Assert
        v3Event.Channel.Should().Be(RegistrationChannel.MobileApp);
    }

    // v1 → v3 Chained Tests
    [Fact]
    public void Upcast_ShouldApplyAllUpcastersInSequence()
    {
        // Arrange
        var v1Event = CreateV1Event();

        // Act
        var v3Event = _upcaster.Upcast(v1Event);

        // Assert
        v3Event.DateOfBirth.Should().BeNull();  // From v1→v2
        v3Event.Channel.Should().Be(RegistrationChannel.MobileApp);  // From v2→v3
    }

    // Helper methods
    private Event CreateV1Event(string verifiedName = "Test Player")
    {
        return new Event
        {
            SchemaVersion = 1,
            EventType = "PlayerRegistered",
            EventData = BsonDocument.Parse($@"{{
                ""playerId"": ""player-123"",
                ""verifiedName"": ""{verifiedName}"",
                ""addressHash"": ""sha256:abc""
            }}")
        };
    }

    private PlayerRegisteredV2 CreateV2Event()
    {
        return new PlayerRegisteredV2(
            PlayerId: "player-123",
            VerifiedName: "Test Player",
            DateOfBirth: new DateTime(1990, 1, 1),
            AddressHash: "sha256:abc"
        );
    }
}
```

### Integration Tests: Event Replay

**Test Aggregate Rebuilding with Mixed Event Versions**:

```csharp
using Xunit;
using FluentAssertions;

public class PlayerAggregateIntegrationTests
{
    [Fact]
    public async Task Replay_ShouldRebuildFromMixedEventVersions()
    {
        // Arrange: Simulate event store with v1, v2, v3 events
        var events = new List<Event>
        {
            CreateV1Event(eventType: "PlayerRegistered"),     // v1
            CreateV2Event(eventType: "LimitIncreased"),        // v2
            CreateV3Event(eventType: "TransactionAuthorized"), // v3
            CreateV2Event(eventType: "SelfExclusionActivated") // v2
        };

        // Act: Replay events to rebuild aggregate
        var player = new PlayerAggregate();
        foreach (var @event in events)
        {
            player.Apply(@event);  // Upcasters applied internally
        }

        // Assert: Verify aggregate state is correct
        player.PlayerId.Should().NotBeNullOrEmpty();
        player.DateOfBirth.Should().BeNull();  // v1 event had no dateOfBirth
        player.SelfExclusion.Active.Should().BeTrue();
    }

    [Fact]
    public async Task Replay_ShouldHandleAllVersionsWithoutErrors()
    {
        // Arrange
        var events = GenerateMixedVersionEvents(count: 100);

        // Act
        var player = new PlayerAggregate();
        Action act = () =>
        {
            foreach (var @event in events)
            {
                player.Apply(@event);
            }
        };

        // Assert: Should not throw any exceptions
        act.Should().NotThrow();
    }

    private List<Event> GenerateMixedVersionEvents(int count)
    {
        var events = new List<Event>();
        var random = new Random(42);

        for (int i = 0; i < count; i++)
        {
            var version = random.Next(1, 4);  // Random version 1-3
            events.Add(version switch
            {
                1 => CreateV1Event(),
                2 => CreateV2Event(),
                _ => CreateV3Event()
            });
        }

        return events;
    }
}
```

### Load Tests: Upcasting Performance

**Benchmark Event Replay with Upcasting**:

```csharp
using Xunit;
using FluentAssertions;
using System.Diagnostics;

public class UpcastingPerformanceTests
{
    [Fact]
    public void Replay_ShouldProcess10000EventsInUnder1Second()
    {
        // Arrange: Mix of v1, v2, v3 events
        var events = GenerateEvents(10_000);

        // Act
        var stopwatch = Stopwatch.StartNew();

        var player = new PlayerAggregate();
        foreach (var @event in events)
        {
            player.Apply(@event);
        }

        stopwatch.Stop();

        // Assert
        stopwatch.ElapsedMilliseconds.Should().BeLessThan(1000,
            "replaying 10,000 events should take less than 1 second");
    }

    [Theory]
    [InlineData(1_000)]
    [InlineData(5_000)]
    [InlineData(10_000)]
    public void Replay_ShouldScaleLinearlyWithEventCount(int eventCount)
    {
        // Arrange
        var events = GenerateEvents(eventCount);

        // Act
        var stopwatch = Stopwatch.StartNew();

        var player = new PlayerAggregate();
        foreach (var @event in events)
        {
            player.Apply(@event);
        }

        stopwatch.Stop();

        // Assert: < 0.1ms per event
        var avgTimePerEvent = (double)stopwatch.ElapsedMilliseconds / eventCount;
        avgTimePerEvent.Should().BeLessThan(0.1,
            $"average time per event should be < 0.1ms (was {avgTimePerEvent:F3}ms)");
    }

    private List<Event> GenerateEvents(int count)
    {
        var events = new List<Event>();
        var random = new Random(42);

        for (int i = 0; i < count; i++)
        {
            var version = random.Next(1, 4);  // Mix of v1, v2, v3
            events.Add(version switch
            {
                1 => CreateV1Event(),
                2 => CreateV2Event(),
                _ => CreateV3Event()
            });
        }

        return events;
    }
}
```

---

## Migration Procedures

### When to Migrate Events?

**General Rule**: **Never** migrate events in the event store. Let upcasters handle old versions.

**Exception**: If upcaster performance becomes unacceptable (e.g., 10+ schema versions, slow replay).

### Emergency Migration: Event Store Compaction

**Scenario**: After 5 years, event store has 10 schema versions. Replaying aggregates slow (need to upcast 10 times per event).

**Solution**: **One-time migration** to rewrite events in latest schema (DANGEROUS, last resort).

**Procedure**:

**Step 1: Backup Event Store**

```bash
mongodump --db=event_store --out=/backups/pre-migration-$(date +%Y%m%d)
```

**Step 2: Test Migration Script on Copy**

```csharp
// MigrationScript.cs
using MongoDB.Driver;
using MongoDB.Bson;
using System;
using System.Threading.Tasks;

public class EventMigrationScript
{
    public async Task MigrateEventsAsync()
    {
        var client = new MongoClient("mongodb://localhost:27017");
        var db = client.GetDatabase("event_store_COPY");  // Test on copy first

        var events = db.GetCollection<BsonDocument>("events");

        var migratedCount = 0;

        // Find old versions
        var filter = Builders<BsonDocument>.Filter.Lt("schemaVersion", 10);
        var cursor = await events.FindAsync(filter);

        await foreach (var @event in cursor.ToAsyncEnumerable())
        {
            // Apply upcaster
            var upcastedEvent = UpcastToLatestVersion(@event);

            // Replace event (DANGER: modifies immutable data)
            var replaceFilter = Builders<BsonDocument>.Filter.Eq("_id", @event["_id"]);
            await events.ReplaceOneAsync(replaceFilter, upcastedEvent);

            migratedCount++;

            if (migratedCount % 10_000 == 0)
            {
                Console.WriteLine($"Migrated {migratedCount} events...");
            }
        }

        Console.WriteLine($"Migration complete: {migratedCount} events upgraded");
    }

    private BsonDocument UpcastToLatestVersion(BsonDocument @event)
    {
        // Apply all upcasting logic to bring event to latest version
        var currentVersion = @event["schemaVersion"].AsInt32;
        var eventType = @event["eventType"].AsString;

        // Chain upcasters based on event type
        return eventType switch
        {
            "PlayerRegistered" => UpcastPlayerRegisteredToLatest(@event),
            "TransactionAuthorized" => UpcastTransactionAuthorizedToLatest(@event),
            _ => @event  // Unknown event type, return as-is
        };
    }

    private BsonDocument UpcastPlayerRegisteredToLatest(BsonDocument @event)
    {
        // Implementation of upcasting logic
        // This would chain all version-specific upcasters
        throw new NotImplementedException("Implement based on specific upcasting rules");
    }

    private BsonDocument UpcastTransactionAuthorizedToLatest(BsonDocument @event)
    {
        // Implementation of upcasting logic
        throw new NotImplementedException("Implement based on specific upcasting rules");
    }
}

// Program.cs
public class Program
{
    public static async Task Main(string[] args)
    {
        try
        {
            var migration = new EventMigrationScript();
            await migration.MigrateEventsAsync();
        }
        catch (Exception ex)
        {
            Console.Error.WriteLine($"Migration failed: {ex.Message}");
            Environment.Exit(1);
        }
    }
}
```

**Step 3: Validate Migration on Copy**

```bash
# Run validation script
dotnet run --project ValidationScript.csproj

# Check:
# - All events now schemaVersion: 10
# - Event counts match (no data loss)
# - Replay aggregates (state unchanged)
```

**Step 4: Schedule Downtime**

- Announce 4-hour maintenance window (02:00 - 06:00 AM)
- Block all writes to event store
- Take final backup

**Step 5: Run Migration on Production**

```bash
dotnet run --project MigrationScript.csproj -- --db event_store --confirm
```

**Step 6: Validate Production**

```bash
dotnet run --project ValidationScript.csproj -- --db event_store
```

**Step 7: Resume Service**

- Re-enable writes
- Monitor for errors (replay failures, aggregate state mismatches)
- Rollback if issues (restore from backup)

**Warning**: This is a **last resort**. Prefer adding more efficient upcasters.

---

## Governance & Approval

### Schema Change Approval Process

**Level 1: Minor Change (Add Optional Field)**

- **Approver**: Technical Lead
- **Review**: Code review by 2 senior developers
- **Timeline**: 1-2 days
- **Example**: Add `registrationChannel` field

**Level 2: Major Change (Remove Field, Change Type)**

- **Approver**: Architecture Review Board
- **Review**: Design document, impact analysis, migration plan
- **Timeline**: 1-2 weeks
- **Example**: Remove `nickname` field, change `dateOfBirth` type

**Level 3: Breaking Change (Split Event, Merge Events)**

- **Approver**: CTO + Architecture Board
- **Review**: RFC (Request for Comments), stakeholder sign-off
- **Timeline**: 1 month
- **Example**: Split `PlayerRegistered` into two events

### Change Request Template

**Title**: [Event Name] Schema Change: [Brief Description]

**Requestor**: [Name]
**Date**: [YYYY-MM-DD]
**Priority**: [Low / Medium / High / Critical]

**Current Schema Version**: v[X]
**Proposed Schema Version**: v[X+1]

**Reason for Change**:
- Business requirement: [e.g., Need to track registration channel for analytics]
- Technical debt: [e.g., Field no longer used, causes confusion]

**Proposed Changes**:
- Add field: `registrationChannel` (String, enum: mobile_app, web, kiosk)
- Default value for old events: `'mobile_app'`

**Impact Analysis**:
- Affected aggregates: Player
- Affected event handlers: PlayerRegistered handler, Analytics projector
- Backward compatibility: Yes (additive change, default provided)
- Performance impact: None (no additional lookups)

**Migration Plan**:
- Upcaster: Add field with default value
- Testing: Unit tests for upcaster, integration tests for event replay
- Rollout: Deploy upcaster first, then event emitters (1-week gap)

**Risks**:
- Low: Additive change, no data loss

**Approvals**:
- [ ] Technical Lead: [Name]
- [ ] Senior Developer 1: [Name]
- [ ] Senior Developer 2: [Name]

---

## Tools & Utilities

### Event Schema Validator

**Tool**: JSON Schema validation before event append

```csharp
using System.ComponentModel.DataAnnotations;
using MongoDB.Driver;
using MongoDB.Bson;

// Define validator using Data Annotations
public record PlayerRegisteredV3Validator
{
    [Required]
    [RegularExpression(@"^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$")]
    public required string PlayerId { get; init; }

    [Required]
    [MinLength(1)]
    public required string VerifiedName { get; init; }

    public DateTime? DateOfBirth { get; init; }

    [Required]
    [RegularExpression(@"^sha256:")]
    public required string AddressHash { get; init; }

    [Required]
    [EnumDataType(typeof(RegistrationChannel))]
    public required RegistrationChannel RegistrationChannel { get; init; }
}

// Event store with validation
public class EventStore
{
    private readonly IMongoCollection<BsonDocument> _events;

    public EventStore(IMongoDatabase database)
    {
        _events = database.GetCollection<BsonDocument>("events");
    }

    public async Task AppendEventAsync(BsonDocument @event)
    {
        // Validate event schema before appending
        ValidateEventSchema(@event);

        await _events.InsertOneAsync(@event);
    }

    private void ValidateEventSchema(BsonDocument @event)
    {
        var eventType = @event["eventType"].AsString;
        var eventData = @event["eventData"].AsBsonDocument;

        // Validate based on event type and version
        if (eventType == "PlayerRegistered" && @event["schemaVersion"].AsInt32 == 3)
        {
            var validator = BsonSerializer.Deserialize<PlayerRegisteredV3Validator>(eventData);

            var validationContext = new ValidationContext(validator);
            var validationResults = new List<ValidationResult>();

            if (!Validator.TryValidateObject(validator, validationContext, validationResults, true))
            {
                var errors = string.Join(", ", validationResults.Select(r => r.ErrorMessage));
                throw new ValidationException($"Invalid event schema: {errors}");
            }
        }
    }
}
```

### Event Catalog Generator

**Tool**: Auto-generate Event Catalog from C# source using Reflection

```csharp
using System.Reflection;
using System.Text;

public class EventCatalogGenerator
{
    public string GenerateMarkdownCatalog()
    {
        var sb = new StringBuilder();
        sb.AppendLine("# Event Catalog");
        sb.AppendLine();

        // Find all event record types
        var eventTypes = Assembly.GetExecutingAssembly()
            .GetTypes()
            .Where(t => t.Name.StartsWith("PlayerRegistered") && t.IsClass)
            .OrderBy(t => t.Name);

        foreach (var eventType in eventTypes)
        {
            sb.AppendLine($"## {eventType.Name}");
            sb.AppendLine();

            var properties = eventType.GetProperties();
            sb.AppendLine("**Fields**:");
            foreach (var prop in properties)
            {
                var nullable = Nullable.GetUnderlyingType(prop.PropertyType) != null ? "?" : "";
                var typeName = Nullable.GetUnderlyingType(prop.PropertyType)?.Name ?? prop.PropertyType.Name;
                sb.AppendLine($"- `{prop.Name}`: {typeName}{nullable}");
            }
            sb.AppendLine();
        }

        return sb.ToString();
    }
}

// Usage
var generator = new EventCatalogGenerator();
var catalog = generator.GenerateMarkdownCatalog();
await File.WriteAllTextAsync("docs/Event-Catalog.md", catalog);
```

### Upcaster Test Generator

**Tool**: Generate boilerplate tests for upcasters using T4 templates or Source Generators

```csharp
// UpcasterTestGenerator.cs
using System.Text;

public class UpcasterTestGenerator
{
    public string GenerateTestClass(string eventName, int fromVersion, int toVersion)
    {
        var sb = new StringBuilder();

        sb.AppendLine("// Auto-generated by UpcasterTestGenerator");
        sb.AppendLine("using Xunit;");
        sb.AppendLine("using FluentAssertions;");
        sb.AppendLine();
        sb.AppendLine($"public class {eventName}UpcasterTests");
        sb.AppendLine("{");
        sb.AppendLine($"    private readonly {eventName}Upcaster _upcaster = new();");
        sb.AppendLine();

        for (int v = fromVersion; v < toVersion; v++)
        {
            sb.AppendLine($"    // v{v} → v{v + 1} Tests");
            sb.AppendLine("    [Fact]");
            sb.AppendLine($"    public void UpcastV{v}ToV{v + 1}_ShouldAddMissingFields()");
            sb.AppendLine("    {");
            sb.AppendLine("        // TODO: Implement test");
            sb.AppendLine($"        // Arrange: Create v{v} event");
            sb.AppendLine($"        // Act: Upcast to v{v + 1}");
            sb.AppendLine("        // Assert: Verify new fields");
            sb.AppendLine("    }");
            sb.AppendLine();
        }

        sb.AppendLine("}");

        return sb.ToString();
    }
}

// Usage
var generator = new UpcasterTestGenerator();
var testCode = generator.GenerateTestClass("PlayerRegistered", fromVersion: 1, toVersion: 3);
await File.WriteAllTextAsync("tests/PlayerRegisteredUpcasterTests.cs", testCode);
```

Generated test:

```csharp
// Auto-generated by UpcasterTestGenerator
using Xunit;
using FluentAssertions;

public class PlayerRegisteredUpcasterTests
{
    private readonly PlayerRegisteredUpcaster _upcaster = new();

    // v1 → v2 Tests
    [Fact]
    public void UpcastV1ToV2_ShouldAddMissingFields()
    {
        // TODO: Implement test
        // Arrange: Create v1 event
        // Act: Upcast to v2
        // Assert: Verify new fields
    }

    // v2 → v3 Tests
    [Fact]
    public void UpcastV2ToV3_ShouldAddMissingFields()
    {
        // TODO: Implement test
        // Arrange: Create v2 event
        // Act: Upcast to v3
        // Assert: Verify new fields
    }
}
```

---

## Appendix: Common Pitfalls

### Pitfall 1: Forgetting to Upcast in All Event Handlers

**Problem**: Upcaster implemented, but forgot to call in one event handler.

**Symptom**: Aggregate state incorrect when specific old event is replayed.

**Solution**: Centralize upcasting in `Apply()` method (before dispatching to handler).

```csharp
public class PlayerAggregate
{
    private readonly EventUpcaster _upcaster = new();

    public void Apply(Event @event)
    {
        // ALWAYS upcast before handling
        var upcastedEvent = _upcaster.Upcast(@event);

        // Dispatch to specific handler
        switch (upcastedEvent.EventType)
        {
            case "PlayerRegistered":
                HandlePlayerRegistered(upcastedEvent);
                break;
            case "LimitIncreased":
                HandleLimitIncreased(upcastedEvent);
                break;
            // ...
        }
    }

    private void HandlePlayerRegistered(Event @event)
    {
        var data = @event.EventData.ToObject<PlayerRegisteredV3>();
        // Handle the event...
    }

    private void HandleLimitIncreased(Event @event)
    {
        var data = @event.EventData.ToObject<LimitIncreasedV2>();
        // Handle the event...
    }
}
```

### Pitfall 2: Breaking Change Without Upcaster

**Problem**: Renamed field in event schema, but no upcaster written.

**Symptom**: Old events fail to load, aggregate cannot be rebuilt.

**Solution**: **Always** write upcaster when changing schema (even for "simple" renames).

### Pitfall 3: Assuming Latest Schema Version

**Problem**: Code assumes all events are in latest schema (no upcasting).

**Symptom**: Production incidents when old events are replayed (e.g., during backup restore).

**Solution**: **Never** assume schema version. Always upcast.

### Pitfall 4: Upcaster Performance Issues

**Problem**: Upcaster does expensive operation (e.g., database lookup) for every event.

**Symptom**: Aggregate loading slow (10+ seconds), timeout errors.

**Solution**: Optimize upcaster (cache lookups, avoid I/O, use default values).

**Bad**:
```csharp
public class PlayerRegisteredUpcaster
{
    private readonly IConsentIdService _consentIdService;

    public async Task<PlayerRegisteredV2> UpcastAsync(Event @event)
    {
        if (@event.SchemaVersion == 1)
        {
            var v1Data = @event.EventData.ToObject<PlayerRegisteredV1>();

            // BAD: Database lookup for every event (blocks event replay)
            var dateOfBirth = await _consentIdService.FetchDateOfBirthAsync(
                v1Data.ConsentIdSubjectId);

            return new PlayerRegisteredV2(
                PlayerId: v1Data.PlayerId,
                VerifiedName: v1Data.VerifiedName,
                DateOfBirth: dateOfBirth,
                AddressHash: v1Data.AddressHash
            );
        }

        return @event.EventData.ToObject<PlayerRegisteredV2>();
    }
}
```

**Good**:
```csharp
public class PlayerRegisteredUpcaster
{
    public PlayerRegisteredV2 Upcast(Event @event)
    {
        if (@event.SchemaVersion == 1)
        {
            var v1Data = @event.EventData.ToObject<PlayerRegisteredV1>();

            // GOOD: Use default, fetch later if needed
            return new PlayerRegisteredV2(
                PlayerId: v1Data.PlayerId,
                VerifiedName: v1Data.VerifiedName,
                DateOfBirth: null,  // Default, can be fetched lazily when needed
                AddressHash: v1Data.AddressHash
            );
        }

        return @event.EventData.ToObject<PlayerRegisteredV2>();
    }
}
```

---

## Document Control

**Version**: 1.1
**Author**: е-Играч Architecture Team
**Last Updated**: 2025-10-26
**Status**: Ready for Implementation
**Tech Stack**: C# .NET 9, MongoDB C# Driver, MassTransit

**Related Documents**:
- [MongoDB Schema Design](./MongoDB-Schema-Design.md)
- [Event Catalog](./Event-Catalog.md) (to be created)
- [CQRS Implementation Guide](./CQRS-Implementation-Guide.md) (to be created)

**Next Steps**:
1. Review with backend development team
2. Create Event Catalog (Appendix to this playbook)
3. Implement upcaster framework (reusable utilities)
4. Write tests for existing events (v1 upcasters)

---

**END OF DOCUMENT**
