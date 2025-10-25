# Implementation Plan: е-Играч (e-Igrac) Mobile Application System

## Executive Summary

This implementation plan outlines the phased development and deployment of the е-Играч mobile application system - a comprehensive responsible gambling platform for the Republic of Serbia. The system integrates state-run authentication (ConsentID), centralized player registry, mandatory spending limits, and multi-operator transaction tracking to create a unified gambling control ecosystem.

**Project Timeline**: 18-24 months from inception to full deployment
**Estimated User Base**: 50,000-150,000 active users (online), expanding to physical casinos
**Technical Complexity**: High (state system integration, real-time transaction validation, multi-operator coordination)

---

## Architecture Overview

### Event Sourcing & CQRS Architecture

The е-Играч system is built on **Event Sourcing** and **CQRS (Command Query Responsibility Segregation)** principles:

**Core Principles**:
- All state changes stored as immutable events in MongoDB event store
- Current state derived by replaying events (with snapshots for performance)
- Separate read models optimized for queries (updated asynchronously)
- Complete audit trail built into architecture (every state change is an event)
- Temporal queries possible (reconstruct system state at any point in time)

**Technology Stack**:
1. **MongoDB**: Primary and only database
   - Event store (source of truth)
   - Read models (CQRS projections)
   - Snapshots (performance optimization)

2. **Apache Kafka**: Event bus
   - Publishes events to interested consumers
   - Guarantees event ordering per player
   - Enables microservices to react to events independently

3. **Apache Flink**: Stream processing engine
   - Processes events in real-time
   - Updates read models from events
   - Complex event pattern detection (fraud, problem gambling)
   - Real-time analytics and aggregations
   - Windowing for time-based calculations

4. **Azure Kubernetes Service (AKS)**: Execution environment
   - On-premises deployment (no cloud dependencies)
   - Horizontal pod autoscaling for microservices
   - StatefulSets for MongoDB, Kafka, Flink
   - Self-hosted in government datacenter

**Event Flow**:
```
[User Action]
    ↓
[Command API] → Validate business rules
    ↓
[Append Event to MongoDB Event Store]
    ↓
[Publish Event to Kafka]
    ↓
[Flink Processes Event Stream]
    ↓ ↓ ↓
[Update Read Models] [Fraud Detection] [Analytics]
    ↓                      ↓                 ↓
[MongoDB Collections]  [Alerts]        [Dashboards]
```

**Benefits of This Architecture**:
- **Auditability**: Complete history of all state changes
- **Debuggability**: Replay events to reproduce issues
- **Scalability**: Independent scaling of read/write sides
- **Flexibility**: New projections can be created from existing events
- **Real-time insights**: Flink processes event stream for analytics
- **Regulatory compliance**: Immutable audit log by design

---

## Technology Stack Summary

### Core Infrastructure
- **Backend Services**: Node.js/NestJS with TypeScript (microservices architecture with event sourcing)
- **Database**: MongoDB (primary and only database - event store and read models)
- **Event Streaming**: Apache Kafka (event bus for event sourcing)
- **Stream Processing**: Apache Flink (complex event processing, real-time analytics, pattern detection)
- **API Gateway**: Kong
- **Container Orchestration**: Azure Kubernetes Service (AKS)
- **Infrastructure**: On-premises datacenter with AKS (no cloud dependencies)

### Mobile Applications
- **Android**: Kotlin with Jetpack Compose (native)
- **iOS**: Swift with SwiftUI (native)
- **Alternative**: React Native considered but rejected (see ADR-002)

### Authentication & Security
- **Primary Authentication**: ConsentID (OAuth 2.0 / OIDC integration)
- **Token Management**: JWT with refresh tokens
- **Biometric Authentication**: Platform-native APIs (BiometricPrompt for Android, LocalAuthentication for iOS)
- **Encryption**: TLS 1.3, AES-256 for data at rest

### Integration & Communication
- **API Protocol**: RESTful APIs with OpenAPI 3.0 specification
- **QR Code**: Dynamic QR generation with time-limited tokens (60s TTL)
- **Real-time Communication**: WebSockets for live transaction updates
- **External Integrations**: ConsentID, Tax Administration (income verification), operator systems

### Monitoring & Compliance
- **Logging**: ELK Stack (Elasticsearch, Logstash, Kibana)
- **Monitoring**: Prometheus + Grafana (including Flink metrics)
- **APM**: Azure Application Insights
- **Stream Processing Monitoring**: Apache Flink Dashboard, Kafka monitoring
- **Compliance Auditing**: Immutable event store in MongoDB with cryptographic hashing

---

## Implementation Phases

### Phase 0: Foundation & Planning (Months 1-2)

**Timeline**: 8 weeks
**Objective**: Establish project governance, technical foundation, and regulatory framework

**Key Deliverables**:
- Project charter and governance model
- Technical architecture blueprint
- ConsentID integration agreement and API access
- Development environment setup
- CI/CD pipeline foundation
- Security compliance framework (GDPR, Serbian data protection laws)
- Regulatory approval for pilot program
- Initial team recruitment (12-15 person core team)

**Prerequisites**:
- Government mandate and budget allocation
- Legal framework for Central Player Registry
- ConsentID API documentation and test environment access
- Agreements with 3-5 pilot operators

**Success Criteria**:
- All regulatory approvals obtained
- Development environments operational
- ConsentID test environment accessible
- Team fully staffed with security clearances
- Architecture review board established

**Risks & Mitigations**:
- **Risk**: ConsentID API delays or limitations
  - **Mitigation**: Early engagement with ConsentID team, fallback to mock authentication for initial development
- **Risk**: Regulatory approval delays
  - **Mitigation**: Parallel track regulatory engagement with technical planning
- **Risk**: Recruitment challenges for specialized roles
  - **Mitigation**: Partner with specialized recruitment firms, consider international talent with security clearances

---

### Phase 1: Core Platform Development (Months 3-7)

**Timeline**: 20 weeks
**Objective**: Build foundational Central Player Registry, authentication system, and basic mobile applications

**Key Deliverables**:

**Backend Services**:
- Central Player Registry database schema and APIs
- User management service (registration, profile, authentication)
- ConsentID integration service (OAuth 2.0 flow)
- Transaction processing engine (basic validation)
- Spending limit enforcement engine
- Administrative portal (operator management, user support)

**Mobile Applications**:
- iOS and Android apps with core functionality:
  - ConsentID authentication flow
  - User registration and profile management
  - Monthly spending limit configuration
  - Transaction history view
  - Basic QR code generation (for Phase 2)
  - Push notification infrastructure

**Infrastructure**:
- Azure Kubernetes Service (AKS) cluster setup (staging environment)
- MongoDB cluster with replica sets (event store and read models)
- Apache Kafka cluster (3 brokers minimum)
- Apache Flink cluster for stream processing
- API Gateway (Kong) configuration
- Basic monitoring and logging (Prometheus, Grafana, ELK)

**Security**:
- End-to-end encryption implementation
- Secure token storage on mobile devices
- Penetration testing framework
- Initial security audit

**Prerequisites**:
- Phase 0 completion
- ConsentID production API access granted
- Mobile developer accounts (Apple Developer, Google Play Console)

**Success Criteria**:
- ConsentID authentication successfully tested with 100 test users
- Central Player Registry stores and retrieves user data correctly
- Mobile apps pass internal QA testing
- API response times < 200ms (95th percentile)
- Zero critical security vulnerabilities
- All services containerized and deployable via CI/CD

**Risks & Mitigations**:
- **Risk**: ConsentID integration complexity exceeds estimates
  - **Mitigation**: Allocate 2-week buffer for OAuth integration, maintain close communication with ConsentID technical team
- **Risk**: Database schema changes during development
  - **Mitigation**: Use database migration tools (Flyway/Liquibase), maintain strict version control
- **Risk**: Mobile platform compatibility issues
  - **Mitigation**: Test on wide range of devices (minimum 20 models), establish device compatibility matrix

---

### Phase 2: Pilot Program - Online Operators (Months 8-12)

**Timeline**: 20 weeks
**Objective**: Launch limited pilot with 1,000-5,000 users and 3-5 online operators

**Key Deliverables**:

**Operator Integration**:
- Operator API SDK (documentation, sample code, test environment)
- One-time token registration system for online accounts
- Real-time transaction validation API
- Self-exclusion enforcement API
- Marketing opt-out compliance API
- Operator onboarding portal

**Enhanced Mobile Features**:
- Online account registration flow (token generation)
- Real-time spending limit tracking with notifications
- Self-exclusion management (3/6/12 months, 5/10 years, permanent)
- Marketing preferences management
- Enhanced transaction details (per-operator breakdown)
- Biometric authentication for sensitive operations

**Backend Enhancements**:
- Real-time cross-operator spending aggregation
- Spending limit cooling-off period enforcement (72-hour delay for increases)
- Self-exclusion registry with immediate enforcement
- Transaction reconciliation system
- Enhanced audit logging
- Income verification integration (Tax Administration API)
- General and personal maximum limit calculations

**User Support**:
- Help desk system
- FAQ and documentation
- In-app support chat
- User feedback collection mechanism

**Testing & Validation**:
- Beta testing program (1,000 users)
- Operator integration testing (3-5 operators)
- Load testing (5,000 concurrent users)
- Security penetration testing (external firm)
- GDPR compliance audit

**Prerequisites**:
- Phase 1 completion with all acceptance criteria met
- Pilot operators signed up and trained
- Apple App Store and Google Play Store approvals
- Production infrastructure deployed
- 24/7 support team staffed

**Success Criteria**:
- 1,000-5,000 active users registered
- 3-5 operators successfully integrated
- < 5% technical failure rate
- > 80% user satisfaction score
- Zero security incidents
- < 100ms transaction validation latency
- 99.5% API uptime
- Regulatory approval to proceed to Phase 3

**Risks & Mitigations**:
- **Risk**: Low user adoption during pilot
  - **Mitigation**: Incentive program for early adopters, extensive user education campaign
- **Risk**: Operator integration difficulties
  - **Mitigation**: Dedicated integration support team, comprehensive API documentation, sandbox environment
- **Risk**: Performance issues under load
  - **Mitigation**: Extensive load testing, auto-scaling infrastructure, Redis caching optimization
- **Risk**: User experience friction
  - **Mitigation**: UX testing with focus groups, iterative design improvements, simplified onboarding flow

---

### Phase 3: Mandatory Online Deployment (Months 13-18)

**Timeline**: 24 weeks
**Objective**: Scale to all online operators, enforce mandatory compliance

**Key Deliverables**:

**Scale-Up Infrastructure**:
- Production AKS cluster expansion (handle 100,000+ users)
- Additional hardware nodes for horizontal scaling
- MongoDB sharding for distributed data storage
- Enhanced DDoS protection (hardware appliances)
- Auto-scaling configuration for peak loads (HPA policies)

**Operator Migration**:
- Bulk user migration tools (existing operator accounts → Central Registry)
- Legacy system integration adapters
- Operator compliance monitoring dashboard
- Automated compliance reporting
- Enforcement mechanism for non-compliant operators

**Enhanced Features**:
- Advanced analytics dashboard for users (spending trends, patterns)
- Parental controls for young adults (18-21 age bracket with enhanced protections)
- Integration with problem gambling support services
- Anonymous helpline integration
- Enhanced notification system (approaching limits, self-exclusion reminders)

**Regulatory Compliance**:
- Comprehensive audit trail system
- Regulatory reporting automation
- Data retention and deletion policies implementation
- GDPR right-to-be-forgotten implementation
- Cross-border player handling (non-residents)

**Public Education**:
- National media campaign
- Tutorial videos and guides
- Operator staff training program (nationwide)
- Problem gambling awareness materials

**Prerequisites**:
- Phase 2 pilot successful with regulatory sign-off
- Legal mandate for mandatory online operator compliance
- All operators notified with 6-month compliance deadline
- Migration support team established

**Success Criteria**:
- 50,000-100,000 active users
- 95%+ of licensed online operators integrated
- < 2% technical failure rate
- 99.9% API uptime
- Zero critical security incidents
- Successful migration of 80%+ existing players within 6-month window
- Compliance enforcement actions against non-compliant operators

**Risks & Mitigations**:
- **Risk**: Operator resistance to mandatory integration
  - **Mitigation**: Clear regulatory mandate, phased deadline approach, integration support incentives
- **Risk**: Infrastructure unable to handle scale
  - **Mitigation**: Gradual rollout by operator size, continuous load testing, over-provisioned infrastructure
- **Risk**: User migration friction
  - **Mitigation**: Automated migration tools, dedicated support channels, grace period for dual access
- **Risk**: Political or regulatory pushback
  - **Mitigation**: Transparent reporting on pilot success, stakeholder engagement, public support campaign

---

### Phase 4: Physical Casino Integration (Months 19-24)

**Timeline**: 24 weeks
**Objective**: Extend system to physical casinos, betting shops, and cash-based gambling

**Key Deliverables**:

**QR Code Transaction System**:
- Enhanced QR code generation (60-second TTL, encrypted payload)
- Cashier terminal integration SDK
- Offline transaction queuing (for network outages)
- Transaction reconciliation system
- Receipt printing integration (unique transaction IDs)

**Physical Location Support**:
- Kiosk application for users without smartphones
- Staff training program and materials
- Hardware subsidy program for small operators
- POS system integration adapters (existing casino systems)
- Backup authentication methods (NFC cards as fallback)

**Non-Resident Support**:
- Passport-based registration system (via casino cashiers)
- International ID validation
- Currency conversion handling
- Tax withholding integration

**Enhanced Security**:
- Anti-fraud detection (pattern analysis for QR code abuse)
- Geolocation verification (transaction must occur at registered venue)
- Camera integration for physical verification (optional, privacy-compliant)
- Transaction velocity limits

**Operational Features**:
- Venue management portal
- Real-time transaction monitoring per venue
- Cash reconciliation reports
- Compliance dashboards for casino operators
- Staff access controls and audit logs

**Prerequisites**:
- Phase 3 completion
- Legal framework for physical casino integration
- Casino operator agreements
- Hardware procurement (kiosks, scanners)
- Pilot program with 3-5 large casinos

**Success Criteria**:
- QR code transactions complete in < 5 seconds (95th percentile)
- < 1% QR code validation failures
- Kiosks operational in 50+ high-traffic locations
- 90%+ of physical gambling venues integrated
- Zero cash discrepancy incidents in reconciliation
- Successful handling of peak loads (e.g., major sporting events)

**Risks & Mitigations**:
- **Risk**: Older demographic unable to use smartphone app
  - **Mitigation**: Kiosk deployment, in-person assistance, alternative card-based system
- **Risk**: Network connectivity issues in remote locations
  - **Mitigation**: Offline mode with delayed sync, 4G/5G backup connections, local transaction caching
- **Risk**: QR code fraud or replay attacks
  - **Mitigation**: Time-limited codes, encrypted payloads, geolocation verification, one-time use enforcement
- **Risk**: Hardware compatibility with legacy casino systems
  - **Mitigation**: Flexible integration adapters, phased rollout, hardware upgrade subsidies

---

### Phase 5: Optimization & Advanced Features (Months 25-30)

**Timeline**: 24 weeks (ongoing)
**Objective**: Continuous improvement, advanced analytics, and responsible gambling enhancements

**Key Deliverables**:

**Advanced Responsible Gambling Tools**:
- AI-powered risk detection (identify problem gambling patterns)
- Proactive intervention system (automated warnings, cooldown suggestions)
- Integration with healthcare system (therapist referrals, medical override for self-exclusion)
- Family protection features (linked accounts for couples)
- Financial counseling integration

**Analytics & Reporting**:
- National gambling statistics dashboard (anonymized, aggregated data)
- Research data export for academic studies (privacy-compliant)
- Operator benchmarking reports
- Tax revenue optimization analytics
- Public health reporting integration

**Platform Enhancements**:
- Multi-language support (English, minority languages)
- Accessibility improvements (screen readers, high contrast mode)
- Apple Watch / Android Wear integration
- Voice assistant integration (limit checking, transaction history)
- Progressive Web App (PWA) for desktop access

**Ecosystem Expansion**:
- International player support (tourists, cross-border workers)
- Cryptocurrency integration (if legally permitted)
- Lottery integration
- Sports betting enhancements (live betting controls)

**Prerequisites**:
- Phase 4 successful deployment
- Stable user base (100,000+ active users)
- Data science team established
- Partnership with healthcare providers

**Success Criteria**:
- AI risk detection identifies 70%+ of problem gamblers before self-reporting
- 30%+ reduction in problem gambling cases (year-over-year)
- Positive public health outcomes documented
- International recognition as best-practice system
- 90%+ user satisfaction with advanced features

**Risks & Mitigations**:
- **Risk**: Privacy concerns with AI monitoring
  - **Mitigation**: Transparent opt-in system, strict data minimization, independent privacy audits
- **Risk**: Feature creep and complexity
  - **Mitigation**: Strict prioritization framework, user research-driven development
- **Risk**: Healthcare integration challenges
  - **Mitigation**: Pilot with willing providers, clear liability framework, gradual rollout

---

## Architecture Decision Records

### ADR-001: Use ConsentID for Primary Authentication

**Status**: Accepted

**Context**:
The е-Играч system requires verifiable age verification and identity assurance to prevent underage gambling and ensure regulatory compliance. We need a trusted authentication mechanism that:
- Provides legally binding proof of age (18+ requirement)
- Integrates with existing Serbian government identity infrastructure
- Offers high security and cannot be easily spoofed
- Supports OAuth 2.0 / OpenID Connect standards for mobile integration
- Eliminates need for custom identity verification infrastructure

**Decision**:
Integrate with ConsentID (eid.gov.rs) as the primary authentication provider for Serbian citizens. ConsentID authentication will be mandatory for account activation. The system will use OAuth 2.0 authorization code flow with PKCE (Proof Key for Code Exchange) for mobile app authentication.

**Consequences**:

*Positive*:
- Legally binding age verification (maloletnici cannot bypass)
- No need to implement custom KYC infrastructure
- Leverages existing government identity system trusted by citizens
- Automatic updates when government ID data changes
- Reduced liability for identity verification failures
- Strong authentication inherently supports MFA (government eID credentials)
- Simplified user experience (citizens already familiar with eUprava login)

*Negative*:
- Dependency on external government system (availability risk)
- No control over ConsentID feature roadmap or changes
- Potential latency during authentication flow (external API calls)
- Limited to Serbian citizens initially (non-residents require alternative flow)
- Possible user experience friction if ConsentID has issues
- Requires ongoing coordination with government ConsentID team

**Alternatives Considered**:

1. **Custom Identity Verification System**
   - Rejected: High cost, regulatory complexity, lower trust than government system

2. **Third-Party KYC Providers (e.g., Onfido, Jumio)**
   - Rejected: Expensive per-verification costs, not integrated with Serbian government data, legal uncertainty

3. **Bank ID Systems (BankID)**
   - Rejected: Not all citizens have bank accounts, fragmented across banks, no government mandate

4. **Manual ID Verification at Physical Locations**
   - Rejected: Not scalable, high operational cost, poor user experience, fraud risk

---

### ADR-002: Native Mobile Applications (iOS/Android) vs. Cross-Platform

**Status**: Accepted

**Context**:
We need to deliver high-quality mobile applications for both iOS and Android platforms. Key considerations:
- Biometric authentication integration (Face ID, Touch ID, fingerprint)
- QR code generation with precise timing (60-second expiration)
- Push notifications for transaction alerts and limit warnings
- Secure storage of refresh tokens and sensitive data
- Performance requirements (< 200ms UI response, real-time transaction updates)
- Future integration with NFC (for physical casino card alternative)
- Government-grade security requirements

**Decision**:
Develop native applications using Kotlin (Android) and Swift (iOS) with Jetpack Compose and SwiftUI respectively. Reject cross-platform frameworks (React Native, Flutter).

**Consequences**:

*Positive*:
- Superior performance and battery efficiency
- Native access to platform security features (Keychain, Keystore)
- First-class biometric authentication support
- Better integration with OS-level features (Widgets, Shortcuts, WatchKit)
- Easier compliance with App Store / Play Store security requirements
- Smaller attack surface (no JavaScript bridge vulnerabilities)
- Platform-specific UI/UX best practices (native look and feel)
- Future-proof for platform-specific features

*Negative*:
- Higher development cost (two separate codebases)
- Longer initial development time
- Need for two specialized development teams
- Maintenance overhead across two platforms
- Feature parity challenges (must coordinate releases)
- Duplicate QA effort

**Alternatives Considered**:

1. **React Native**
   - Rejected: Security concerns with JavaScript runtime, slower biometric integration, performance overhead for cryptographic operations

2. **Flutter**
   - Rejected: Less mature security ecosystem, government security auditors unfamiliar with Dart, uncertain long-term Google support

3. **Progressive Web App (PWA)**
   - Rejected: Cannot meet biometric authentication requirements, poor offline support, no push notification reliability, inadequate secure storage

**Justification**:
For a government-mandated security-critical application handling financial transactions and personal identity data, the additional cost of native development is justified by superior security, performance, and regulatory compliance.

---

### ADR-003: Event Sourcing with MongoDB as Primary Event Store

**Status**: Accepted

**Context**:
The Central Player Registry is the core of the system, storing:
- Player identity data (linked to ConsentID)
- Complete transaction history across all operators
- Spending limits and self-exclusion status
- Marketing preferences

Key requirements:
- GDPR compliance (right to erasure, data minimization, pseudonymization)
- High-performance transaction validation (< 100ms)
- Support for 100,000+ concurrent users
- Complete auditability and immutability of all state changes
- Privacy protection (operators should not receive JMBG or full identity data)
- Cross-operator transaction aggregation in real-time
- Event replay capability for debugging and compliance audits
- Temporal queries (state of system at any point in time)

**Decision**:
Implement **Event Sourcing** architecture using **MongoDB as the primary and only database**:

**Event Sourcing Architecture**:
Instead of storing current state, store all state changes as immutable events. Current state is derived by replaying events.

**MongoDB Collections**:

1. **events** (Event Store - Source of Truth)
   ```javascript
   {
     _id: ObjectId(),
     aggregateId: "player-uuid",
     aggregateType: "Player|Transaction|Limit",
     eventType: "PlayerRegistered|TransactionAuthorized|LimitIncreased|SelfExclusionActivated",
     eventData: { /* event-specific payload */ },
     metadata: {
       timestamp: ISODate(),
       userId: "player-uuid",
       operatorId: "operator-id",
       ipAddressHash: "hash",
       correlationId: "request-trace-id"
     },
     version: 1, // optimistic locking
     sequenceNumber: 12345 // global ordering
   }
   ```
   - **Indexes**: aggregateId + version, aggregateType + timestamp, eventType + timestamp
   - **Immutable**: Never updated or deleted (append-only)
   - **Sharding**: Shard by aggregateId for horizontal scaling

2. **player_read_model** (CQRS Read Model)
   ```javascript
   {
     _id: "player-uuid",
     consentIdSubjectId: "consent-id",
     profile: {
       verifiedName: "Petar Petrović",
       dateOfBirth: ISODate("1990-01-01"),
       addressHash: "hash"
     },
     limits: {
       monthlyLimit: 20000,
       currentMonthSpending: 5000,
       lastLimitChange: ISODate(),
       coolOffEndDate: null
     },
     selfExclusion: {
       active: false,
       startDate: null,
       endDate: null,
       type: null
     },
     marketingOptOut: false,
     accountStatus: "active",
     lastEventVersion: 42, // last processed event
     updatedAt: ISODate()
   }
   ```
   - **Purpose**: Fast reads for transaction validation
   - **Updated**: Asynchronously by Flink event processors
   - **Indexes**: consentIdSubjectId (unique), limits.currentMonthSpending

3. **transaction_read_model** (CQRS Read Model)
   ```javascript
   {
     _id: "transaction-uuid",
     playerId: "player-uuid",
     operatorId: "operator-id",
     amount: 5000,
     transactionType: "deposit|bet|withdrawal",
     status: "authorized|completed|rejected",
     timestamp: ISODate(),
     monthYear: "2025-10" // for monthly aggregation
   }
   ```
   - **Purpose**: Transaction history queries, monthly spending aggregation
   - **Indexes**: playerId + timestamp, monthYear + playerId
   - **TTL Index**: Auto-delete after 7 years (regulatory compliance)

4. **operator_accounts_read_model** (CQRS Read Model)
   ```javascript
   {
     _id: ObjectId(),
     playerId: "player-uuid",
     operatorId: "operator-id",
     operatorAccountId: "operator-specific-id",
     registrationToken: "token-hash",
     linkedAt: ISODate()
   }
   ```

5. **snapshots** (Performance Optimization)
   ```javascript
   {
     _id: ObjectId(),
     aggregateId: "player-uuid",
     aggregateType: "Player",
     snapshot: { /* full aggregate state */ },
     version: 100, // event version at snapshot
     createdAt: ISODate()
   }
   ```
   - **Purpose**: Avoid replaying thousands of events (snapshot every 100 events)
   - **Rebuild**: Can be deleted and rebuilt from event store

**Event Processing Flow**:
1. **Command arrives** (e.g., "Authorize Transaction")
2. **Load aggregate** from snapshot (if exists) + events since snapshot
3. **Validate command** against current state
4. **Append event** to events collection (immutable)
5. **Publish event** to Kafka
6. **Flink processes event** → updates read models asynchronously
7. **Return response** to caller

**CQRS (Command Query Responsibility Segregation)**:
- **Commands** (writes): Go through event store
- **Queries** (reads): Use read models (optimized for fast reads)

**Consequences**:

*Positive*:
- **Complete audit trail**: Every state change recorded as event (compliance win)
- **Temporal queries**: Can reconstruct system state at any point in time
- **Event replay**: Debug production issues by replaying events
- **Horizontal scalability**: MongoDB sharding on aggregateId
- **Flexible schema**: MongoDB document model handles evolving event schemas
- **GDPR compliance**: Events can be pseudonymized, read models can be deleted/rebuilt
- **No impedance mismatch**: Events are naturally JSON documents
- **Microservices friendly**: Services consume events independently
- **Real-time analytics**: Flink processes event stream for insights
- **Eventually consistent**: Acceptable for gambling domain (spending limits updated within seconds)

*Negative*:
- **Eventual consistency**: Read models lag behind event store (typically < 1 second)
- **Complexity**: Event sourcing has learning curve, more complex than CRUD
- **Event schema evolution**: Must handle old event versions (versioning strategy required)
- **Storage growth**: Event store grows indefinitely (mitigated by archiving old events)
- **Debugging**: Harder to debug (must understand event flow)
- **No ad-hoc queries**: Must create read models for new query patterns

**Alternatives Considered**:

1. **PostgreSQL with Traditional CRUD**
   - Rejected: Lacks audit trail, horizontal scaling difficult, schema migrations complex

2. **Cassandra for Event Store**
   - Rejected: Overkill for initial scale, operational complexity, weaker query capabilities

3. **Hybrid (PostgreSQL + MongoDB)**
   - Rejected: Operational complexity of two databases, unclear data ownership

4. **Azure CosmosDB**
   - Rejected: Vendor lock-in, higher cost, MongoDB compatibility mode has limitations

**Event Versioning Strategy**:
- **Event Version Field**: Each event has `schemaVersion: 1`
- **Upcasting**: Old events transformed to new schema on read
- **Never mutate events**: Add new event types instead of changing existing

**Data Retention Policy**:
- **Event store**: 7 years minimum (regulatory), then archive to cold storage (Azure Blob)
- **Read models**: Can be deleted and rebuilt from events (no regulatory requirement)
- **Snapshots**: Ephemeral (can be deleted, rebuilt from events)
- **Right to erasure**: Pseudonymize events (replace PII with tombstone), delete read models

---

### ADR-004: Event-Driven Architecture with Kafka and Apache Flink for Complex Event Processing

**Status**: Accepted

**Context**:
Transaction validation and complex event processing must occur in real-time when:
- Player deposits funds with online operator
- Cashier scans QR code at physical location
- Player attempts to increase spending limit
- Detecting problem gambling patterns
- Aggregating spending across operators
- Enforcing regulatory compliance rules

Requirements:
- < 100ms latency for transaction approval/rejection
- Handle concurrent transactions across multiple operators
- Aggregate spending across all operators in real-time
- Enforce spending limits atomically (prevent race conditions)
- Self-exclusion checks must be immediate
- System must handle 10,000+ transactions per minute (peak sporting events)
- Complex event pattern detection (fraud, problem gambling)
- Real-time analytics and reporting
- Event replay for audit and debugging

**Decision**:
Implement event-driven architecture with **Apache Kafka** (event bus) and **Apache Flink** (stream processing):

**Architecture Components**:

1. **Apache Kafka** (Event Bus)
   - Topics: `player-events`, `transaction-events`, `limit-events`, `self-exclusion-events`
   - Partitioning: By playerId (ensures ordering per player)
   - Retention: 7 days (events persisted in MongoDB for long-term storage)
   - Replication factor: 3 (high availability)

2. **Apache Flink** (Stream Processing Engine)
   - Processes events from Kafka in real-time
   - Updates MongoDB read models (CQRS)
   - Detects complex patterns (fraud, problem gambling)
   - Aggregates metrics (spending per player, per operator)
   - Windowing for time-based analytics (hourly, daily, monthly)

3. **MongoDB** (Event Store + Read Models)
   - Event store: Immutable source of truth
   - Read models: Materialized views for fast queries

**Processing Flow**:

```
[Command API] → [Domain Service] → [Event Store (MongoDB)]
                                           ↓
                                    [Publish Event]
                                           ↓
                                      [Kafka Topic]
                                     ↙     ↓     ↘
                            [Flink Job 1] [Flink Job 2] [Flink Job 3]
                                 ↓              ↓            ↓
                     [Update Read Model]  [Fraud Detection]  [Analytics]
                              ↓                    ↓              ↓
                        [MongoDB]          [Alert Service]   [Dashboards]
```

**Flink Processing Jobs**:

1. **Transaction Validator Job**
   ```java
   eventStream
     .keyBy(event -> event.playerId)
     .process(new TransactionValidatorFunction())
     // Validates against spending limits in real-time
     // Updates player_read_model in MongoDB
     // < 100ms latency target
   ```

2. **Spending Aggregator Job**
   ```java
   eventStream
     .filter(event -> event.type == "TransactionAuthorized")
     .keyBy(event -> event.playerId)
     .window(TumblingEventTimeWindows.of(Time.days(30)))
     .aggregate(new MonthlySpendingAggregator())
     // Calculates monthly spending per player
     // Updates MongoDB read model
   ```

3. **Problem Gambling Detector Job**
   ```java
   eventStream
     .keyBy(event -> event.playerId)
     .process(new CEPPatternDetector())
     // Detects patterns:
     // - Spending > 80% of limit in first week
     // - Multiple failed limit increase attempts
     // - Transactions at unusual hours (3-6 AM)
     // - Rapid escalation of bet sizes
     // Triggers interventions
   ```

4. **Fraud Detection Job**
   ```java
   eventStream
     .keyBy(event -> event.playerId)
     .process(new FraudDetector())
     // Detects:
     // - Multiple QR codes generated simultaneously
     // - Geographically impossible transactions
     // - Account sharing patterns
     // - Velocity anomalies
   ```

5. **Read Model Projector Job**
   ```java
   eventStream
     .keyBy(event -> event.aggregateId)
     .process(new ReadModelProjector())
     // Updates all read models from events
     // Ensures eventual consistency
     // Can rebuild read models from scratch
   ```

**Event Processing Guarantees**:
- **Exactly-once semantics**: Flink checkpointing + Kafka transactional writes
- **Event ordering**: Guaranteed per player (keyBy playerId)
- **Fault tolerance**: Flink state snapshots, automatic recovery
- **Idempotency**: Event handlers check `lastEventVersion` before applying

**Consequences**:

*Positive*:
- **Real-time analytics**: Flink processes millions of events per second
- **Complex pattern detection**: CEP library for fraud/problem gambling
- **Horizontal scalability**: Add Flink task managers under load
- **Separation of concerns**: Each Flink job handles specific domain
- **Event replay**: Rebuild read models from event store
- **Stateful processing**: Flink maintains state for aggregations
- **Watermarking**: Handles out-of-order events
- **Backpressure handling**: Flink automatically throttles sources
- **Exactly-once guarantees**: No duplicate processing
- **Low latency**: Sub-second event processing

*Negative*:
- **Operational complexity**: Kafka + Flink require specialized expertise
- **Eventual consistency**: Read models lag ~100-500ms behind events
- **Resource intensive**: Flink requires memory for state
- **Learning curve**: Team needs training on stream processing concepts
- **Testing complexity**: Integration tests for stream processing challenging

**Alternatives Considered**:

1. **Kafka Streams Instead of Flink**
   - Rejected: Less mature CEP library, weaker exactly-once semantics, less flexible deployment

2. **Spark Streaming**
   - Rejected: Micro-batching not true real-time, higher latency than Flink

3. **Synchronous Processing (No Streaming)**
   - Rejected: Cannot achieve < 100ms latency, poor scalability, no pattern detection

4. **AWS Kinesis + Lambda**
   - Rejected: Requires cloud services, vendor lock-in

**Flink Deployment**:
- **Execution mode**: Kubernetes (Flink Native Kubernetes)
- **State backend**: RocksDB (embedded) with periodic snapshots to persistent volume
- **Checkpointing**: Every 60 seconds
- **High availability**: Flink JobManager HA via Kubernetes
- **Scaling**: Horizontal pod autoscaling based on Kafka lag

**Monitoring**:
- Flink Dashboard: Job metrics, backpressure, checkpointing
- Prometheus: Kafka lag, Flink throughput, latency
- Grafana: Real-time dashboards for event processing

---

### ADR-005: QR Code Security Model - Time-Limited JWT with Device Binding

**Status**: Accepted

**Context**:
QR codes are used for in-person transactions at casinos and betting shops. Security requirements:
- Prevent QR code sharing (user cannot screenshot and send to friend)
- Prevent replay attacks (QR code cannot be reused)
- 60-second expiration window
- Bind to specific transaction context (amount, operator)
- Work in offline scenarios (cashier scans, network validates)

Threat model:
- Malicious user tries to share QR code with underage friend
- Attacker intercepts QR code and attempts replay
- Man-in-the-middle attack during QR scan

**Decision**:
Implement QR codes as signed JWT tokens with the following structure:

**QR Code Payload (JWT)**:
```json
{
  "player_uuid": "a1b2c3d4-...",
  "qr_id": "unique-qr-identifier",
  "iat": 1634567890,
  "exp": 1634567950,
  "device_fingerprint": "hash-of-device-id",
  "purpose": "transaction_authorization",
  "nonce": "cryptographic-random-nonce"
}
```

**Generation Process**:
1. User clicks "Show QR Code" in mobile app
2. App generates device fingerprint (hashed combination of device ID, app installation ID)
3. App calls backend API: `POST /api/qr/generate`
4. Backend creates JWT with 60-second expiration, signs with private key
5. App displays QR code (encoded JWT), starts 60-second countdown timer

**Validation Process**:
1. Cashier scans QR code at POS terminal
2. Terminal sends JWT to validation API: `POST /api/qr/validate`
3. Backend verifies:
   - JWT signature valid
   - Expiration not exceeded (current time < exp)
   - QR ID not in used-QR-cache (Redis set with 5-minute TTL)
   - Player not self-excluded
   - Player UUID exists
4. On first successful validation:
   - Add QR ID to used-QR-cache (prevent reuse)
   - Return player pseudonymized data and authorization
5. Subsequent scans with same QR ID: Rejected

**Consequences**:

*Positive*:
- Cryptographically secure (signed JWTs cannot be forged)
- Time-limited (60-second window minimizes sharing risk)
- One-time use enforced (Redis cache prevents replay)
- Device fingerprint makes screenshot sharing detectable (future enhancement)
- Offline-capable (cashier can scan, validation happens during API call)
- Auditable (each QR generation logged with device fingerprint)

*Negative*:
- Requires precise time synchronization across systems (NTP critical)
- Redis dependency for used-QR-cache (if Redis fails, fallback to database, slight latency increase)
- Device fingerprinting not foolproof (advanced users can spoof, but raises friction)
- 60-second window might be tight for busy cashiers (user can regenerate if expired)

**Alternatives Considered**:

1. **Simple Random Codes (No JWT)**
   - Rejected: Requires database lookup for every code, no offline capability, harder to audit

2. **Longer Expiration (5-10 minutes)**
   - Rejected: Increases sharing risk, violates security requirement

3. **Challenge-Response System (Cashier Sends Challenge)**
   - Rejected: More complex UX, requires two-way communication, slower

4. **Bluetooth/NFC Instead of QR**
   - Considered for future (Phase 4 enhancement), but QR is universal and requires no hardware upgrade

**Future Enhancements**:
- Geolocation binding (QR only valid within casino geofence)
- Liveness detection (require biometric authentication to generate QR)
- Camera-based verification (optional photo capture during scan, privacy implications)

---

### ADR-006: API Design for Operator Integration - RESTful with Rate Limiting

**Status**: Accepted

**Context**:
Operators (online casinos, betting sites) must integrate with е-Играч to:
- Validate one-time registration tokens
- Request transaction authorization before accepting deposits
- Report self-exclusion status
- Sync transaction history (reconciliation)

Requirements:
- Simple integration for operators (some have limited technical capacity)
- Secure (prevent unauthorized access, API abuse)
- Well-documented (OpenAPI 3.0 spec)
- Versioned (allow backward compatibility during upgrades)
- Rate-limited (prevent DDoS, ensure fair usage)
- Auditable (log all operator API calls)

**Decision**:
Design RESTful API with the following principles:

**Authentication**:
- API Key authentication (each operator receives unique API key)
- API keys scoped to specific operator ID
- Keys stored hashed in database (bcrypt)
- Rotation policy: Annual mandatory rotation, immediate rotation on suspected compromise

**Rate Limiting**:
- Token bucket algorithm (via API Gateway)
- Limits:
  - Registration token validation: 10 requests/second per operator
  - Transaction authorization: 100 requests/second per operator
  - Transaction history: 5 requests/second per operator
- Burst allowance: 2x rate for 10 seconds
- Response headers include rate limit status

**API Versioning**:
- URL-based versioning: `/api/v1/...`, `/api/v2/...`
- Support N-1 version for 12 months after new version release
- Deprecation warnings in response headers

**Core Endpoints**:

1. **POST /api/v1/registration/validate**
   - Input: `{token: "7f9-248-680-423"}`
   - Output: `{player_uuid: "...", verified_name: "...", verified_address: "...", status: "valid"}`
   - Rate limit: 10/sec

2. **POST /api/v1/transactions/authorize**
   - Input: `{player_uuid: "...", amount: 5000, currency: "RSD", operator_transaction_id: "..."}`
   - Output: `{authorized: true, central_transaction_id: "...", remaining_limit: 15000}`
   - Rate limit: 100/sec

3. **GET /api/v1/players/{player_uuid}/status**
   - Output: `{self_excluded: false, marketing_opt_out: true, account_status: "active"}`
   - Rate limit: 10/sec

4. **POST /api/v1/transactions/report**
   - Input: `{central_transaction_id: "...", outcome: "completed", final_amount: 5000}`
   - Output: `{acknowledged: true}`
   - Rate limit: 100/sec

**SDK Provided**:
- Node.js SDK (open-source on GitHub)
- PHP SDK (many operators use PHP)
- Postman collection for testing
- Sandbox environment with test data

**Consequences**:

*Positive*:
- RESTful APIs are universally understood (low learning curve)
- OpenAPI spec enables automatic client generation
- Rate limiting prevents abuse and ensures fair usage
- API key authentication simple to implement and manage
- Versioning allows breaking changes without disrupting operators
- Sandbox environment reduces integration risk

*Negative*:
- API keys less secure than OAuth (but simpler for operators)
- RESTful not ideal for real-time updates (WebSockets considered for future)
- Versioning maintenance burden (supporting multiple versions)

**Alternatives Considered**:

1. **GraphQL API**
   - Rejected: Too complex for operators with limited technical capacity, higher learning curve

2. **OAuth 2.0 Client Credentials Flow**
   - Rejected: Overkill for server-to-server communication, operators may struggle with implementation

3. **Webhooks for Real-Time Updates**
   - Deferred to Phase 5: Operators would need to expose public endpoints (security complexity)

4. **gRPC Instead of REST**
   - Rejected: Less universal tooling support, operators may not have gRPC expertise

**Security Measures**:
- TLS 1.3 mandatory (refuse TLS 1.2 and below)
- IP whitelisting optional (operators can restrict API keys to specific IPs)
- Request signing available for high-security operators (HMAC-SHA256)
- Comprehensive audit logging (all requests logged with operator ID, timestamp, IP)

---

### ADR-007: On-Premises AKS Deployment with Horizontal Pod Autoscaling

**Status**: Accepted

**Context**:
System must handle:
- 100,000+ active users (Phase 3)
- 10,000+ transactions per minute during peak events (e.g., Champions League finals)
- Unpredictable load spikes
- 99.9% uptime requirement (less than 8.76 hours downtime per year)
- Data sovereignty requirements (all data on-premises)
- No cloud service dependencies

**Decision**:
Deploy on **on-premises Azure Kubernetes Service (AKS)** with horizontal pod autoscaling (HPA):

**Architecture**:
- **Execution environment**: Self-hosted AKS in government datacenter
- **Infrastructure**: Bare-metal servers or on-premises VMs
- **Container runtime**: containerd
- **Microservices**: All containerized (Docker images)
- **Networking**: Calico CNI for pod networking
- **Storage**: Local persistent volumes (SSD storage for MongoDB, Kafka, Flink state)

**Horizontal Pod Autoscaling (HPA)**:
- **Command Services** (Event writers):
  - Player Service: Scale at 70% CPU, max 30 pods
  - Transaction Service: Scale at 70% CPU, max 50 pods
  - Limit Service: Scale at 70% CPU, max 20 pods

- **Query Services** (Read models):
  - Player Query API: Scale at 60% CPU, max 40 pods
  - Transaction History API: Scale at 60% CPU, max 30 pods
  - Analytics API: Scale at 70% CPU, max 20 pods

- **Infrastructure Services**:
  - API Gateway (Kong): Scale at 60% CPU, max 30 pods
  - Authentication Service: Scale at 70% CPU, max 20 pods

**Stateful Services** (StatefulSets):
- **MongoDB Cluster**:
  - 3-node replica set (primary + 2 secondaries)
  - Sharding enabled for horizontal scaling
  - Persistent volumes: 2TB SSD per node
  - Anti-affinity rules: Each node on separate physical server

- **Apache Kafka Cluster**:
  - 5 brokers (for high throughput)
  - 3 ZooKeeper nodes (coordination)
  - Persistent volumes: 1TB SSD per broker
  - Replication factor: 3

- **Apache Flink Cluster**:
  - 1 JobManager (2 replicas for HA)
  - 10-20 TaskManagers (auto-scaling based on Kafka lag)
  - State backend: RocksDB on persistent volumes
  - Checkpointing: Every 60 seconds to persistent storage

**Load Balancing**:
- **External Load Balancer**: MetalLB (bare-metal load balancer)
- **Layer 7 Routing**: Kong API Gateway with rate limiting
- **Internal Service Mesh**: Optional Istio for advanced traffic management

**Storage Architecture**:
- **MongoDB**: Local SSD storage with RAID 10 for redundancy
- **Kafka**: Separate disk for commit logs (high-throughput SSDs)
- **Flink State**: NVMe SSDs for low-latency checkpointing
- **Backups**: Separate storage array for snapshots

**Consequences**:

*Positive*:
- **Data sovereignty**: All data remains on-premises
- **No cloud dependencies**: No external service failures
- **Cost predictability**: No per-transaction cloud costs
- **Full control**: Complete control over infrastructure and networking
- **Automatic scaling**: HPA handles traffic spikes automatically
- **High availability**: Multi-node deployment with pod restarts on failure
- **Infrastructure as Code**: All deployments via Helm charts and Terraform
- **Efficient resource utilization**: Kubernetes bin packing optimizes server usage

*Negative*:
- **Capital expenditure**: Upfront hardware costs (servers, storage, networking)
- **Operational complexity**: Requires DevOps team with Kubernetes expertise
- **Physical capacity limits**: Cannot instantly scale beyond available hardware
- **Maintenance burden**: Team responsible for hardware failures, updates
- **Longer procurement**: Hardware procurement can take weeks/months

**Alternatives Considered**:

1. **Azure Cloud AKS**
   - Rejected: Data sovereignty concerns, cloud dependency, ongoing costs

2. **Traditional VMs with Manual Scaling**
   - Rejected: Cannot handle unpredictable spikes, inefficient resource utilization

3. **OpenShift (Red Hat Kubernetes)**
   - Rejected: Additional licensing costs, AKS preferred for Microsoft ecosystem integration

4. **Nomad (HashiCorp)**
   - Rejected: Less mature ecosystem than Kubernetes, fewer available integrations

**Hardware Sizing (Initial Deployment)**:
- **Worker Nodes**: 10 servers (64 CPU cores, 256GB RAM, 4TB SSD each)
- **Master Nodes**: 3 servers (16 CPU cores, 64GB RAM, 1TB SSD each)
- **Storage Cluster**: Dedicated storage array (50TB usable, SSD-backed)
- **Network**: 10Gbps internal networking, redundant switches

**Capacity Planning**:
- **Initial capacity**: 50,000 users
- **Peak capacity**: 100,000 users (with all nodes active)
- **Scaling headroom**: 30% CPU/memory reserved for spikes
- **Expansion plan**: Add 2 worker nodes per 20,000 additional users

**Disaster Recovery**:
- **Secondary datacenter**: Standby AKS cluster in backup location
- **MongoDB replication**: Async replication to secondary site
- **Kafka mirroring**: MirrorMaker 2 for cross-datacenter replication
- **Backup strategy**: Daily MongoDB snapshots, 7-day retention on-site, 30-day cold storage
- **RTO (Recovery Time Objective)**: 1 hour
- **RPO (Recovery Point Objective)**: 5 minutes (real-time replication) / 24 hours (cold backup)

**Monitoring & Operations**:
- **Metrics**: Prometheus (self-hosted) for metrics collection
- **Visualization**: Grafana dashboards for infrastructure and application metrics
- **Logging**: ELK stack (self-hosted) for centralized logging
- **Alerting**: Prometheus Alertmanager with PagerDuty integration
- **Capacity monitoring**: Track resource utilization, alert at 70% capacity

---

### ADR-008: Data Protection and GDPR Compliance Strategy

**Status**: Accepted

**Context**:
The system processes highly sensitive personal data:
- Government-issued identity (via ConsentID)
- Financial transaction history
- Gambling behavior patterns
- Health-related data (self-exclusion for addiction)

Legal requirements:
- GDPR (Serbia is candidate for EU membership, aligning with GDPR)
- Serbian Law on Personal Data Protection
- Financial regulations (AML, transaction reporting)
- Data retention requirements (7 years for financial records)

Specific GDPR rights to support:
- Right to access (users view all their data)
- Right to erasure ("right to be forgotten")
- Right to data portability
- Right to rectification
- Privacy by design and default

**Decision**:
Implement comprehensive data protection strategy:

**1. Pseudonymization Architecture**:
- Player UUID used throughout system (never JMBG or passport number)
- ConsentID Subject ID mapped to Player UUID (one-way, not reversible without key)
- Operators receive only Player UUID and verified name/address (no government IDs)

**2. Data Minimization**:
- Collect only essential data for legal compliance
- Do NOT store:
  - JMBG (retrieved from ConsentID only during authentication, not persisted)
  - Full passport scans
  - Biometric data (used locally on device, never transmitted)
  - Bank account numbers (not needed, operators handle payments)

**3. Encryption**:
- Data at rest: AES-256 encryption for MongoDB (Encryption at Rest with KMIP)
- Data in transit: TLS 1.3 for all communications (MongoDB, Kafka, Flink)
- Token storage: iOS Keychain, Android Keystore (hardware-backed when available)
- Backup encryption: AES-256 for all MongoDB snapshots

**4. Access Controls**:
- Role-Based Access Control (RBAC) for administrative access
- Principle of least privilege
- Multi-factor authentication for admin access
- Audit logging for all data access (who accessed what, when)

**5. Right to Erasure Implementation**:
- User requests deletion via app
- Personal identifiers replaced with anonymized tombstone: `DELETED_USER_{timestamp}`
- Transaction amounts retained (aggregated statistics, no personal link)
- Self-exclusion records retained (anonymized) for public health research
- Operators notified to delete linked accounts
- Deletion completed within 30 days (GDPR requirement)

**6. Data Portability**:
- Export feature in app: "Download My Data"
- Output format: JSON and CSV
- Includes: Transaction history, spending limits history, self-exclusion events, account settings
- Generated within 24 hours, available for 7 days (encrypted download link)

**7. Privacy Impact Assessment (PIA)**:
- Conducted before Phase 1 launch
- Annual reviews
- Independent audit by privacy firm

**8. Data Processing Agreement (DPA)**:
- All operators sign DPA as data processors
- Government (Ministry of Finance) is data controller
- Operators prohibited from using data for other purposes

**Consequences**:

*Positive*:
- GDPR compliant (reduces legal risk)
- Enhanced user trust (transparent privacy practices)
- Pseudonymization protects user identity from operator breaches
- Minimal data collection reduces attack surface
- Right to erasure automated (no manual intervention)
- Future-proof for EU accession

*Negative*:
- Deletion complexity (coordinating across operators)
- Cannot fully anonymize transaction data (conflict with audit requirements)
- Privacy features add development time
- Encryption overhead (minimal performance impact, but measurable)

**Alternatives Considered**:

1. **Full Anonymization of All Data**
   - Rejected: Incompatible with legal audit requirements (need to trace transactions)

2. **Blockchain for Immutable Audit Trail**
   - Rejected: Right to erasure conflicts with blockchain immutability

3. **Store JMBG for Simplified Identity Verification**
   - Rejected: GDPR data minimization violation, unnecessary risk

4. **On-Premises Only (Avoid Cloud for Privacy)**
   - Rejected: Insufficient scalability, higher cost, no proven privacy advantage with proper encryption

**Compliance Monitoring**:
- Annual GDPR compliance audit (external firm)
- Quarterly internal privacy reviews
- Data protection officer (DPO) appointed (legal requirement)
- Breach notification process (72-hour reporting to authorities)
- User-facing privacy policy (clear language, accessible from app)

**Data Retention Schedule**:
- **Event Store**: 7 years (archived to cold storage after 2 years)
- **Read Models**: Can be deleted and rebuilt from events (no regulatory requirement)
- **Snapshots**: 90 days (ephemeral, for performance only)
- **Self-exclusion events**: Permanent (pseudonymized after account deletion)
- **Deleted user tombstones**: 7 years (to prevent re-registration during self-exclusion)
- **Kafka retention**: 7 days (events persisted in MongoDB)

---

### ADR-009: Mobile App Security - Certificate Pinning and Root Detection

**Status**: Accepted

**Context**:
Mobile apps are vulnerable to:
- Man-in-the-middle (MITM) attacks (intercepting API calls)
- Reverse engineering (extracting API keys, secrets)
- Rooted/jailbroken devices (bypassing security controls)
- Malicious apps on same device (attempting to steal tokens)
- Network sniffing on public WiFi

Security requirements:
- Protect API communications from interception
- Prevent debugging and tampering
- Detect compromised devices
- Secure storage of refresh tokens
- Prevent screenshot sharing of QR codes

**Decision**:
Implement multi-layered mobile security:

**1. Certificate Pinning**:
- Pin backend API certificate public key in mobile app
- Reject connections if certificate doesn't match (even if CA-signed)
- Prevents MITM attacks via rogue certificates
- Backup pins included (for certificate rotation)
- Implementation: TrustKit (iOS), Network Security Config (Android)

**2. Root/Jailbreak Detection**:
- Detect rooted Android devices (check for su binary, Magisk, common root apps)
- Detect jailbroken iOS devices (check for Cydia, abnormal file paths)
- Warning screen on detected root/jailbreak: "Your device is compromised. е-Играч may not function securely."
- Allow user to proceed with warning (not blocking, to avoid false positives)
- Log root detection events (analytics for risk assessment)

**3. SSL Pinning Bypass Detection**:
- Detect Frida, Xposed Framework (common hooking tools)
- Detect debugger attachment
- Obfuscate security checks (prevent easy bypass)

**4. Secure Token Storage**:
- iOS: Store refresh tokens in Keychain with `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`
- Android: Store in EncryptedSharedPreferences (Jetpack Security)
- Hardware-backed keystore when available
- Tokens encrypted with device-specific key (cannot be extracted and used on another device)

**5. Code Obfuscation**:
- ProGuard/R8 (Android) to obfuscate code
- Swift obfuscation tools (iOS)
- String encryption for API endpoints, keys
- Not foolproof, but raises reverse engineering difficulty

**6. Screenshot Protection for QR Codes**:
- Flag QR code screen as secure (Android: `FLAG_SECURE`)
- iOS: Blur screen when app enters background (prevent screenshot in app switcher)
- Warning message: "Do not screenshot this QR code"

**7. Network Security**:
- Disable cleartext HTTP traffic
- Enforce TLS 1.3
- Network Security Config (Android) to block custom CAs
- App Transport Security (iOS) with strict settings

**Consequences**:

*Positive*:
- MITM attacks prevented (certificate pinning)
- Awareness of compromised devices (root detection)
- Tokens protected from extraction (hardware-backed storage)
- Reverse engineering difficulty increased (obfuscation)
- Screenshot sharing of QR codes discouraged

*Negative*:
- Certificate pinning complicates certificate rotation (requires app update)
- Root detection has false positives (some users root for legitimate reasons)
- Obfuscation adds build complexity
- Security measures can be bypassed by determined attackers (no perfect solution)
- May alienate power users (some disable apps with root detection)

**Alternatives Considered**:

1. **No Root Detection (Trust All Devices)**
   - Rejected: Unacceptable risk for financial/identity app

2. **Block Rooted Devices Entirely**
   - Rejected: Too harsh, false positives would block legitimate users

3. **Runtime Application Self-Protection (RASP) Tools**
   - Considered but deferred: Expensive, complex integration, may revisit in Phase 5

4. **No Certificate Pinning (Rely on OS Certificate Store)**
   - Rejected: Vulnerable to MITM with rogue CA certificates

**Certificate Rotation Strategy**:
- Primary pin + 2 backup pins in app
- Rotate certificates annually
- 6-month overlap period (old + new cert both valid)
- Emergency rotation process (if certificate compromised):
  - Push app update with new pin within 48 hours
  - Force update (users cannot use old version)

**Incident Response**:
- Security monitoring for unusual patterns (mass root detection, pinning failures)
- Automated alerts for security events (> 5% users report pinning failures)
- Security bulletin system (inform users of threats)

---

### ADR-010: Self-Exclusion Medical Override Process - Digital Workflow Integration

**Status**: Accepted

**Context**:
Self-exclusion is a critical responsible gambling feature. Users can self-exclude for periods:
- 3, 6, 12 months
- 5, 10 years
- Permanent

Legal requirement: Self-exclusion can ONLY be lifted before expiration by approval from a licensed addiction specialist.

Challenges:
- How to integrate medical professionals into digital workflow?
- Verify doctor credentials (prevent fraud)
- Maintain patient privacy (GDPR, medical confidentiality)
- Prevent abuse (user cannot "shop" for lenient doctors)
- Audit trail for regulatory compliance

**Decision**:
Implement digital workflow with integration to Serbian Medical Chamber registry:

**Process Flow**:

1. **User Requests Early Termination**:
   - User opens app, navigates to "Self-Exclusion Status"
   - Sees remaining time (e.g., "180 days remaining")
   - Clicks "Request Early Termination" (only available after 30 days minimum elapsed)
   - System creates termination request (status: pending medical review)

2. **Medical Referral**:
   - App provides list of licensed addiction specialists (pulled from Medical Chamber registry)
   - User schedules appointment independently (system does NOT book appointments)
   - User receives unique referral code (e.g., `REF-12345`)

3. **Doctor Assessment**:
   - Doctor evaluates patient (in-person visit required)
   - Doctor logs into е-Играч Medical Portal (separate web app for healthcare providers)
   - Doctor enters referral code
   - System shows patient's self-exclusion history (not gambling transaction details - privacy protection)
   - Doctor completes assessment form:
     - Clinical notes (encrypted, only visible to medical professionals)
     - Decision: Approve / Deny
     - Justification (required field)
   - Doctor digitally signs decision (qualified electronic signature)

4. **System Processing**:
   - If approved: Self-exclusion lifted after 7-day cooling-off period
   - If denied: User notified, self-exclusion remains, cannot reapply for 90 days
   - Doctor's decision logged in audit trail
   - User cannot see doctor's clinical notes (medical confidentiality)

**Doctor Verification**:
- Integration with Serbian Medical Chamber API
- Verify:
  - Doctor license active
  - Specialization in addiction medicine or psychiatry
  - No disciplinary actions
- Real-time verification on each decision submission

**Fraud Prevention**:
- Rate limiting: User can only request medical review once per 90 days
- Doctor rate limiting: Flag doctors who approve > 80% of requests (statistical outlier detection)
- Audit by Medical Chamber: Quarterly reports on high-approval-rate doctors
- Require in-person visit (verified by doctor's statement)

**Consequences**:

*Positive*:
- Complies with legal requirement (medical approval)
- Digital workflow reduces processing time (vs. paper forms)
- Verifies doctor credentials automatically (prevents fraud)
- Audit trail for regulatory compliance
- Patient privacy protected (doctors don't see full transaction history)
- Prevents "doctor shopping" (rate limiting, statistical monitoring)

*Negative*:
- Dependency on Medical Chamber API (if unavailable, manual fallback needed)
- Requires onboarding doctors (education, account setup)
- Some doctors may resist digital workflow (prefer paper)
- 7-day cooling-off period may frustrate users (intentional design)

**Alternatives Considered**:

1. **Paper-Based Process (User Mails Doctor's Letter)**
   - Rejected: Slow (weeks), fraud risk (forged letters), no real-time verification

2. **Allow User to Self-Terminate After 50% of Period**
   - Rejected: Violates legal requirement for medical approval, undermines protection

3. **Telemedicine (Video Call with Doctor)**
   - Deferred to Phase 5: Complex, requires secure video platform, doctor licensing for telemedicine

4. **No Medical Override (Strict Enforcement Until Expiration)**
   - Rejected: Too harsh, no flexibility for legitimate recovery cases

**Doctor Portal Features**:
- Dashboard: Pending referrals assigned to doctor
- Patient history: Self-exclusion timeline (dates, durations, previous early terminations)
- Decision submission form
- Secure messaging (doctor can request additional info from patient)
- Training materials (responsible decision-making guidelines)

**Privacy Safeguards**:
- Doctors sign confidentiality agreement
- Clinical notes encrypted (only accessible to treating doctor and regulatory auditors)
- Patient gambling transaction details NOT shared with doctors
- Doctors cannot see spending amounts (only self-exclusion events)

**Audit & Compliance**:
- Medical Chamber receives quarterly reports (aggregated statistics)
- Flag outlier doctors for peer review
- Random audits of approved cases (10% sample)
- User can report doctor misconduct (anonymous channel)

---

## Critical Dependencies

### Cross-Phase Dependencies

1. **ConsentID Integration Timeline**:
   - Phase 0: API access and test environment
   - Phase 1: Production integration
   - Blocker if delayed: Cannot proceed to Phase 2 without working authentication
   - Mitigation: Early engagement, escalation path to government

2. **Operator Commitment**:
   - Phase 1: 3-5 pilot operators confirmed
   - Phase 2: Pilot operators integrated
   - Phase 3: All operators mandated
   - Blocker if delayed: Pilot cannot proceed without operators
   - Mitigation: Legal mandate, financial incentives for early adopters

3. **Infrastructure Procurement**:
   - Phase 0: Cloud environment approved
   - Phase 1: Staging environment deployed
   - Phase 2: Production environment scaled
   - Blocker if delayed: Cannot deploy without infrastructure
   - Mitigation: Pre-approved vendor contracts, budget allocation

4. **Medical Chamber Integration**:
   - Phase 2: API access for doctor verification
   - Phase 3: Doctor portal deployed
   - Not blocking for Phases 1-2, but required before medical override feature
   - Mitigation: Parallel track, manual fallback for early cases

5. **Tax Administration Integration** (Income Verification):
   - Phase 2: API access for income data
   - Phase 3: Personal maximum limit calculations
   - Not critical for pilot (use general maximum only)
   - Mitigation: Phase 3 feature, negotiate API access during Phase 1

### External System Dependencies

| System | Purpose | Criticality | Fallback |
|--------|---------|-------------|----------|
| ConsentID (eid.gov.rs) | User authentication | Critical | None - system cannot function without |
| Tax Administration | Income verification for limits | Medium | Use general maximum only |
| Medical Chamber | Doctor credential verification | Medium | Manual verification (slower) |
| Operator Systems | Transaction reporting | High | Operators cannot process bets without integration |
| Serbian Dinar Exchange Rate API | Currency conversion (non-residents) | Low | Fixed rates updated daily |

---

## Risk Register

### High-Priority Risks

| Risk | Impact | Probability | Mitigation | Owner |
|------|--------|-------------|------------|-------|
| ConsentID unavailability during peak usage | High - users cannot authenticate | Medium | Graceful degradation, cached sessions, SLA with ConsentID team | Technical Lead |
| Operator resistance to mandatory integration | High - system ineffective if operators don't comply | Medium | Legal mandate, phased enforcement, integration support team | Project Manager |
| Data breach exposing player identities | Critical - reputational damage, legal liability | Low | Multi-layer security, penetration testing, incident response plan | Security Officer |
| Infrastructure unable to handle peak loads | High - transaction failures during major events | Medium | Load testing, auto-scaling, over-provisioned resources | DevOps Lead |
| Medical override process abuse (doctor fraud) | Medium - undermines self-exclusion | Low | Statistical monitoring, Medical Chamber audits | Compliance Officer |
| Budget overruns delaying deployment | High - project timeline extended | Medium | Contingency budget (20%), phased rollout allows cost adjustment | Finance Manager |
| User adoption lower than expected | Medium - system underutilized | Medium | Public education campaign, user incentives, simplified UX | Product Manager |
| Legal challenges from gambling industry | Medium - delays, scope changes | Low | Stakeholder engagement, transparent process, legal defense fund | Legal Counsel |

### Technical Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| PostgreSQL performance bottleneck | Medium - slow transaction validation | Medium | Read replicas, partitioning, Redis caching |
| Kafka operational complexity | Medium - message processing delays | Medium | Managed Kafka (Azure Event Hubs), expert training |
| Mobile app compatibility issues | Low - some users cannot use app | High | Extensive device testing, backward compatibility |
| Certificate pinning breaks after rotation | High - app stops working | Low | Backup pins, gradual rollout, emergency update process |
| QR code timing issues (clock skew) | Medium - failed transactions | Low | NTP synchronization, grace period (5 seconds) |
| API rate limiting too aggressive | Low - legitimate traffic blocked | Medium | Generous limits, burst allowance, monitoring |

### Organizational Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Key personnel turnover | High - knowledge loss, delays | Medium | Documentation, cross-training, retention bonuses |
| Insufficient DevOps expertise | Medium - operational issues | Medium | External consultants, training program, managed services |
| Scope creep from stakeholders | Medium - timeline delays | High | Strict change control process, phased feature releases |
| Regulatory changes during development | Medium - rework required | Low | Flexible architecture, stakeholder engagement, agile methodology |

---

## Success Metrics & KPIs

### Technical Performance Metrics

- **Transaction Validation Latency**: < 100ms (95th percentile)
- **API Uptime**: 99.9% (measured monthly)
- **Mobile App Crash Rate**: < 0.1% of sessions
- **QR Code Success Rate**: > 99% (first scan)
- **Authentication Success Rate**: > 95% (ConsentID)

### User Adoption Metrics

- **Active Users** (Phase 2): 1,000-5,000
- **Active Users** (Phase 3): 50,000-100,000
- **Daily Active Users / Monthly Active Users**: > 40%
- **App Store Rating**: > 4.0 stars
- **User Satisfaction Score**: > 80%

### Responsible Gambling Metrics

- **Self-Exclusion Adoption**: > 5% of users
- **Users Approaching Limits (80%+)**: < 20% (indicates healthy limit setting)
- **Medical Override Approval Rate**: 20-40% (too high = abuse, too low = overly restrictive)
- **Marketing Opt-Out Rate**: Track baseline (informational, not target)

### Operator Compliance Metrics

- **Operator Integration Rate** (Phase 3): > 95% of licensed operators
- **Transaction Reporting Accuracy**: > 99.9%
- **Operator API Error Rate**: < 1%

### Security Metrics

- **Security Incidents**: 0 critical, < 2 medium per quarter
- **Penetration Test Pass Rate**: 100% (all critical findings remediated)
- **Data Breach Incidents**: 0
- **Root Detection Rate**: Track for analytics (informational)

### Business Metrics

- **Cost per Transaction**: Target < 0.05 RSD
- **Infrastructure Cost per User**: Target < 50 RSD/month
- **Budget Adherence**: ±10% of approved budget
- **Timeline Adherence**: Phase milestones ±2 weeks

---

## Stakeholder Communication Plan

### Internal Stakeholders

- **Ministry of Finance**: Monthly executive briefings, quarterly reviews
- **Development Team**: Daily standups, weekly sprint reviews
- **Security Team**: Weekly security reviews, immediate escalation for incidents
- **Legal/Compliance**: Bi-weekly compliance reviews

### External Stakeholders

- **Gambling Operators**: Monthly integration office hours, dedicated Slack channel
- **ConsentID Team**: Weekly sync during Phase 1, monthly thereafter
- **Medical Chamber**: Quarterly coordination meetings (starting Phase 2)
- **End Users**: In-app announcements, email newsletters, social media

### Regulatory Bodies

- **Gaming Commission**: Monthly progress reports, regulatory review meetings
- **Data Protection Authority**: Quarterly privacy compliance reports
- **Ministry of Health**: Annual responsible gambling outcomes report

---

## Conclusion

This implementation plan provides a comprehensive roadmap for deploying the е-Играч system over 18-24 months using modern **Event Sourcing** and **CQRS** architecture. The phased approach allows for:

1. **Risk Mitigation**: Pilot program validates assumptions before full deployment
2. **Incremental Value**: Users and operators benefit from each phase
3. **Organizational Readiness**: Time for training, education, and adaptation
4. **Technical Excellence**: Event-driven architecture with complete auditability

**Architectural Highlights**:
- **Event Sourcing**: All state changes captured as immutable events in MongoDB
- **Apache Flink**: Real-time stream processing for complex event patterns
- **On-Premises AKS**: Full data sovereignty, no cloud dependencies
- **Horizontal Scalability**: Kubernetes-based auto-scaling for peak loads
- **Complete Audit Trail**: Built into architecture by design (regulatory compliance)

**Critical Success Factors**:
- ConsentID integration completed on time
- Operator buy-in and compliance
- User adoption through education and simplified UX
- DevOps team expertise in Kubernetes, Kafka, Flink, MongoDB
- Event sourcing best practices and training
- Regulatory support and legal enforcement

**Technical Prerequisites**:
1. **Hardware procurement**: On-premises servers for AKS cluster (10 worker nodes + 3 master nodes)
2. **Team expertise**: Recruit/train team on event sourcing, Flink stream processing, MongoDB
3. **Infrastructure setup**: AKS, MongoDB replica sets, Kafka cluster, Flink cluster
4. **Development tools**: Event store libraries, CQRS frameworks, Flink job templates

**Next Steps**:
1. Secure executive approval and budget allocation
2. Initiate ConsentID API access negotiations
3. Recruit core project team (15-20 members including Flink/Kafka specialists)
4. Hardware procurement for on-premises AKS cluster
5. Begin Phase 0 planning activities
6. Team training on event sourcing and stream processing
7. Engage pilot operators

The system, when fully deployed, will provide Serbia with a world-class responsible gambling infrastructure built on cutting-edge event-driven architecture, protecting vulnerable users while enabling licensed operators to conduct business in a regulated, transparent manner. The event sourcing approach ensures complete auditability, temporal queries, and the ability to evolve the system by creating new projections from historical events.

---

**Document Version**: 2.0
**Last Updated**: 2025-10-25
**Author**: е-Играč Implementation Planning Team
**Status**: Refined Architecture - Event Sourcing with MongoDB, Flink, On-Premises AKS

**Architecture Approach**: Event Sourcing + CQRS + Stream Processing
**Database**: MongoDB (only database)
**Stream Processing**: Apache Flink
**Event Bus**: Apache Kafka
**Execution Environment**: On-premises AKS

**Version History**:
- v1.0: Initial plan with PostgreSQL/traditional architecture
- v2.0: Refined with event sourcing, MongoDB, Apache Flink, on-premises AKS

**Appendices** (to be developed):
- Appendix A: MongoDB Event Store Schema & Indexes
- Appendix B: Event Catalog & Versioning Strategy
- Appendix C: Flink Job Specifications & Stream Processing Patterns
- Appendix D: API Specification (OpenAPI 3.0)
- Appendix E: Mobile App Wireframes
- Appendix F: AKS Cluster Configuration (Helm Charts, YAML manifests)
- Appendix G: Hardware Procurement & Cost-Benefit Analysis
- Appendix H: Event Sourcing & CQRS Training Materials
- Appendix I: Kafka Topic Design & Partitioning Strategy
- Appendix J: MongoDB Backup & Recovery Procedures
