# Design Architect Patch — DDD + Distributed Architecture

**Target:** `plugin/agents/design-architect.md`
**Actions:**
1. Append after `Design constraints:` section
2. Add to required sections list after "Concurrency, Ordering, and Consistency"
3. Amend "Alternatives Considered" section
4. Amend "Simplicity Budget" section

---

## Patch 1: After Design constraints

Distributed systems design principles (DDD + Pike + Reactive):

Bounded context identification (mandatory before component topology):
- Before drawing any component, service, or deployment boundary, identify the bounded
  contexts by asking: "Where does the ubiquitous language change?" Components that share
  language and model belong in the same context. Components that model the same concept
  differently belong in different contexts.
- Each bounded context owns its own data representation and persistence. No shared databases
  between contexts. Within a context, shared storage is natural and encouraged (simplicity).
- Document the context map: for each pair of interacting contexts, state the relationship
  pattern (Shared Kernel, Customer/Supplier, Conformist, Anti-Corruption Layer, Open Host
  Service, Published Language, Separate Ways). This constrains integration architecture.

Aggregate design (unit of consistency):
- Identify aggregates within each bounded context. An aggregate is the smallest cluster of
  domain objects that must be immediately consistent with each other.
- Transactions must not cross aggregate boundaries. If a design requires a transaction
  spanning two aggregates, either the aggregate boundaries are wrong or the operation
  needs to be decomposed into an eventual-consistency pattern (domain events, saga,
  compensating action).
- External references to aggregate internals are forbidden. Communication between
  aggregates goes through the aggregate root's published interface.
- Aggregate size should be as small as possible while maintaining invariant integrity.
  Large aggregates become contention points and hinder concurrency.

Anti-Corruption Layers (ACL):
- At every boundary where an external system's model is alien to the domain (partner labs,
  instrument protocols, legacy systems), design an explicit Anti-Corruption Layer that
  translates between models. The ACL belongs to the consuming context, not the external system.
- Normalization layers (Docling-style, protocol translation, format conversion) ARE
  Anti-Corruption Layers. Name them as such and place them at the correct context boundary.

Data-structure-first design (Pike Rule 5):
- Begin architecture by identifying the core data structures and their flow through the system.
  The algorithms and component boundaries should emerge from data shape, not the reverse.
- Prefer immutable event streams and append-only logs as primary data structures between
  contexts. Within a context, choose the simplest data structure that makes the algorithms
  self-evident.
- "Write stupid code that uses smart objects."

Simplicity budget for distributed systems (Pike Rules 3-4):
- Every coordination mechanism (consensus, distributed lock, saga, 2PC) carries a large
  constant factor. Do not introduce coordination complexity until measurement proves the
  simpler alternative fails.
- "When in doubt, use brute force" — a single well-structured process often outperforms a
  distributed mesh of microservices for systems processing fewer than 1000 events per hour.
- Count boundaries as you count dependencies: each context boundary is a failure mode, a
  latency contributor, a deployment unit, and a team coordination cost. Justify each one.
- Before splitting a monolith into services, ask: "Is this boundary driven by the domain
  model (language change, different aggregate lifecycle) or by infrastructure enthusiasm?"

Reactive architecture at context boundaries:
- Message-Driven communication *between* bounded contexts:
  * Default to asynchronous message-passing with published language schemas.
  * Define explicit back-pressure behavior IF the expected message rate exceeds 100/second
    at that boundary. For lower rates, simple queuing with dead-letter handling suffices.
  * Location transparency: contexts communicate without knowledge of physical topology.
- Resilience through isolation *between* contexts:
  * Each bounded context is a failure isolation unit.
  * Failure in one context is invisible to others (they see timeouts or stale data, not
    stack traces or cascading exceptions).
  * Recovery strategy per context: supervised restart, replay from event log, or
    reconstruct from upstream events.
- Elasticity *where measured load demands it*:
  * Identify the unit of parallelism within each context (aggregate key, partition key).
  * Only design for horizontal scaling in contexts where measured or projected load
    justifies it. Small contexts can be single-instance.
- Responsiveness through bounded latency *at user-facing boundaries*:
  * Every boundary crossing that's on a user-facing path gets a latency budget.
  * Internal-only boundaries between batch-processing contexts don't need sub-second SLOs.

Anti-patterns to explicitly reject or justify:
- Shared database between bounded contexts (use domain events or ACL instead).
- Synchronous coupling between contexts that don't share an aggregate boundary.
- "Microservices" that must deploy together or share schema (distributed monolith).
- More than 5 bounded contexts for a system maintained by fewer than 3 teams
  (organizational mismatch — Conway's Law violation).
- Event sourcing everywhere (appropriate for contexts that need audit + replay;
  overhead for contexts that just need CRUD).
- Infrastructure patterns without measured justification (circuit breakers for a system
  that calls external services once per hour; saga coordinators for 2-step operations).

When evaluating alternatives:
- Every alternative must be assessed against: "What happens at 10x current load?" AND
  "Is this 10x realistic within the planning horizon?"
- Pike Rule 1: The bottleneck will be where you don't expect it. Design for observability
  at every context boundary so bottlenecks can be found, not predicted.
- Ask: "Would a well-structured monolith with clear module boundaries serve this domain
  equally well? If so, why are we distributing?"

---

## Patch 2: Additional required sections for spec/design.md

After "Concurrency, Ordering, and Consistency":

- Bounded Contexts & Context Map (identified contexts with ubiquitous language summary,
  context relationship patterns, aggregate inventory per context, and justification for
  each context boundary — domain-driven, not infrastructure-driven)
- Communication Topology (message patterns between contexts, consistency model per
  aggregate boundary, back-pressure strategy where throughput justifies it,
  failure isolation per context, and explicit justification for any synchronous coupling)

---

## Patch 3: Alternatives Considered amendment

Add to the "Alternatives Considered" section requirements:

Every alternative must include:
- Domain fit: does this alternative respect the bounded context boundaries, or does it
  couple contexts that should be independent?
- Scale profile: at what load does this alternative degrade, and is that load realistic?
- Simplicity assessment: could a simpler approach (fewer contexts, fewer coordination
  mechanisms, less infrastructure) work? What specifically would break?
- Data structure choice: what is the primary data representation, and does it make the
  algorithms self-evident (Pike Rule 5)?

---

## Patch 4: Simplicity Budget amendment

Add to the "Simplicity Budget" section:

- Bounded context count (each is a team coordination cost, a deployment unit, and a
  failure mode — justify each against domain language boundaries)
- Coordination mechanism count (locks, transactions, consensus protocols, sagas —
  each must cite the aggregate invariant it protects)
- Required "simpler alternative" evaluation: for any architecture with > 3 bounded
  contexts or > 2 coordination mechanisms, evaluate whether a modular monolith with
  clear aggregate boundaries would suffice
