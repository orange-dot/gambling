# MongoDB Schema Design: е-Играч Event Store & Read Models

**Document Version**: 1.0
**Date**: 2025-10-25
**Architecture**: Event Sourcing + CQRS
**Deployment**: Single MongoDB cluster with strict sharding discipline

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Database Organization](#database-organization)
3. [Event Store Schema](#event-store-schema)
4. [Read Model Schemas](#read-model-schemas)
5. [Indexes & Performance](#indexes--performance)
6. [Sharding Strategy](#sharding-strategy)
7. [Retention & Archival](#retention--archival)
8. [Backup & Recovery](#backup--recovery)
9. [Security & Access Control](#security--access-control)

---

## Architecture Overview

### Event Sourcing Principles

The е-Играч system uses **Event Sourcing** where:
- All state changes are captured as **immutable events**
- Events are stored in an **append-only event store**
- Current state is derived by **replaying events** (with snapshots for performance)
- **Read models** (CQRS projections) are materialized views optimized for queries

### MongoDB Cluster Organization

```
┌─────────────────────────────────────────────────────────┐
│            Single MongoDB Cluster (Sharded)             │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────────────┐  ┌────────────────────────┐ │
│  │  event_store DB      │  │  read_models DB        │ │
│  │  ─────────────────── │  │  ──────────────────    │ │
│  │  Shard: eventStore   │  │  Shard: readModels     │ │
│  │                      │  │                        │ │
│  │  Collections:        │  │  Collections:          │ │
│  │  • events            │  │  • player_read_model   │ │
│  │  • snapshots         │  │  • transaction_rm      │ │
│  │                      │  │  • operator_accounts   │ │
│  └──────────────────────┘  └────────────────────────┘ │
│                                                         │
│  Isolation: Separate shards prevent workload contention│
└─────────────────────────────────────────────────────────┘
```

### Key Design Decisions

1. **Two Databases, One Cluster**:
   - `event_store` DB: Immutable events (append-only, write-optimized)
   - `read_models` DB: Query projections (read-optimized, frequently updated)

2. **Shard Tags for Isolation**:
   - Event store on dedicated shard(s)
   - Read models on separate shard(s)
   - Prevents I/O contention between writes and reads

3. **Event as Source of Truth**:
   - Read models can be deleted and rebuilt from events
   - Events are NEVER modified, only appended

---

## Database Organization

### Database: `event_store`

**Purpose**: Store all domain events (source of truth)

**Collections**:
- `events` - All domain events (immutable)
- `snapshots` - Aggregate snapshots for performance (ephemeral)

**Characteristics**:
- Write-heavy workload (high throughput appends)
- Sequential writes (events ordered by timestamp)
- Long retention (7 years minimum)
- Critical for compliance and audit

### Database: `read_models`

**Purpose**: Materialized views optimized for queries

**Collections**:
- `player_read_model` - Current player state
- `transaction_read_model` - Transaction history
- `operator_accounts_read_model` - Operator account links
- `limit_history_read_model` - Spending limit changes
- `self_exclusion_read_model` - Self-exclusion records

**Characteristics**:
- Read-heavy workload (queries from mobile app, APIs)
- Frequently updated (by Flink processors)
- Can be rebuilt from events (ephemeral)
- Short retention (operational data only)

---

## Event Store Schema

### Collection: `event_store.events`

**Purpose**: Immutable log of all state changes

**Schema**:

```javascript
{
  _id: ObjectId("507f1f77bcf86cd799439011"),

  // Aggregate identification
  aggregateId: "player-a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  aggregateType: "Player",  // Player | Transaction | Limit | SelfExclusion

  // Event metadata
  eventType: "PlayerRegistered",  // Specific event type
  eventVersion: 1,  // Version of this aggregate (optimistic locking)
  schemaVersion: 1,  // Schema version for event payload

  // Event payload (varies by eventType)
  eventData: {
    playerId: "player-a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    consentIdSubjectId: "consent-xyz123",
    verifiedName: "Petar Petrović",
    dateOfBirth: ISODate("1990-01-15"),
    addressHash: "sha256:abc123...",
    registrationSource: "mobile_app"
  },

  // Event context
  metadata: {
    timestamp: ISODate("2025-10-25T14:30:00Z"),
    userId: "player-a1b2c3d4...",  // Actor who triggered event
    operatorId: null,  // Null for player-initiated events
    ipAddressHash: "sha256:def456...",  // Privacy: hashed IP
    userAgent: "eIgrac-iOS/1.0.0",
    correlationId: "req-trace-uuid",  // For distributed tracing
    causationId: "parent-event-uuid",  // Event that caused this event
    sessionId: "session-uuid"
  },

  // Global ordering
  sequenceNumber: 12345,  // Monotonically increasing global sequence

  // Audit & compliance
  committedAt: ISODate("2025-10-25T14:30:00.123Z"),  // Precise commit time
  streamPosition: 1,  // Position within this aggregate's stream

  // Cryptographic integrity (optional, for high-security environments)
  eventHash: "sha256:ghijk789...",  // Hash of eventData for tamper detection
  previousEventHash: "sha256:lmn012..."  // Chain to previous event (blockchain-style)
}
```

**Field Descriptions**:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `aggregateId` | String (UUID) | Yes | Unique identifier for the aggregate (e.g., playerId) |
| `aggregateType` | String (enum) | Yes | Type of aggregate: Player, Transaction, Limit, SelfExclusion |
| `eventType` | String | Yes | Specific event name (e.g., PlayerRegistered, TransactionAuthorized) |
| `eventVersion` | Integer | Yes | Version of this aggregate (for optimistic locking, prevents concurrent writes) |
| `schemaVersion` | Integer | Yes | Version of the event schema (for evolution/migration) |
| `eventData` | Object | Yes | Event-specific payload (varies by eventType) |
| `metadata` | Object | Yes | Context: timestamp, actor, IP, correlation IDs |
| `sequenceNumber` | Long | Yes | Global ordering across all events |
| `committedAt` | ISODate | Yes | Precise commit timestamp (for audit) |
| `streamPosition` | Integer | Yes | Position within aggregate's event stream |
| `eventHash` | String | Optional | SHA256 hash of eventData (tamper detection) |
| `previousEventHash` | String | Optional | Hash of previous event (blockchain chaining) |

**Indexes**:

```javascript
// Primary key
db.events.createIndex({ _id: 1 }, { unique: true })

// Query by aggregate (most common: load aggregate history)
db.events.createIndex(
  { aggregateId: 1, eventVersion: 1 },
  { unique: true, name: "aggregate_version_unique" }
)

// Query by aggregate type and time range
db.events.createIndex(
  { aggregateType: 1, "metadata.timestamp": -1 },
  { name: "aggregate_type_time" }
)

// Query by event type (for analytics)
db.events.createIndex(
  { eventType: 1, "metadata.timestamp": -1 },
  { name: "event_type_time" }
)

// Query by sequence number (global ordering, event replay)
db.events.createIndex(
  { sequenceNumber: 1 },
  { unique: true, name: "sequence_number_unique" }
)

// Query by correlation ID (distributed tracing)
db.events.createIndex(
  { "metadata.correlationId": 1 },
  { name: "correlation_id" }
)
```

**Constraints**:

- **Immutability**: Events NEVER updated or deleted (append-only)
- **Uniqueness**: `(aggregateId, eventVersion)` must be unique (prevents duplicate events)
- **Ordering**: `sequenceNumber` must be monotonically increasing (global event order)
- **Validation**: All fields validated before insert (JSON schema validation)

---

### Collection: `event_store.snapshots`

**Purpose**: Performance optimization - avoid replaying thousands of events

**Schema**:

```javascript
{
  _id: ObjectId(),

  // Aggregate identification
  aggregateId: "player-a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  aggregateType: "Player",

  // Snapshot data (full aggregate state at this point)
  snapshot: {
    playerId: "player-a1b2c3d4...",
    consentIdSubjectId: "consent-xyz123",
    profile: {
      verifiedName: "Petar Petrović",
      dateOfBirth: ISODate("1990-01-15"),
      addressHash: "sha256:abc123..."
    },
    limits: {
      monthlyLimit: 20000,
      currentMonthSpending: 5000,
      lastLimitChange: ISODate("2025-10-01"),
      coolOffEndDate: null
    },
    selfExclusion: {
      active: false,
      startDate: null,
      endDate: null,
      type: null
    },
    marketingOptOut: false,
    accountStatus: "active"
  },

  // Snapshot metadata
  eventVersion: 100,  // Snapshot taken after event version 100
  createdAt: ISODate("2025-10-25T14:30:00Z"),
  expiresAt: ISODate("2025-11-24T14:30:00Z")  // TTL: 30 days
}
```

**Snapshot Strategy**:

- **When to snapshot**: Every 100 events per aggregate
- **How to rebuild**: Load snapshot + replay events since snapshot version
- **Expiry**: 30-day TTL (snapshots deleted automatically, can rebuild from events)
- **Trade-off**: Storage space vs. replay performance

**Indexes**:

```javascript
// Query by aggregateId (most recent snapshot)
db.snapshots.createIndex(
  { aggregateId: 1, eventVersion: -1 },
  { name: "aggregate_latest_snapshot" }
)

// TTL index (auto-delete old snapshots)
db.snapshots.createIndex(
  { expiresAt: 1 },
  { expireAfterSeconds: 0, name: "snapshot_ttl" }
)
```

---

## Read Model Schemas

### Collection: `read_models.player_read_model`

**Purpose**: Current player state for fast queries (transaction validation, app display)

**Schema**:

```javascript
{
  _id: "player-a1b2c3d4-e5f6-7890-abcd-ef1234567890",  // playerId (natural key)

  // Identity (from PlayerRegistered event)
  consentIdSubjectId: "consent-xyz123",

  profile: {
    verifiedName: "Petar Petrović",
    dateOfBirth: ISODate("1990-01-15"),
    addressHash: "sha256:abc123...",  // Privacy: hashed address
    ageVerified: true,
    registrationDate: ISODate("2025-01-15")
  },

  // Spending limits (from LimitIncreased/LimitDecreased events)
  limits: {
    monthlyLimit: 20000,  // RSD
    generalMaximumLimit: 100000,  // RSD (based on average income)
    personalMaximumLimit: 150000,  // RSD (based on individual income, if verified)
    currentMonthSpending: 5000,  // RSD (updated by Flink aggregator)
    currentMonth: "2025-10",  // YYYY-MM format
    lastLimitChange: ISODate("2025-10-01"),
    coolOffEndDate: null,  // ISODate if in cooling-off period (72 hours for increases)
    pendingLimitIncrease: null  // Amount if pending cooling-off
  },

  // Self-exclusion (from SelfExclusionActivated/SelfExclusionDeactivated events)
  selfExclusion: {
    active: false,
    startDate: null,
    endDate: null,
    type: null,  // 3M | 6M | 12M | 5Y | 10Y | PERMANENT
    medicalOverrideRequested: false,
    medicalOverrideStatus: null  // PENDING | APPROVED | DENIED
  },

  // Marketing preferences (from MarketingOptOutChanged event)
  marketingOptOut: false,

  // Account status
  accountStatus: "active",  // active | suspended | deleted

  // Event sourcing metadata
  lastEventVersion: 42,  // Last event version applied to this read model
  lastUpdated: ISODate("2025-10-25T14:30:00Z"),

  // Audit
  createdAt: ISODate("2025-01-15T10:00:00Z"),
  updatedAt: ISODate("2025-10-25T14:30:00Z")
}
```

**Indexes**:

```javascript
// Primary key
db.player_read_model.createIndex({ _id: 1 }, { unique: true })

// Query by ConsentID (for authentication flow)
db.player_read_model.createIndex(
  { consentIdSubjectId: 1 },
  { unique: true, name: "consent_id_unique" }
)

// Query by spending limit (analytics: find players near limit)
db.player_read_model.createIndex(
  { "limits.currentMonthSpending": 1, "limits.monthlyLimit": 1 },
  { name: "spending_analysis" }
)

// Query by self-exclusion status (filter excluded players)
db.player_read_model.createIndex(
  { "selfExclusion.active": 1, "selfExclusion.endDate": 1 },
  { name: "self_exclusion_status" }
)

// Query by account status
db.player_read_model.createIndex(
  { accountStatus: 1 },
  { name: "account_status" }
)
```

**Update Pattern** (by Flink):

```javascript
// When PlayerRegistered event is consumed
db.player_read_model.insertOne({
  _id: event.eventData.playerId,
  consentIdSubjectId: event.eventData.consentIdSubjectId,
  profile: { ... },
  limits: { monthlyLimit: 20000, currentMonthSpending: 0, ... },
  lastEventVersion: event.eventVersion,
  createdAt: new Date(),
  updatedAt: new Date()
})

// When TransactionAuthorized event is consumed
db.player_read_model.updateOne(
  { _id: event.eventData.playerId, lastEventVersion: { $lt: event.eventVersion } },
  {
    $inc: { "limits.currentMonthSpending": event.eventData.amount },
    $set: {
      lastEventVersion: event.eventVersion,
      updatedAt: new Date()
    }
  }
)
```

---

### Collection: `read_models.transaction_read_model`

**Purpose**: Transaction history for player viewing, analytics, compliance

**Schema**:

```javascript
{
  _id: "transaction-uuid",

  // Transaction identification
  playerId: "player-a1b2c3d4...",
  operatorId: "operator-abc123",
  operatorTransactionId: "op-txn-456",  // Operator's internal ID

  // Transaction details
  amount: 5000,  // RSD
  transactionType: "deposit",  // deposit | bet | win | withdrawal
  status: "completed",  // authorized | completed | rejected | voided

  // Timestamps
  authorizedAt: ISODate("2025-10-25T14:30:00Z"),
  completedAt: ISODate("2025-10-25T14:30:10Z"),

  // Context
  monthYear: "2025-10",  // For monthly aggregation
  quarter: "2025-Q4",  // For quarterly reporting

  // Metadata
  paymentMethod: "credit_card",  // For deposits/withdrawals
  gameType: "slots",  // For bets (slots | sports | poker | roulette | etc.)

  // Event sourcing
  eventId: ObjectId("507f1f77bcf86cd799439011"),  // Link to source event

  // Audit
  createdAt: ISODate("2025-10-25T14:30:00Z"),
  updatedAt: ISODate("2025-10-25T14:30:10Z")
}
```

**Indexes**:

```javascript
// Query by player (most common: show transaction history)
db.transaction_read_model.createIndex(
  { playerId: 1, authorizedAt: -1 },
  { name: "player_transactions" }
)

// Query by player and month (monthly spending calculation)
db.transaction_read_model.createIndex(
  { playerId: 1, monthYear: 1, status: 1 },
  { name: "player_monthly_spending" }
)

// Query by operator and time range (operator reporting)
db.transaction_read_model.createIndex(
  { operatorId: 1, authorizedAt: -1 },
  { name: "operator_transactions" }
)

// Query by status (find pending transactions)
db.transaction_read_model.createIndex(
  { status: 1, authorizedAt: -1 },
  { name: "transaction_status" }
)
```

**TTL Policy**:

```javascript
// Auto-delete transactions older than 7 years (regulatory retention)
db.transaction_read_model.createIndex(
  { createdAt: 1 },
  { expireAfterSeconds: 220752000, name: "transaction_ttl" }  // 7 years in seconds
)
```

---

### Collection: `read_models.operator_accounts_read_model`

**Purpose**: Link players to operator accounts (for registration token validation)

**Schema**:

```javascript
{
  _id: ObjectId(),

  // Link between player and operator
  playerId: "player-a1b2c3d4...",
  operatorId: "operator-abc123",
  operatorAccountId: "op-account-xyz789",  // Operator's internal player ID

  // Registration
  registrationToken: "sha256:token-hash...",  // One-time token used
  tokenUsedAt: ISODate("2025-10-25T14:30:00Z"),

  // Account status
  accountStatus: "active",  // active | suspended | closed

  // Timestamps
  linkedAt: ISODate("2025-10-25T14:30:00Z"),
  lastActivityAt: ISODate("2025-10-25T18:00:00Z"),

  // Event sourcing
  eventId: ObjectId("507f..."),  // Link to OperatorAccountLinked event

  createdAt: ISODate("2025-10-25T14:30:00Z"),
  updatedAt: ISODate("2025-10-25T18:00:00Z")
}
```

**Indexes**:

```javascript
// Query by player (list all operator accounts)
db.operator_accounts_read_model.createIndex(
  { playerId: 1, operatorId: 1 },
  { unique: true, name: "player_operator_unique" }
)

// Query by registration token (validate token during registration)
db.operator_accounts_read_model.createIndex(
  { registrationToken: 1 },
  { unique: true, sparse: true, name: "registration_token" }
)

// Query by operator (operator-specific queries)
db.operator_accounts_read_model.createIndex(
  { operatorId: 1, playerId: 1 },
  { name: "operator_players" }
)
```

---

## Indexes & Performance

### Index Strategy

**General Principles**:
1. **Compound indexes**: Most queries filter by multiple fields (player + date, operator + status)
2. **Sort order**: Indexes must match query sort order (timestamps descending for recent-first)
3. **Selectivity**: High-cardinality fields first (playerId > status)
4. **Covered queries**: Include projected fields in index when possible

### Query Performance Targets

| Query Type | Collection | Target Latency | Index Used |
|------------|------------|----------------|------------|
| Load aggregate events | `events` | < 50ms | `aggregateId_1_eventVersion_1` |
| Validate transaction | `player_read_model` | < 10ms | `_id` (primary key) |
| Show transaction history (last 30 days) | `transaction_read_model` | < 100ms | `playerId_1_authorizedAt_-1` |
| Monthly spending calculation | `transaction_read_model` | < 50ms | `playerId_1_monthYear_1_status_1` |
| Check self-exclusion | `player_read_model` | < 10ms | `_id` (primary key, cached) |

### Index Maintenance

**Monitoring**:
```javascript
// Check index usage statistics
db.events.aggregate([
  { $indexStats: {} }
])

// Find slow queries
db.currentOp({ "secs_running": { $gte: 1 } })
```

**Rebuild indexes** (if fragmented):
```javascript
db.events.reIndex()  // During maintenance window only
```

---

## Sharding Strategy

### Shard Configuration

**Cluster Setup**:
- **Shard 1 (eventStore)**: Dedicated to `event_store` database
- **Shard 2 (readModels)**: Dedicated to `read_models` database
- **Config servers**: 3 replicas (metadata storage)
- **Mongos routers**: 2+ instances (query routing)

**Shard Tags**:

```javascript
// Tag shards
sh.addShardTag("shard0001", "eventStore")
sh.addShardTag("shard0002", "readModels")

// Assign database ranges to shards
sh.addTagRange(
  "event_store.events",
  { _id: MinKey },
  { _id: MaxKey },
  "eventStore"
)

sh.addTagRange(
  "read_models.player_read_model",
  { _id: MinKey },
  { _id: MaxKey },
  "readModels"
)
```

### Shard Keys

**event_store.events**:
```javascript
// Shard by aggregateId (distributes player events across shards)
sh.shardCollection("event_store.events", { aggregateId: 1, eventVersion: 1 })
```

**Rationale**:
- All events for a player on same shard (efficient aggregate loading)
- Even distribution (UUIDs are random)
- Supports targeted queries (fetch events for specific player)

**read_models.player_read_model**:
```javascript
// Shard by _id (playerId)
sh.shardCollection("read_models.player_read_model", { _id: 1 })
```

**Rationale**:
- Even distribution (UUIDs are random)
- Most queries are by playerId (no scatter-gather)

**read_models.transaction_read_model**:
```javascript
// Shard by playerId + authorizedAt (range-based)
sh.shardCollection(
  "read_models.transaction_read_model",
  { playerId: 1, authorizedAt: -1 }
)
```

**Rationale**:
- Queries always filter by playerId (avoid scatter-gather)
- Time-based range queries efficient (contiguous data on same chunk)

### Balancer Configuration

```javascript
// Run balancer during off-peak hours only
sh.setBalancerState(true)
sh.startBalancer()

// Schedule balancing window (02:00 - 06:00)
use config
db.settings.updateOne(
  { _id: "balancer" },
  { $set: { activeWindow: { start: "02:00", stop: "06:00" } } },
  { upsert: true }
)
```

---

## Retention & Archival

### Retention Policies

| Collection | Retention Period | Archival Strategy |
|------------|------------------|-------------------|
| `events` | 7 years (regulatory) | After 2 years: move to cold storage (Azure Blob, Glacier) |
| `snapshots` | 30 days | TTL index, auto-delete |
| `transaction_read_model` | 7 years | TTL index, auto-delete after 7 years |
| `player_read_model` | Active players only | Delete when player closes account (GDPR) |
| `operator_accounts_read_model` | Active links only | Delete when operator link removed |

### Archival Process

**Step 1: Export old events to cold storage**

```bash
# Export events older than 2 years
mongoexport \
  --db=event_store \
  --collection=events \
  --query='{"metadata.timestamp": {"$lt": ISODate("2023-01-01")}}' \
  --out=events_archive_2023.json

# Compress
gzip events_archive_2023.json

# Upload to cold storage
az storage blob upload \
  --account-name eigracstorage \
  --container-name event-archives \
  --file events_archive_2023.json.gz \
  --name events_2023.json.gz
```

**Step 2: Delete archived events from hot storage**

```javascript
// DANGER: Only after verifying archive integrity
db.events.deleteMany({
  "metadata.timestamp": { $lt: ISODate("2023-01-01") }
})
```

**Step 3: Restore from archive (if needed)**

```bash
# Download archive
az storage blob download \
  --account-name eigracstorage \
  --container-name event-archives \
  --name events_2023.json.gz \
  --file events_archive_2023.json.gz

# Decompress
gunzip events_archive_2023.json.gz

# Restore to MongoDB
mongoimport \
  --db=event_store \
  --collection=events \
  --file=events_archive_2023.json
```

---

## Backup & Recovery

### Backup Strategy

**Full Backups**:
- **Frequency**: Daily (02:00 AM)
- **Method**: `mongodump` with `--oplog` for point-in-time consistency
- **Retention**: 7 daily, 4 weekly, 12 monthly backups
- **Storage**: On-premises backup server + offsite cold storage

**Incremental Backups**:
- **Frequency**: Hourly
- **Method**: Replica set oplog tailing
- **Tool**: Percona Backup for MongoDB or MongoDB Ops Manager
- **RPO**: 1 hour (max data loss)

**Continuous Replication**:
- **Method**: Replica set with delayed secondary (4-hour delay)
- **Purpose**: Protection against accidental deletes/updates
- **Secondary can be promoted** if primary data corrupted

### Backup Commands

**Full Backup**:

```bash
# Backup entire cluster
mongodump \
  --host=mongos01.eigrac.local:27017 \
  --out=/backups/eigrac-$(date +%Y%m%d) \
  --oplog

# Compress backup
tar -czf /backups/eigrac-$(date +%Y%m%d).tar.gz /backups/eigrac-$(date +%Y%m%d)

# Copy to offsite storage
rsync -avz /backups/eigrac-$(date +%Y%m%d).tar.gz backup-server:/mnt/backups/
```

**Restore**:

```bash
# Extract backup
tar -xzf eigrac-20251025.tar.gz

# Restore to MongoDB
mongorestore \
  --host=mongos01.eigrac.local:27017 \
  --dir=eigrac-20251025 \
  --oplogReplay
```

### Disaster Recovery

**RTO (Recovery Time Objective)**: 1 hour
**RPO (Recovery Point Objective)**: 5 minutes (continuous replication)

**DR Procedures**:
1. **Primary datacenter failure**: Promote secondary datacenter replica set to primary (manual failover, 15 minutes)
2. **Entire cluster failure**: Restore from latest backup + apply oplog (full restore, 45 minutes)
3. **Data corruption**: Restore from delayed secondary replica (immediate, < 5 minutes)

---

## Security & Access Control

### Authentication

**MongoDB Authentication**:
```javascript
// Create admin user
use admin
db.createUser({
  user: "eigrac-admin",
  pwd: "STRONG_PASSWORD",
  roles: [
    { role: "userAdminAnyDatabase", db: "admin" },
    { role: "dbAdminAnyDatabase", db: "admin" },
    { role: "readWriteAnyDatabase", db: "admin" }
  ]
})

// Create application users
use event_store
db.createUser({
  user: "eigrac-event-writer",
  pwd: "STRONG_PASSWORD",
  roles: [
    { role: "readWrite", db: "event_store" }
  ]
})

use read_models
db.createUser({
  user: "eigrac-projector",
  pwd: "STRONG_PASSWORD",
  roles: [
    { role: "readWrite", db: "read_models" }
  ]
})

use read_models
db.createUser({
  user: "eigrac-query-api",
  pwd: "STRONG_PASSWORD",
  roles: [
    { role: "read", db: "read_models" }
  ]
})
```

### Authorization (RBAC)

**Custom Roles**:

```javascript
// Event writer: can only append events, never update/delete
use admin
db.createRole({
  role: "eventAppendOnly",
  privileges: [
    {
      resource: { db: "event_store", collection: "events" },
      actions: ["insert", "find"]  // NO update, remove
    }
  ],
  roles: []
})

// Projector: can update read models
db.createRole({
  role: "projectorRole",
  privileges: [
    {
      resource: { db: "read_models", collection: "" },  // All collections
      actions: ["insert", "update", "find", "remove"]
    }
  ],
  roles: []
})

// Query API: read-only access
db.createRole({
  role: "queryApiRole",
  privileges: [
    {
      resource: { db: "read_models", collection: "" },
      actions: ["find"]  // Read-only
    }
  ],
  roles: []
})
```

### Encryption

**Encryption at Rest**:

```yaml
# mongod.conf
security:
  enableEncryption: true
  encryptionKeyFile: /etc/mongodb/encryption-key
  kmip:
    serverName: kmip.eigrac.local
    port: 5696
    clientCertificateFile: /etc/mongodb/client-cert.pem
```

**Encryption in Transit**:

```yaml
# mongod.conf (TLS)
net:
  tls:
    mode: requireTLS
    certificateKeyFile: /etc/mongodb/mongodb-cert.pem
    CAFile: /etc/mongodb/ca-cert.pem
```

### Audit Logging

**Enable Auditing**:

```yaml
# mongod.conf
auditLog:
  destination: file
  format: JSON
  path: /var/log/mongodb/audit.json
  filter: '{
    "atype": "authCheck",
    "param.command": { "$in": ["update", "delete", "drop"] }
  }'
```

**What to Audit**:
- All authentication attempts (failed and successful)
- Write operations on `events` collection (should be rare - investigate if many updates/deletes)
- Schema changes (createCollection, dropCollection, createIndex)
- User/role management operations

---

## Appendix: Event Types Reference

### Player Events

| Event Type | Description | Payload Fields |
|------------|-------------|----------------|
| `PlayerRegistered` | Player creates account via ConsentID | playerId, consentIdSubjectId, verifiedName, dateOfBirth, addressHash |
| `PlayerProfileUpdated` | Player updates profile (address change) | playerId, newAddressHash |
| `PlayerAccountSuspended` | Account suspended (fraud, violation) | playerId, reason, suspendedUntil |
| `PlayerAccountReactivated` | Suspended account reactivated | playerId |
| `PlayerAccountDeleted` | GDPR deletion (right to be forgotten) | playerId, deletionReason |

### Transaction Events

| Event Type | Description | Payload Fields |
|------------|-------------|----------------|
| `TransactionAuthorized` | Transaction approved by system | transactionId, playerId, operatorId, amount, transactionType |
| `TransactionCompleted` | Transaction finalized by operator | transactionId, completedAmount, completedAt |
| `TransactionRejected` | Transaction denied (limit exceeded) | transactionId, playerId, rejectionReason |
| `TransactionVoided` | Transaction reversed (error, refund) | transactionId, voidReason |

### Limit Events

| Event Type | Description | Payload Fields |
|------------|-------------|----------------|
| `LimitIncreased` | Player increases monthly limit | playerId, oldLimit, newLimit, effectiveDate (after cooling-off) |
| `LimitDecreased` | Player decreases limit (immediate) | playerId, oldLimit, newLimit |
| `LimitCoolingOffStarted` | 72-hour cooling-off for increase | playerId, pendingLimit, coolingOffEndDate |
| `LimitCoolingOffCancelled` | Player cancels pending increase | playerId, cancelledLimit |

### Self-Exclusion Events

| Event Type | Description | Payload Fields |
|------------|-------------|----------------|
| `SelfExclusionActivated` | Player activates self-exclusion | playerId, exclusionType (3M, 6M, 12M, 5Y, 10Y, PERMANENT), startDate, endDate |
| `SelfExclusionExpired` | Self-exclusion period ended | playerId, exclusionType |
| `MedicalOverrideRequested` | Doctor approval requested | playerId, referralCode, doctorId |
| `MedicalOverrideApproved` | Doctor approves early termination | playerId, doctorId, approvalDate, coolingOffEndDate (7 days) |
| `MedicalOverrideDenied` | Doctor denies request | playerId, doctorId, denialReason |

### Marketing Events

| Event Type | Description | Payload Fields |
|------------|-------------|----------------|
| `MarketingOptOutChanged` | Player opts out of marketing | playerId, optOut (true/false) |

### Operator Events

| Event Type | Description | Payload Fields |
|------------|-------------|----------------|
| `OperatorAccountLinked` | Player links account at operator | playerId, operatorId, operatorAccountId, registrationToken |
| `OperatorAccountUnlinked` | Operator account removed | playerId, operatorId, unlinkReason |

---

## Appendix: MongoDB Configuration

### Recommended `mongod.conf`

```yaml
# Network
net:
  port: 27017
  bindIp: 0.0.0.0
  tls:
    mode: requireTLS
    certificateKeyFile: /etc/mongodb/mongodb-cert.pem
    CAFile: /etc/mongodb/ca-cert.pem

# Storage
storage:
  dbPath: /var/lib/mongodb
  engine: wiredTiger
  wiredTiger:
    engineConfig:
      cacheSizeGB: 64  # 50% of RAM
      journalCompressor: snappy
    collectionConfig:
      blockCompressor: snappy
    indexConfig:
      prefixCompression: true

# Replication
replication:
  replSetName: "rs-eventstore"
  oplogSizeMB: 51200  # 50 GB (large for 7-day retention)

# Sharding (on mongos instances)
sharding:
  clusterRole: shardsvr

# Security
security:
  authorization: enabled
  enableEncryption: true
  encryptionKeyFile: /etc/mongodb/encryption-key

# Operations
operationProfiling:
  mode: slowOp
  slowOpThresholdMs: 100

# Auditing
auditLog:
  destination: file
  format: JSON
  path: /var/log/mongodb/audit.json

# System Log
systemLog:
  destination: file
  path: /var/log/mongodb/mongod.log
  logAppend: true
  logRotate: reopen
```

---

## Document Control

**Version**: 1.0
**Author**: е-Играч Architecture Team
**Last Updated**: 2025-10-25
**Status**: Ready for Implementation

**Next Steps**:
1. Review with MongoDB DBA team
2. Provision hardware for sharded cluster (6 servers minimum: 2 shards × 3 replicas each)
3. Deploy cluster in staging environment
4. Load test with synthetic event data (1M+ events)
5. Tune indexes and sharding based on actual query patterns

---

**END OF DOCUMENT**
