# Anti-Bias Analysis

## The Problem: Two Failure Modes

LLM-assisted software architecture has two opposite failure modes:

### Failure Mode 1: Transactional Bias (Default LLM Behavior)

**Root cause:** Training data over-represents CRUD tutorials, monolithic web apps, and
database-centric patterns.

**Symptoms:**
- Shared database as integration layer between logical services
- Synchronous request/response as the default communication pattern
- No failure isolation (one component's failure cascades everywhere)
- "The database" as a universal noun (conflating storage and coordination)
- Distributed transactions or 2PC for operations that should be eventually consistent
- No back-pressure, no flow control, unbounded queues
- Performance requirements stated as fixed numbers without measurement

**Why it fails:** Systems designed this way work at prototype scale but collapse under
production load, multi-region deployment, or partial failure scenarios. The database
becomes a single point of failure, a bottleneck, and a coupling mechanism.

### Failure Mode 2: Distributed Enthusiasm (Over-Correction)

**Root cause:** Applying distributed-systems patterns universally without considering
actual scale, team size, or domain structure.

**Symptoms:**
- 9+ microservices for a system processing 50 events per day
- Circuit breakers on services called once per hour
- Saga coordinators for 2-step operations
- Event sourcing in contexts that only need CRUD
- "Eventually consistent" for operations that could safely use local ACID
- Back-pressure requirements for throughputs that will never materialize
- Infrastructure topology driven by technology enthusiasm, not domain structure
- More deployment units than team members

**Why it fails:** The operational complexity (deployment, monitoring, debugging,
coordination overhead) exceeds the value of fault isolation and scalability for
systems that will never reach the scale those patterns serve.

---

## How This Overlay Corrects Both

### Against Transactional Bias

| Correction | Source | Mechanism |
|---|---|---|
| Separate storage from coordination | DDD (contexts own persistence) | Requirements can't name "the database" as integration |
| Async by default between contexts | Reactive (message-driven) | Design must justify synchronous coupling |
| Failure isolation at boundaries | Reactive (resilient) + DDD (contexts) | Each context fails independently |
| Consistency scoped to aggregates | DDD (aggregate as unit) | No system-wide transactions by default |
| Data structures before algorithms | Pike Rule 5 | Event logs and streams as primary data structures |

### Against Distributed Enthusiasm

| Correction | Source | Mechanism |
|---|---|---|
| Boundary count justified by domain | DDD (language change drives contexts) | Can't split without model rationale |
| Scale must be measured or projected | Pike Rules 1-2 | "Measure before tuning" applied to architecture |
| Simple beats clever at small n | Pike Rules 3-4 | "When in doubt, brute force" |
| Conway's Law check | DDD + organizational reality | > 5 contexts for < 3 teams is an anti-pattern |
| Back-pressure conditional on throughput | Pike + scoped Reactive | Only mandate for > 100 msg/sec boundaries |
| Monolith as valid alternative | Pike Rule 4 + DDD | "Simpler alternative" evaluation is required |

---

## Self-Test: Is This Overlay Biased?

After applying the overlay, check these indicators:

**Signs the overlay is working correctly:**
- The design has 2-4 bounded contexts with clear domain rationale for each
- Aggregates are small (5-7 entities) and own their consistency
- Reactive properties (async, back-pressure, circuit breakers) appear only at context boundaries with measured scale justification
- The "simpler alternative" section in the design doc has a genuine evaluation
- Performance budgets are marked "to be validated"

**Signs the overlay is producing distributed enthusiasm:**
- More than 5 bounded contexts for a small team
- Back-pressure requirements for a system processing < 100 events/day
- Every module has its own message queue
- Circuit breakers on internal calls
- The word "microservice" appears without domain justification
- Event sourcing in contexts that don't need audit replay

**Signs the overlay is being ignored (transactional bias leaking through):**
- A single database shared between what should be different contexts
- No failure isolation boundaries named
- Synchronous call chains without timeouts
- "The system shall update X and then Y" (prescribing transactional coupling)
- Performance requirements stated as fixed numbers without measurement caveat
- No bounded contexts identified in the design

---

## The Goldilocks Principle

For a typical 3-5 person team building a domain-specific system:

| Indicator | Too Little (Transactional) | Just Right | Too Much (Distributed) |
|---|---|---|---|
| Bounded contexts | 1 (monolith with no boundaries) | 2-4 (domain-driven) | 8+ (infrastructure-driven) |
| Coordination mechanisms | 0 (no failure handling) | 1-2 (per aggregate invariant) | 5+ (saga for everything) |
| Deployment units | 1 (can't scale any piece) | 2-3 (align with contexts) | 9+ (each context = 3 services) |
| Infrastructure patterns | None (no resilience) | At boundaries with scale | Everywhere "for safety" |
| Consistency model | Strong everywhere (can't scale) | Per-aggregate (scoped) | Eventual everywhere (over-relaxed) |
