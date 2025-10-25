# Event Versioning Playbook: е-Играч Event Schema Evolution

**Document Version**: 1.0
**Date**: 2025-10-25
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

**Upcaster Code** (TypeScript):

```typescript
interface PlayerRegisteredV1 {
  playerId: string;
  verifiedName: string;
  addressHash: string;
}

interface PlayerRegisteredV2 {
  playerId: string;
  verifiedName: string;
  dateOfBirth: Date | null;  // Nullable for upcasted v1 events
  addressHash: string;
}

function upcastPlayerRegistered(event: Event): PlayerRegisteredV2 {
  if (event.schemaVersion === 1) {
    const v1Data = event.eventData as PlayerRegisteredV1;

    // Upcast: add missing field with default
    return {
      ...v1Data,
      dateOfBirth: null,  // Default: null (fetch from ConsentID later)
      schemaVersion: 2  // Mark as upcasted
    };
  }

  // Already v2, return as-is
  return event.eventData as PlayerRegisteredV2;
}
```

**Usage in Event Handler**:

```typescript
class PlayerAggregate {
  apply(event: Event): void {
    switch (event.eventType) {
      case 'PlayerRegistered':
        const upcastedEvent = upcastPlayerRegistered(event);
        this.handlePlayerRegistered(upcastedEvent);
        break;
      // ...
    }
  }

  private handlePlayerRegistered(data: PlayerRegisteredV2): void {
    this.playerId = data.playerId;
    this.name = data.verifiedName;
    this.dateOfBirth = data.dateOfBirth;  // May be null for old events
    this.addressHash = data.addressHash;
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

```typescript
function upcastPlayerRegistered(event: Event): PlayerRegisteredV2 {
  if (event.schemaVersion === 1) {
    const v1Data = event.eventData as PlayerRegisteredV1;

    return {
      playerId: v1Data.playerId,
      verifiedName: v1Data.name,  // Rename: name → verifiedName
      dateOfBirth: null,
      addressHash: v1Data.addressHash,
      schemaVersion: 2
    };
  }

  return event.eventData as PlayerRegisteredV2;
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

```typescript
function upcastPlayerRegistered(event: Event): PlayerRegisteredV3 {
  // Handle v1 → v2 (from previous upcaster)
  let data = upcastToV2(event);

  // Handle v2 → v3
  if (event.schemaVersion === 2) {
    const v2Data = data as PlayerRegisteredV2;

    return {
      ...v2Data,
      dateOfBirth: v2Data.dateOfBirth
        ? new Date(v2Data.dateOfBirth)  // Parse string to Date
        : null,
      schemaVersion: 3
    };
  }

  // Already v3
  return data as PlayerRegisteredV3;
}
```

**Chaining Upcasters**:

```typescript
function upcastPlayerRegistered(event: Event): PlayerRegisteredV3 {
  let currentVersion = event.schemaVersion;
  let data = event.eventData;

  // Chain: v1 → v2 → v3
  if (currentVersion === 1) {
    data = upcastV1toV2(data);
    currentVersion = 2;
  }

  if (currentVersion === 2) {
    data = upcastV2toV3(data);
    currentVersion = 3;
  }

  return data;
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

```typescript
function upcastPlayerRegistered(event: Event): Event[] {
  if (event.schemaVersion === 1) {
    const v1Data = event.eventData as PlayerRegisteredV1;

    // Return TWO events to replace one v1 event
    return [
      {
        eventType: "PlayerRegistered",
        schemaVersion: 2,
        eventData: {
          playerId: v1Data.playerId,
          verifiedName: v1Data.verifiedName
          // monthlyLimit omitted
        }
      },
      {
        eventType: "LimitSet",
        schemaVersion: 1,
        eventData: {
          playerId: v1Data.playerId,
          monthlyLimit: v1Data.monthlyLimit
        }
      }
    ];
  }

  // v2: already split, return as-is
  return [event];
}
```

**Warning**: Returning multiple events from upcaster is complex. Prefer adding new fields instead.

---

## Schema Evolution Workflows

### Workflow: Adding a New Field

**Scenario**: Add `registrationChannel` field to track how player registered (mobile app, web, kiosk).

**Step 1: Define New Schema (v2)**

```typescript
interface PlayerRegisteredV2 {
  playerId: string;
  verifiedName: string;
  dateOfBirth: Date | null;
  addressHash: string;
  registrationChannel: 'mobile_app' | 'web' | 'kiosk';  // NEW FIELD
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

```typescript
function upcastPlayerRegistered(event: Event): PlayerRegisteredV3 {
  // Chain previous upcasters
  let data = upcastToV2(event);

  // v2 → v3: Add registrationChannel
  if (event.schemaVersion < 3) {
    return {
      ...data,
      registrationChannel: 'mobile_app',  // Default for old events
      schemaVersion: 3
    };
  }

  return data as PlayerRegisteredV3;
}
```

**Step 4: Update Event Emitters**

```typescript
// New code: emit v3 events
function registerPlayer(command: RegisterPlayerCommand): void {
  const event = {
    eventType: 'PlayerRegistered',
    schemaVersion: 3,  // New version
    eventData: {
      playerId: generateUuid(),
      verifiedName: command.name,
      dateOfBirth: command.dateOfBirth,
      addressHash: hashAddress(command.address),
      registrationChannel: command.source  // NEW: capture channel
    }
  };

  eventStore.append(event);
}
```

**Step 5: Test**

```typescript
describe('PlayerRegistered upcasting', () => {
  it('should upcast v1 to v3', () => {
    const v1Event = {
      schemaVersion: 1,
      eventData: {
        playerId: 'player-123',
        verifiedName: 'Petar',
        addressHash: 'sha256:abc'
      }
    };

    const upcastedEvent = upcastPlayerRegistered(v1Event);

    expect(upcastedEvent.schemaVersion).toBe(3);
    expect(upcastedEvent.eventData.registrationChannel).toBe('mobile_app');
    expect(upcastedEvent.eventData.dateOfBirth).toBeNull();
  });

  it('should handle v3 events unchanged', () => {
    const v3Event = {
      schemaVersion: 3,
      eventData: {
        playerId: 'player-456',
        verifiedName: 'Ana',
        dateOfBirth: new Date('1995-05-20'),
        addressHash: 'sha256:def',
        registrationChannel: 'kiosk'
      }
    };

    const upcastedEvent = upcastPlayerRegistered(v3Event);

    expect(upcastedEvent).toEqual(v3Event);  // Unchanged
  });
});
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

In v2, add `@deprecated` comment:

```typescript
interface PlayerRegisteredV2 {
  playerId: string;
  verifiedName: string;
  nickname?: string;  // @deprecated: Will be removed in v3, use verifiedName
}
```

**Step 2: Wait 6-12 Months**

- Ensure no code depends on `nickname` field
- Monitor analytics: is field ever queried?
- Communicate deprecation to stakeholders

**Step 3: Remove Field (v3)**

```typescript
interface PlayerRegisteredV3 {
  playerId: string;
  verifiedName: string;
  // nickname removed
}
```

**Step 4: Update Upcaster**

```typescript
function upcastPlayerRegistered(event: Event): PlayerRegisteredV3 {
  if (event.schemaVersion < 3) {
    const oldData = event.eventData as PlayerRegisteredV2;

    // Strip deprecated field
    return {
      playerId: oldData.playerId,
      verifiedName: oldData.verifiedName,
      // nickname omitted (removed)
      schemaVersion: 3
    };
  }

  return event.eventData as PlayerRegisteredV3;
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

```typescript
describe('PlayerRegistered upcasting', () => {
  describe('v1 → v2', () => {
    it('should add dateOfBirth with null default', () => {
      const v1Event = createV1Event();
      const v2Event = upcastV1toV2(v1Event);

      expect(v2Event.dateOfBirth).toBeNull();
      expect(v2Event.schemaVersion).toBe(2);
    });

    it('should preserve existing fields', () => {
      const v1Event = createV1Event({ verifiedName: 'Petar' });
      const v2Event = upcastV1toV2(v1Event);

      expect(v2Event.verifiedName).toBe('Petar');
    });
  });

  describe('v2 → v3', () => {
    it('should add registrationChannel with default', () => {
      const v2Event = createV2Event();
      const v3Event = upcastV2toV3(v2Event);

      expect(v3Event.registrationChannel).toBe('mobile_app');
    });
  });

  describe('v1 → v3 (chained)', () => {
    it('should apply all upcasters in sequence', () => {
      const v1Event = createV1Event();
      const v3Event = upcastPlayerRegistered(v1Event);

      expect(v3Event.schemaVersion).toBe(3);
      expect(v3Event.dateOfBirth).toBeNull();  // From v1→v2
      expect(v3Event.registrationChannel).toBe('mobile_app');  // From v2→v3
    });
  });
});
```

### Integration Tests: Event Replay

**Test Aggregate Rebuilding with Mixed Event Versions**:

```typescript
describe('Player aggregate event replay', () => {
  it('should rebuild from mixed event versions', async () => {
    // Simulate event store with v1, v2, v3 events
    const events = [
      createV1Event({ eventType: 'PlayerRegistered' }),  // v1
      createV2Event({ eventType: 'LimitIncreased' }),    // v2
      createV3Event({ eventType: 'TransactionAuthorized' }),  // v3
      createV2Event({ eventType: 'SelfExclusionActivated' })  // v2
    ];

    // Replay events to rebuild aggregate
    const player = new PlayerAggregate();
    for (const event of events) {
      player.apply(event);  // Upcasters applied internally
    }

    // Verify aggregate state is correct
    expect(player.playerId).toBeDefined();
    expect(player.dateOfBirth).toBeNull();  // v1 event had no dateOfBirth
    expect(player.selfExclusion.active).toBe(true);
  });
});
```

### Load Tests: Upcasting Performance

**Benchmark Event Replay with Upcasting**:

```typescript
describe('Upcasting performance', () => {
  it('should replay 10,000 events in < 1 second', async () => {
    const events = generateEvents(10_000);  // Mix of v1, v2, v3

    const startTime = Date.now();

    const player = new PlayerAggregate();
    for (const event of events) {
      player.apply(event);
    }

    const duration = Date.now() - startTime;

    expect(duration).toBeLessThan(1000);  // < 1 second
  });
});
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

```javascript
// migration-script.js
const { MongoClient } = require('mongodb');

async function migrateEvents() {
  const client = await MongoClient.connect('mongodb://localhost:27017');
  const db = client.db('event_store_COPY');  // Test on copy first

  const events = db.collection('events');

  let migratedCount = 0;

  const cursor = events.find({ schemaVersion: { $lt: 10 } });  // Old versions

  for await (const event of cursor) {
    // Apply upcaster
    const upcastedEvent = upcastToLatestVersion(event);

    // Replace event (DANGER: modifies immutable data)
    await events.replaceOne(
      { _id: event._id },
      upcastedEvent
    );

    migratedCount++;

    if (migratedCount % 10_000 === 0) {
      console.log(`Migrated ${migratedCount} events...`);
    }
  }

  console.log(`Migration complete: ${migratedCount} events upgraded`);

  await client.close();
}

migrateEvents().catch(console.error);
```

**Step 3: Validate Migration on Copy**

```bash
# Run validation script
node validate-migration.js

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
node migration-script.js --db=event_store --confirm
```

**Step 6: Validate Production**

```bash
node validate-migration.js --db=event_store
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

```typescript
import Ajv from 'ajv';

const ajv = new Ajv();

// Define schema for PlayerRegistered v3
const playerRegisteredV3Schema = {
  type: 'object',
  properties: {
    playerId: { type: 'string', format: 'uuid' },
    verifiedName: { type: 'string', minLength: 1 },
    dateOfBirth: { type: ['string', 'null'], format: 'date' },
    addressHash: { type: 'string', pattern: '^sha256:' },
    registrationChannel: { type: 'string', enum: ['mobile_app', 'web', 'kiosk'] }
  },
  required: ['playerId', 'verifiedName', 'addressHash', 'registrationChannel'],
  additionalProperties: false
};

const validate = ajv.compile(playerRegisteredV3Schema);

// Validate before appending event
function appendEvent(event: Event): void {
  if (!validate(event.eventData)) {
    throw new Error(`Invalid event schema: ${ajv.errorsText(validate.errors)}`);
  }

  eventStore.append(event);
}
```

### Event Catalog Generator

**Tool**: Auto-generate Event Catalog from TypeScript interfaces

```bash
# Extract event schemas from code
npx ts-json-schema-generator \
  --path 'src/events/*.ts' \
  --type 'PlayerRegistered*' \
  --out event-catalog.json

# Generate Markdown documentation
node scripts/generate-event-catalog.js
```

### Upcaster Test Generator

**Tool**: Generate boilerplate tests for upcasters

```bash
# Generate test file
node scripts/generate-upcaster-tests.js \
  --event PlayerRegistered \
  --fromVersion 1 \
  --toVersion 3 \
  --out src/__tests__/upcasters/player-registered.test.ts
```

Generated test:

```typescript
// Auto-generated by upcaster-test-generator
describe('PlayerRegistered upcasting', () => {
  describe('v1 → v2', () => {
    it('should add missing fields', () => {
      // TODO: Implement test
    });
  });

  describe('v2 → v3', () => {
    it('should transform field types', () => {
      // TODO: Implement test
    });
  });
});
```

---

## Appendix: Common Pitfalls

### Pitfall 1: Forgetting to Upcast in All Event Handlers

**Problem**: Upcaster implemented, but forgot to call in one event handler.

**Symptom**: Aggregate state incorrect when specific old event is replayed.

**Solution**: Centralize upcasting in `apply()` method (before dispatching to handler).

```typescript
class PlayerAggregate {
  apply(event: Event): void {
    // ALWAYS upcast before handling
    const upcastedEvent = upcastEvent(event);

    // Dispatch to specific handler
    switch (upcastedEvent.eventType) {
      case 'PlayerRegistered':
        this.handlePlayerRegistered(upcastedEvent);
        break;
      // ...
    }
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
```typescript
function upcastPlayerRegistered(event: Event): PlayerRegisteredV2 {
  if (event.schemaVersion === 1) {
    // BAD: Database lookup for every event
    const dateOfBirth = await fetchFromConsentID(event.eventData.consentIdSubjectId);

    return { ...event.eventData, dateOfBirth };
  }
}
```

**Good**:
```typescript
function upcastPlayerRegistered(event: Event): PlayerRegisteredV2 {
  if (event.schemaVersion === 1) {
    // GOOD: Use default, fetch later if needed
    return { ...event.eventData, dateOfBirth: null };
  }
}
```

---

## Document Control

**Version**: 1.0
**Author**: е-Играч Architecture Team
**Last Updated**: 2025-10-25
**Status**: Ready for Implementation

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
