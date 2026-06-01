# Executor Patch — DDD + Distributed Implementation Discipline

**Target:** `plugin/agents/executor.md`
**Action:** Insert new section after `## Anti-additive bias` (before `## Slop catalog`)

---

## Domain-driven implementation discipline

Bounded context enforcement:
- Do not import or reference types from another bounded context directly. If the task
  requires data from another context, use the published interface (events, API, ACL).
  If no interface exists, stop and request one from the design — do not create ad-hoc coupling.
- Each bounded context owns its own persistence. Do not write queries that join across
  context boundaries. If you need data from two contexts, read from each independently
  and compose in the application layer.
- When implementing an Anti-Corruption Layer (normalization, protocol translation), keep
  it thin and stateless. Its job is translation, not business logic.

Aggregate discipline:
- Mutations within an aggregate are immediately consistent (local transaction, in-memory
  state, whatever is simplest). Do not introduce eventual consistency *within* an aggregate
  unless the task plan explicitly requires it.
- Mutations *between* aggregates are eventually consistent. Do not write code that assumes
  another aggregate's state is current. Read the last-known state and handle staleness.
- If you find yourself needing a transaction that spans two aggregates, stop. Either the
  aggregate boundaries are wrong (raise to orchestrator) or the operation needs decomposition
  into domain events with compensating actions.
- Keep aggregates small. If an aggregate is growing beyond 5-7 closely related entities,
  ask whether some of those entities have independent lifecycles and should be their own aggregate.

Data-structure-first implementation (Pike Rule 5):
- Before writing algorithms, verify the data structures are right. If you find yourself
  writing complex logic to compensate for data shape, stop and reconsider the data structure.
- Prefer immutable data passed via messages between contexts over mutable shared state.
- When choosing between a clever algorithm and a simpler one with better data structures,
  choose the latter (Pike Rules 3-4).

Simplicity under concurrency (Pike Rules 3-4):
- Do not introduce coordination mechanisms (locks, semaphores, distributed transactions)
  unless the task plan explicitly requires them and a specific aggregate invariant is at risk.
- Prefer share-nothing designs where each aggregate processes its own commands independently.
- "When in doubt, use brute force" — replication of simple processing often beats
  sophisticated partitioning.

Communication at context boundaries:
- When implementing inter-context communication:
  * Use the published message schema. Do not add fields ad-hoc.
  * Make message handlers idempotent — assume at-least-once delivery.
  * Include correlation IDs for tracing across context boundaries.
  * Include explicit timeouts on all outbound calls that cross a context boundary.
- When implementing failure handling at boundaries:
  * Contain failures at context boundaries — do not let exceptions propagate across
    boundaries without transformation into domain-appropriate responses.
  * Prefer supervised restart over complex recovery logic within the failing component.
  * Distinguish between transient failures (retry) and semantic failures (record and proceed).

Scale-proportionate implementation:
- Do not add infrastructure (circuit breakers, retry with jitter, connection pools, bulkheads)
  for operations that happen fewer than 100 times per day unless the task plan explicitly
  requires it. Simple try/catch with logging is sufficient for low-frequency operations.
- If the task involves performance work, instrument first, measure, then optimize the measured
  bottleneck — not the suspected one (Pike Rules 1-2).
- If you cannot measure (no profiling/tracing available), note this as a risk rather than
  guessing at optimizations.
