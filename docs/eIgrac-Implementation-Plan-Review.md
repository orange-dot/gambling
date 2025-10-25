# Feasibility Review: е-Играч Implementation Plan

**Review Date**: 2025-10-25
**Reviewed Plan**: eIgrac-Implementation-Plan.md (Version 2.0)
**Review Type**: Technical Feasibility & Risk Assessment
**Reviewer**: Architecture Review Board

---

## Executive Summary

This document provides a critical feasibility review of the е-Играч Implementation Plan (v2.0), which proposes an event-sourced architecture with MongoDB, Apache Flink, and on-premises AKS deployment. While the overall architectural vision is sound, **several critical contradictions, dependencies, and resource gaps** pose significant implementation risks that must be addressed before project kickoff.

**Risk Level**: **HIGH** - Multiple critical issues require immediate resolution before Phase 0 can commence.

**Recommendation**: **CONDITIONAL APPROVAL** - Plan requires substantial revisions in 5 key areas before proceeding.

---

## Critical Findings

### 1. Infrastructure Contradiction: "On-Premises AKS" vs. No-Cloud Stance

**Severity**: **CRITICAL**
**Impact**: Architectural foundation, vendor lock-in, operational dependencies
**References**: Lines 45, 85, 108, 1147

#### Issue Description

The plan repeatedly states "on-premises AKS with no cloud dependencies" while simultaneously specifying **Azure-specific services**:

- **Line 45**: "Azure Kubernetes Service (AKS): On-premises deployment (no cloud dependencies)"
- **Line 85**: "Container Orchestration: Azure Kubernetes Service (AKS)"
- **Line 108**: "APM: Azure Application Insights"

**Fundamental Contradiction**:
- **AKS** (Azure Kubernetes Service) is a **managed cloud service** provided by Microsoft Azure
- **Azure Application Insights** is a **cloud-based** APM service requiring internet connectivity
- True on-premises deployment cannot use AKS; it requires self-managed Kubernetes

#### Technical Reality Check

**What AKS Actually Is**:
- Managed Kubernetes control plane hosted on Azure cloud
- Requires Azure subscription and internet connectivity
- Node pools can be on-premises (Azure Stack HCI), but control plane remains in Azure
- Monthly costs for control plane and Azure services

**What "On-Premises" Actually Means**:
- All infrastructure physically located in government datacenter
- No dependency on external cloud providers
- No internet connectivity required for core operations
- Full control over all components

#### Risk Assessment

| Risk | Likelihood | Impact | Mitigation Complexity |
|------|------------|--------|----------------------|
| Azure service outage blocks deployments | High | Critical | High |
| Data exfiltration via Application Insights | Medium | Critical | Medium |
| Vendor lock-in to Azure ecosystem | High | High | High |
| Regulatory non-compliance (data sovereignty) | High | Critical | Low |
| Unexpected Azure costs | High | Medium | Low |

#### Recommendations

**Option A: Fully Self-Managed Kubernetes (Recommended)**
- Deploy **vanilla Kubernetes** (not AKS) on bare-metal servers
- Use **kubeadm** or **Kubespray** for cluster bootstrapping
- Replace Azure Application Insights with **self-hosted APM**:
  - **Grafana** (self-hosted): Metrics, logs, traces
  - **Jaeger**: Distributed tracing
  - **Prometheus + Grafana**: Metrics and alerting
  - **ELK Stack**: Logging (already in plan)

**Option B: Hybrid Approach (Not Recommended)**
- Use AKS control plane in Azure (acknowledge cloud dependency)
- Deploy worker nodes on-premises via Azure Stack HCI
- Accept vendor lock-in and ongoing Azure costs
- Require internet connectivity for Kubernetes API access

**Option C: OpenShift (Alternative)**
- Red Hat OpenShift on bare-metal (enterprise Kubernetes)
- Fully self-hosted with vendor support
- Higher licensing costs but proven stability
- Strong government/enterprise adoption

**Impact on Plan**:
- Lines 45, 85, 108, 1147 must be rewritten
- Infrastructure as Code (Helm charts) remains valid
- Additional DevOps expertise required for self-managed Kubernetes
- Hardware specifications remain unchanged
- Monitoring stack must be updated (remove Azure dependencies)

**Estimated Effort**: 2-3 weeks additional planning, 1 month implementation overhead

---

### 2. MongoDB Single-Cluster Risk: Event Store + Projections Workload Isolation

**Severity**: **HIGH**
**Impact**: Data integrity, performance, operational complexity
**References**: Lines 20, 27, 83, 299, 580-707

#### Issue Description

The plan proposes using **a single MongoDB cluster** for:
1. Event store (immutable, append-only, source of truth)
2. Read models (CQRS projections, frequently updated)
3. Snapshots (ephemeral, performance optimization)
4. Operational data (operator accounts, sessions)

**Problems**:
- **Workload contention**: Write-heavy event appends compete with read-heavy query traffic
- **No isolation**: Event store corruption could cascade to read models
- **Retention complexity**: 7-year events mixed with 90-day snapshots in same cluster
- **Schema evolution**: Projections change frequently; event schema must be immutable
- **Backup strategy**: Different backup requirements for event store vs. read models

#### Technical Concerns

**Event Sourcing Best Practices Violated**:
- Event store should be **append-only, immutable, never updated**
- Read models are **ephemeral and rebuildable** from events
- Mixing them in one database creates **unclear data lifecycle**

**Performance Implications**:
- Event store writes: High throughput, sequential writes
- Read model queries: Random access, index-heavy
- Contention on same storage backend degrades both workloads

**Operational Complexity**:
- Cannot independently scale event store vs. read models
- Backup strategy must cover both (different RPO/RTO requirements)
- Upgrades/migrations affect both workloads simultaneously

#### Risk Assessment

| Risk | Likelihood | Impact | Mitigation Complexity |
|------|------------|--------|----------------------|
| Performance degradation under load | High | High | Medium |
| Event corruption during schema migration | Medium | Critical | High |
| Inability to rebuild read models independently | Medium | High | Medium |
| Backup/restore complexity | High | Medium | Low |
| Event versioning conflicts | High | High | High |

#### Recommendations

**Option A: Segregate Event Store from Read Models (Recommended)**

**Architecture**:
```
[Event Store MongoDB Cluster]
  - Purpose: Immutable events only
  - Collections: events, snapshots
  - Optimization: Write-optimized (append-only)
  - Retention: 7 years, archive to cold storage
  - Backup: Daily full + continuous replication

[Read Model MongoDB Cluster]
  - Purpose: CQRS projections
  - Collections: player_read_model, transaction_read_model, etc.
  - Optimization: Read-optimized (indexes for queries)
  - Retention: Ephemeral (can be rebuilt)
  - Backup: Daily snapshot (lower priority)
```

**Benefits**:
- Independent scaling (add shards to event store without affecting queries)
- Clear data lifecycle (events immutable, projections rebuildable)
- Simplified backup strategy (different RPO/RTO per cluster)
- Failure isolation (read model corruption doesn't affect events)

**Costs**:
- Double MongoDB cluster hardware (+3 servers)
- Operational overhead (2 clusters to manage)
- Network bandwidth (Flink reads from event store, writes to read models)

**Option B: Purpose-Built Event Store (Alternative)**

Use specialized event store instead of MongoDB for events:
- **EventStoreDB**: Purpose-built for event sourcing (CQRS native)
- **Apache Kafka** (retain indefinitely): Events stored in Kafka topics
- **Apache Pulsar**: Event streaming with built-in tiered storage

**MongoDB only for read models**:
- Simpler schema (no event versioning complexity)
- Optimized for queries, not append-only writes

**Trade-offs**:
- Additional technology to learn and operate
- EventStoreDB less mature than MongoDB
- Kafka as event store requires careful retention tuning

**Option C: Strict Operational Discipline (Minimal Change)**

Keep single MongoDB cluster but enforce strict separation:
- **Separate databases**: `event_store` DB vs. `read_models` DB
- **Separate shards**: Event collections on dedicated shards
- **Separate connection pools**: Event writers isolated from query traffic
- **Strict schema governance**: Event schema never modified, only new versions added

**Implementation Guidelines**:
```javascript
// Event store - write-optimized shard
sh.shardCollection("event_store.events", { aggregateId: 1 })
sh.addShardTag("shard0001", "eventStore")
sh.addTagRange("event_store.events", { aggregateId: MinKey }, { aggregateId: MaxKey }, "eventStore")

// Read models - read-optimized shard
sh.shardCollection("read_models.player_read_model", { _id: 1 })
sh.addShardTag("shard0002", "readModels")
sh.addTagRange("read_models.player_read_model", { _id: MinKey }, { _id: MaxKey }, "readModels")
```

**Benefits**:
- No additional hardware
- MongoDB expertise reused
- Simpler operational model

**Risks**:
- Still single point of failure
- Contention possible during peak load
- Backup strategy complexity remains

#### Event Versioning Strategy Gap

**Current Plan**: Line 698-701 mentions event versioning but lacks detail.

**Required**:
1. **Schema version field** in every event: `schemaVersion: 2`
2. **Upcasting strategy**: Transform old events to new schema on read
3. **Version registry**: Catalog of all event types and versions
4. **Migration tooling**: Scripts to test event replay with new versions

**Recommendations Summary**

**Short-term (Phase 1)**:
- Implement Option C (strict operational discipline)
- Document event versioning strategy (Appendix B)
- Separate MongoDB databases for event store vs. read models
- Configure separate shards with shard tags

**Long-term (Phase 3)**:
- Evaluate Option A (separate clusters) if performance issues arise
- Consider EventStoreDB migration if MongoDB limitations surface
- Monitor contention metrics closely

**Estimated Effort**: 1 week planning, 2 weeks implementation, ongoing operational overhead

---

### 3. Staffing Inadequacy: 12-20 People Cannot Cover All Specialist Tracks

**Severity**: **HIGH**
**Impact**: Project delivery, quality, operational stability
**References**: Lines 129, 265, 1816

#### Issue Description

The plan estimates **12-15 core team** (Phase 0) expanding to **15-20 members** (by Phase 3), but the required skill matrix demonstrates this is **grossly inadequate**.

#### Required Specialist Tracks

**1. Backend Development** (Event Sourcing, Domain-Driven Design)
- 3-4 senior Node.js/NestJS developers
- Event sourcing expertise (CQRS, aggregates, domain events)
- ConsentID OAuth integration specialist
- API design and security

**2. Stream Processing** (Apache Flink)
- 2-3 Flink developers (Java/Scala)
- CEP (Complex Event Processing) expertise
- Real-time analytics and windowing
- State management and checkpointing

**3. Database & Data Engineering** (MongoDB)
- 2 MongoDB specialists (sharding, replica sets, performance tuning)
- Event store architecture design
- Data retention and archival strategies

**4. Infrastructure & DevOps** (Kubernetes, Kafka)
- 3-4 Kubernetes SREs (on-premises cluster management)
- Kafka cluster operations (ZooKeeper, brokers, monitoring)
- Networking (Calico CNI, MetalLB)
- Infrastructure as Code (Helm, Terraform)

**5. Mobile Development**
- 2-3 native Android developers (Kotlin, Jetpack Compose)
- 2-3 native iOS developers (Swift, SwiftUI)
- Mobile security specialists (certificate pinning, biometrics)

**6. Security & Compliance**
- 2 security engineers (penetration testing, threat modeling)
- 1 compliance officer (GDPR, Serbian data protection laws)
- 1 security auditor (continuous monitoring)

**7. QA & Testing**
- 2-3 QA engineers (mobile + backend)
- 1 performance testing specialist (load testing, Flink job validation)
- Test automation engineers

**8. Integration & Support**
- 2 operator integration specialists (SDK development, support)
- 2-3 technical support engineers (helpdesk, user issues)
- 1 ConsentID integration liaison

**9. Product & Design**
- 1 product manager
- 1 UX designer (mobile app design)
- 1 technical writer (documentation)

**10. Management & Operations**
- 1 technical lead / architect
- 1 project manager
- 1 scrum master / agile coach

#### Total Staffing Requirement

**Minimum**: **28-35 full-time employees**
**Realistic**: **35-45 FTEs** (accounting for overlap, vacations, attrition)

**Current Plan**: 15-20 people
**Shortfall**: **15-25 people** (50-60% understaffed)

#### Risk Assessment

| Risk | Likelihood | Impact | Mitigation Complexity |
|------|------------|--------|----------------------|
| Critical skills gaps (Flink, event sourcing) | High | Critical | High |
| Developer burnout from multitasking | High | High | Medium |
| Missed deadlines due to insufficient capacity | High | Critical | Medium |
| Quality issues from rushed development | High | High | Low |
| Inability to provide 24/7 support | High | Medium | Low |
| Key person dependencies (single expert per domain) | High | Critical | High |

#### 24/7 Support Calculation

**Line 265**: "24/7 support team staffed"

**Reality**:
- 24/7 coverage requires **4.2 FTEs per role** (accounting for weekends, vacations, holidays)
- Minimum 2 support tiers: L1 (helpdesk) + L2 (technical escalation)
- **Total**: 8-10 FTEs for support alone

**Current plan has no dedicated support staff allocation**.

#### Recommendations

**Option A: Expand Team to Realistic Size (Recommended)**

**Phase 0-1** (Core Build): **30-35 FTEs**
- Prioritize backend, mobile, infrastructure, Flink specialists
- Outsource QA and mobile testing to specialized firms
- Engage external consultants for Flink/event sourcing training

**Phase 2-3** (Scaling & Operations): **40-50 FTEs**
- Add support team (10 FTEs for 24/7 coverage)
- Expand DevOps for multi-cluster operations
- Hire dedicated security auditors and compliance officers

**Phase 4+** (Physical Integration): **50-60 FTEs**
- Field engineers for casino hardware support
- Training coordinators for operator staff
- Additional integration specialists

**Option B: Outsource Specialized Functions**

**Outsourced**:
- Mobile app development (contractor teams for iOS/Android)
- QA and testing (specialized testing firms)
- 24/7 support (BPO provider with SLA)
- Security audits (external penetration testing firms)

**In-house core team**: 20-25 FTEs
- Backend, infrastructure, Flink, MongoDB specialists
- Product management and technical leadership

**Risks**:
- Loss of institutional knowledge (contractors rotate)
- Quality control challenges (external teams)
- Security clearance issues (contractors may not qualify)

**Option C: Phased Ramp-Up with Extended Timeline**

Accept that 15-20 people cannot deliver in 18-24 months:
- **Phase 0-1**: 12 months (instead of 7) with 15-20 people
- **Phase 2**: 8 months (instead of 5) as team grows to 30
- **Phase 3**: 10 months (instead of 6) with full team of 40+

**New timeline**: **30-36 months** (vs. 18-24 months planned)

#### Recommendations Summary

**Immediate Actions**:
1. **Conduct skills gap analysis**: Map current team to required roles
2. **Revise budget**: Account for 35-45 FTEs (vs. 15-20 budgeted)
3. **Recruitment plan**: Identify hard-to-fill roles (Flink, event sourcing)
4. **Contractor strategy**: Define which functions to outsource

**Budget Impact**:
- **Current**: 15-20 FTEs × €60k avg = €900k - €1.2M annually
- **Revised**: 35-45 FTEs × €60k avg = €2.1M - €2.7M annually
- **Increase**: **€1.2M - €1.5M per year** (133% increase)

**Timeline Impact**:
- Accept **30-36 month timeline** with current staffing
- OR expand team to **35-45 FTEs** to meet 18-24 month target

**Estimated Effort**: 2-3 months recruitment, ongoing hiring through Phase 2

---

### 4. ConsentID Single-Point Dependency: No Fallback, Success Hinges on Locked-In SLAs

**Severity**: **CRITICAL**
**Impact**: Complete system failure if ConsentID unavailable
**References**: Lines 87, 461-476, 1633, 1665

#### Issue Description

**ConsentID (eid.gov.rs)** is the **mandatory authentication provider** for the entire system. The plan acknowledges this dependency but provides **no contingency** for ConsentID unavailability.

**From ADR-001** (Lines 461-476):
- "ConsentID unavailability during peak usage" listed as HIGH risk
- Mitigation: "Graceful degradation, cached sessions"
- **Problem**: This is insufficient for a system that cannot function without authentication

**Critical Dependency Chain**:
```
User cannot authenticate
  → User cannot use mobile app
    → User cannot register at operators
      → User cannot gamble (system fully blocked)
        → Operators lose revenue
          → Political pressure to disable system
```

#### Technical Reality

**What "Graceful Degradation" Cannot Solve**:
- New user registration: **Blocked** (requires ConsentID for age verification)
- Limit increases: **Blocked** (requires reauthentication per plan)
- Sensitive operations: **Blocked** (biometric + ConsentID reauth)
- Session expiry: **Blocked** (cannot refresh tokens without ConsentID)

**What "Cached Sessions" Provides**:
- Existing authenticated users can continue using app
- Duration: Typically 1-24 hours (depends on token TTL)
- After expiry: **Full system failure**

#### Risk Assessment

| Scenario | Likelihood | Impact | Mitigation Cost |
|----------|------------|--------|-----------------|
| ConsentID planned maintenance (4-8 hours/month) | High | High | Low |
| ConsentID unplanned outage (2-4 hours/quarter) | Medium | Critical | Medium |
| ConsentID API breaking change without notice | Low | Critical | High |
| ConsentID rate limiting during peak events | Medium | High | Medium |
| ConsentID government policy change | Low | Critical | Very High |

**Example Impact Calculation**:
- Peak gambling event: 50,000 concurrent users
- ConsentID outage duration: 2 hours
- Users unable to authenticate: 5,000 (10% session expiry rate)
- Lost transactions: €500,000 (average)
- Reputation damage: Significant

#### Current Plan Gaps

**Line 1633** (Critical Dependencies Table):
| System | Purpose | Criticality | Fallback |
|--------|---------|-------------|----------|
| ConsentID | User authentication | **Critical** | **None - system cannot function without** |

**Line 1665** (Risk Register):
| Risk | Mitigation |
|------|-----------|
| ConsentID unavailability | Graceful degradation, cached sessions, **SLA with ConsentID team** |

**Problem**: No SLA document referenced, no fallback authentication method.

#### Recommendations

**Option A: Negotiate Explicit SLA with ConsentID (Minimum Requirement)**

**Required SLA Terms**:
- **Uptime**: 99.95% (max 4.38 hours downtime per year)
- **Availability during peak hours**: 99.99% (18:00-23:00 local time)
- **Performance**: < 500ms authentication response time (95th percentile)
- **Maintenance windows**: Announced 14 days in advance, max 2 hours, off-peak only
- **Support**: Dedicated technical liaison for е-Играч integration
- **Escalation**: Direct escalation path for outages affecting gambling system

**Penalties**:
- Monetary penalties for SLA breaches
- Mandatory post-incident reviews
- Commitment to API stability (no breaking changes without 6-month notice)

**Fallback Triggers**:
- If ConsentID SLA breached 3 times in 6 months → Activate alternative auth
- If ConsentID announces policy change incompatible with gambling → Immediate contingency

**Option B: Implement Alternative Authentication (Fallback Mode)**

**Fallback Authentication Methods**:

1. **SMS-Based OTP (One-Time Password)**
   - User registers mobile number verified via SMS code
   - Age verification: Manual ID check at first operator registration (in-person)
   - Subsequent logins: SMS OTP
   - **Limitation**: No real-time age verification, fraud risk

2. **Bank ID Integration**
   - Major Serbian banks offer digital identity services
   - Age verified via bank KYC (Know Your Customer)
   - **Limitation**: Not all citizens have bank accounts (10-15% unbanked)

3. **Manual ID Verification at Operators**
   - First-time registration: User presents ID at casino/betting shop
   - Operator verifies and registers user in system
   - Subsequent logins: Password/PIN
   - **Limitation**: Poor UX, not scalable for online-only operators

4. **Temporary Authentication Tokens**
   - During ConsentID outage: Issue 24-hour temp tokens to existing users
   - Requires prior successful ConsentID auth (cached identity proof)
   - **Limitation**: Only works for existing users, not new registrations

**Recommended Fallback**: **SMS OTP + Manual ID Verification**
- Covers 95% of use cases
- Acceptable fraud risk (operators already verify IDs for payouts)
- Can be implemented in Phase 2 (after pilot validates ConsentID integration)

**Option C: Delayed Authentication Mode (Partial Service)**

**During ConsentID Outage**:
- Existing users: Can view transaction history, check limits (read-only mode)
- New transactions: **Queued** with ConsentID pending authentication
- Once ConsentID restored: Process queued transactions retroactively

**Benefits**:
- Users not completely blocked
- Operators can still accept bets (with delayed authorization)

**Risks**:
- Spending limit violations (queued transactions exceed limit)
- Regulatory issues (was bet legally placed if auth was offline?)

#### Implementation Plan

**Phase 1** (Before Pilot):
1. Draft SLA terms with ConsentID team
2. Implement cached session extension (24-hour TTL)
3. Build ConsentID health monitoring dashboard

**Phase 2** (During Pilot):
1. Negotiate and sign SLA with ConsentID
2. Implement SMS OTP fallback (inactive, ready to activate)
3. Test fallback activation procedures (chaos engineering)

**Phase 3** (Production):
1. Monitor ConsentID SLA compliance
2. Quarterly disaster recovery drills (simulate ConsentID outage)
3. Activate fallback if SLA breached repeatedly

#### Recommendations Summary

**Mandatory Before Launch**:
- [ ] Signed SLA with ConsentID (99.95% uptime, < 500ms response time)
- [ ] Fallback authentication implemented and tested (SMS OTP)
- [ ] Disaster recovery playbook for ConsentID outages
- [ ] Monitoring and alerting for ConsentID availability

**Budget Impact**: €50k - €100k (fallback development, testing, SLA negotiations)

**Timeline Impact**: 2-3 months (SLA negotiations, fallback implementation)

**Estimated Effort**: 1 developer-month (fallback auth), ongoing SLA management

---

### 5. Physical Venue Rollout: Hardware, Logistics, Offline Strategies Not Scoped

**Severity**: **MEDIUM-HIGH**
**Impact**: Phase 4 delays, cost overruns, operational failures
**References**: Lines 327-362, 357, 362

#### Issue Description

**Phase 4** (Months 19-24) proposes physical casino integration with:
- QR code scanners at cashier terminals
- Kiosks for users without smartphones
- Offline transaction queuing for network outages
- Hardware subsidies for small operators

**Current Plan Status**: **High-level description only, no detailed scoping**

#### Missing Details

**1. Hardware Procurement** (Line 362: "Hardware procurement (kiosks, scanners)")

**Questions**:
- How many kiosks needed? (50+ mentioned, but based on what analysis?)
- Kiosk specifications: Touchscreen? Biometric reader? Receipt printer?
- QR scanner types: Handheld vs. fixed? Camera-based vs. laser?
- Who manufactures kiosks? (Custom build vs. off-the-shelf?)
- Lead time for procurement? (12-18 months for custom hardware)
- Cost per unit? (€5k - €50k range depending on features)

**Total Hardware Budget**: **Not estimated**

**2. Logistics and Deployment** (Line 327: "Physical Location Support")

**Questions**:
- How many physical venues in Serbia? (100s? 1,000s?)
- Installation timeline: Can 50+ kiosks be deployed in 6 months?
- Installation teams: How many technicians needed?
- Ongoing maintenance: Who repairs broken kiosks?
- Spare parts inventory: Centralized vs. regional warehouses?
- Operator training: Who trains 1,000+ cashiers on new system?

**Total Logistics Cost**: **Not estimated**

**3. Offline Transaction Strategy** (Line 339: "Offline transaction queuing")

**Problem**: QR codes expire in 60 seconds. What if network is down?

**Current Plan**: "Offline mode with delayed sync"

**Unanswered Questions**:
- How long can transactions be queued offline? (minutes? hours?)
- What if player exceeds limit while offline? (Reverse transaction?)
- Reconciliation conflicts: How to resolve when sync fails?
- Security: Offline queues vulnerable to tampering?
- Fraud risk: Player generates QR, network drops, uses same QR at multiple venues?

**Offline Strategy**: **Not detailed**

**4. Non-Resident Support** (Lines 350-355)

**Requirements**:
- Passport-based registration at casino cashiers
- International ID validation
- Currency conversion (EUR, USD → RSD)
- Tax withholding for foreign nationals

**Questions**:
- How do cashiers verify foreign passports? (manual vs. automated?)
- Integration with border control databases? (visa status, entry records)
- Legal framework: Can Serbia enforce spending limits on tourists?
- Compliance: GDPR applies to EU tourists, Serbian law to non-EU?

**Non-Resident System**: **Not scoped**

#### Risk Assessment

| Risk | Likelihood | Impact | Mitigation Complexity |
|------|------------|--------|----------------------|
| Hardware procurement delays Phase 4 by 6+ months | High | High | Medium |
| Kiosk deployment logistics overwhelm small team | High | Medium | Low |
| Offline transaction fraud | Medium | High | High |
| Cashier training inadequate (user friction) | High | Medium | Low |
| Non-resident legal issues (cross-border gambling) | Medium | Critical | Very High |
| Hardware subsidies exceed budget | High | Medium | Low |

#### Cost Estimation (Rough Order of Magnitude)

**Hardware Costs**:
- 50 kiosks × €20k each = €1M
- 500 QR scanners × €500 each = €250k
- Receipt printers, networking equipment = €150k
- **Subtotal**: €1.4M

**Logistics Costs**:
- Installation teams (3 teams × 6 months) = €300k
- Operator training (1,000 cashiers × 4 hours × €50/hour) = €200k
- Ongoing maintenance (5 technicians × 12 months) = €300k
- **Subtotal**: €800k

**Development Costs**:
- Offline transaction queuing (2 developers × 3 months) = €60k
- Kiosk software (3 developers × 6 months) = €180k
- Non-resident module (2 developers × 4 months) = €80k
- **Subtotal**: €320k

**Total Phase 4 Additional Costs**: **€2.5M - €3M** (not in current plan)

#### Recommendations

**Option A: Detailed Phase 4 Pre-Planning (Recommended)**

**Deliverables for Phase 0-1**:
1. **Venue Survey**: Census of all casinos, betting shops, bingo halls in Serbia
   - Count: Estimated 200-500 physical venues
   - Categorize: Large casinos (kiosks) vs. small shops (scanners only)
   - Network assessment: Broadband availability, backup connectivity

2. **Hardware Specification Document**:
   - Kiosk technical specs (touchscreen size, printer type, biometric reader)
   - RFP (Request for Proposal) for hardware vendors
   - Pilot program: Deploy 5 kiosks in Phase 2 (test before mass rollout)

3. **Offline Transaction Playbook**:
   - Maximum offline queue duration: 15 minutes
   - Reconciliation process: Manual review of offline transactions
   - Fraud prevention: One-time QR IDs tracked even offline (sync to central DB when online)
   - Rollback procedure: Void transactions that violated limits while offline

4. **Cashier Training Program**:
   - 4-hour training course (video + hands-on)
   - Certification requirement for cashiers
   - Train-the-trainer model: 10 master trainers → 100 regional trainers → 1,000 cashiers

5. **Non-Resident Legal Review**:
   - Engage international law firm (cross-border gambling regulations)
   - Coordinate with Ministry of Interior (border control integration)
   - Define scope: Tourists only? EU citizens? All foreigners?

**Timeline**:
- Pre-planning: Phase 0-1 (Months 1-7)
- Pilot kiosks: Phase 2 (5 kiosks, 1,000 users)
- Mass deployment: Phase 4 (50+ kiosks, 500+ scanners)

**Option B: Defer Physical Integration to Phase 5**

**Rationale**:
- Focus first 24 months on online gambling (80% of market)
- Phase 4 → Phase 5 (Months 25-30)
- Gives more time for hardware procurement, logistics planning

**Benefits**:
- Reduces initial complexity
- Allows pilot program to inform physical rollout
- More time to negotiate hardware subsidies with operators

**Risks**:
- Regulatory pressure to include physical casinos sooner
- Operators may resist online-only mandate without physical integration timeline

**Option C: Partner with Existing POS Vendors**

**Strategy**:
- Many casinos already have POS systems (cash registers, bet terminals)
- Integrate е-Играч as **software add-on** to existing hardware
- Avoid custom kiosk procurement (use existing terminals)

**Benefits**:
- Faster deployment (no hardware procurement)
- Lower costs (software licensing vs. hardware purchase)
- Operators familiar with existing systems

**Risks**:
- Legacy POS systems may not support QR scanning
- Integration complexity (10+ different POS vendors)
- Security concerns (е-Играч software on operator-controlled hardware)

#### Recommendations Summary

**Immediate Actions (Phase 0)**:
- [ ] Conduct venue survey (count and categorize physical locations)
- [ ] Draft hardware specification document (kiosk requirements)
- [ ] Engage hardware vendors for RFP process
- [ ] Develop offline transaction playbook
- [ ] Legal review of non-resident gambling regulations

**Budget Adjustments**:
- Add **€2.5M - €3M** for Phase 4 hardware and logistics
- Allocate **€500k** for Phase 2 pilot kiosks (5 units)

**Timeline Adjustments**:
- **Option A**: Keep Phase 4 (Months 19-24) but start pre-planning in Phase 0
- **Option B**: Defer to Phase 5 (Months 25-30) for more preparation time

**Estimated Effort**: 3-6 months pre-planning, 6-9 months deployment (Phase 4)

---

## Summary of Critical Action Items

### Immediate (Before Phase 0 Kickoff)

1. **Infrastructure Decision**: Self-managed Kubernetes vs. AKS hybrid (resolve contradiction)
2. **MongoDB Architecture**: Single cluster vs. separate event store cluster (assess risk)
3. **Staffing Plan**: Revise to 35-45 FTEs or extend timeline to 30-36 months
4. **ConsentID SLA**: Initiate negotiations, draft SLA terms
5. **Phase 4 Scoping**: Conduct venue survey, draft hardware specs

### Short-Term (Phase 0, Months 1-2)

6. **Infrastructure Selection**: Finalize Kubernetes approach, update tech stack
7. **APM Replacement**: Replace Azure Application Insights with self-hosted APM
8. **Event Store Strategy**: Document event versioning, retention, backup policies
9. **Recruitment**: Begin hiring for Flink, event sourcing, mobile specialists
10. **ConsentID Fallback**: Design SMS OTP fallback authentication

### Medium-Term (Phase 1, Months 3-7)

11. **MongoDB Segregation**: Implement separate databases/shards for event store vs. read models
12. **Staffing Ramp**: Grow team to 30-35 FTEs
13. **ConsentID SLA**: Sign agreement with explicit uptime guarantees
14. **Hardware Pilots**: Deploy 5 pilot kiosks for testing

### Long-Term (Phase 2-3, Months 8-18)

15. **Team Expansion**: Reach 40-50 FTEs including 24/7 support
16. **Event Store Evaluation**: Assess if separate cluster needed (based on performance)
17. **Physical Rollout Prep**: Finalize hardware procurement, logistics planning

---

## Risk Matrix

| Risk Category | Count | Critical | High | Medium |
|---------------|-------|----------|------|--------|
| Infrastructure | 5 | 2 | 2 | 1 |
| Database | 6 | 1 | 3 | 2 |
| Staffing | 6 | 2 | 3 | 1 |
| Dependencies | 5 | 3 | 2 | 0 |
| Physical Rollout | 6 | 1 | 3 | 2 |
| **TOTAL** | **28** | **9** | **13** | **6** |

**Risk Score**: **HIGH** (9 critical risks, 13 high risks)

**Mitigation Progress**: **0%** (no risks currently mitigated)

**Recommendation**: Address critical and high risks (22 total) before Phase 0 kickoff.

---

## Budget Impact Summary

| Item | Current Budget | Revised Budget | Delta |
|------|----------------|----------------|-------|
| Staffing (annual, 3 years) | €2.7M - €3.6M (15-20 FTEs) | €6.3M - €8.1M (35-45 FTEs) | +€3.6M - €4.5M |
| Phase 4 Hardware | Not allocated | €2.5M - €3M | +€2.5M - €3M |
| ConsentID Fallback | Not allocated | €50k - €100k | +€50k - €100k |
| Self-Managed Kubernetes Overhead | Included in AKS | +€200k (training, ops) | +€200k |
| **TOTAL INCREASE** | - | - | **+€6.35M - €7.8M** |

**Current Total Budget**: €15M - €20M (estimated)
**Revised Total Budget**: €21M - €28M (48% increase)

---

## Timeline Impact Summary

| Scenario | Original Timeline | Revised Timeline | Justification |
|----------|------------------|------------------|---------------|
| **Scenario A**: Expand team to 35-45 FTEs | 18-24 months | 18-24 months | Original timeline feasible with proper staffing |
| **Scenario B**: Keep 15-20 FTEs | 18-24 months | 30-36 months | Insufficient capacity forces timeline extension |
| **Scenario C**: Defer Phase 4 | 18-24 months | 24-30 months | Physical rollout moved to Phase 5 (6-month delay) |

**Recommendation**: **Scenario A** (expand team) to maintain timeline and quality.

---

## Approval Recommendations

### Conditional Approval Criteria

The Implementation Plan (v2.0) may proceed to Phase 0 **only if** the following conditions are met:

**Critical (Must-Have)**:
- [ ] Infrastructure contradiction resolved (self-managed K8s vs. AKS)
- [ ] Staffing plan revised to 35-45 FTEs (or timeline extended to 30+ months)
- [ ] ConsentID SLA negotiations initiated (target signed by Phase 1 start)
- [ ] Budget increased to €21M - €28M (to cover additional staffing and hardware)

**High Priority (Should-Have)**:
- [ ] MongoDB event store strategy documented (single cluster procedures or separate cluster plan)
- [ ] Event versioning playbook drafted (Appendix B)
- [ ] Phase 4 pre-planning initiated (venue survey, hardware specs)
- [ ] ConsentID fallback authentication designed (SMS OTP)

**Medium Priority (Nice-to-Have)**:
- [ ] Physical rollout timeline reevaluated (consider defer to Phase 5)
- [ ] APM replacement selected (remove Azure dependencies)

### Sign-Off Required

- [ ] **Technical Architect**: Infrastructure and database strategy
- [ ] **CTO / Technical Director**: Staffing and budget approval
- [ ] **Project Manager**: Timeline and resource allocation
- [ ] **Finance Director**: Budget increase authorization
- [ ] **Ministry of Finance Representative**: Regulatory and dependency risks

---

## Document Control

**Review Version**: 1.0
**Review Date**: 2025-10-25
**Next Review**: After Phase 0 planning completion
**Status**: **READY FOR STAKEHOLDER REVIEW**

**Distribution**:
- Technical Architecture Review Board
- Ministry of Finance Project Sponsor
- е-Играч Project Management Office
- ConsentID Integration Team

**Confidentiality**: Internal Use Only

---

**END OF FEASIBILITY REVIEW**
