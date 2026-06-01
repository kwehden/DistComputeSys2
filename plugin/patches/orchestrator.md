# Orchestrator Patch — Three-Pillar Distributed Systems Stance

**Target:** Project CLAUDE.md → `## Operating principles` section
**Action:** Append after existing bullets

---

- Domain-first boundaries. Draw system boundaries where the domain model's language changes
  (DDD Bounded Contexts), not where infrastructure naturally splits. Infrastructure topology
  follows domain topology, not the reverse. When a subagent proposes component boundaries,
  require that each boundary corresponds to a coherent domain model with its own ubiquitous language.
- Aggregates as consistency scope. The unit of transactional consistency is the aggregate,
  not "the database" or "the system." Identify aggregates early (the smallest cluster of
  domain objects that must be consistent with each other). Inter-aggregate communication
  is eventually consistent by default. Only claim stronger consistency (linearizability,
  serializability) when a specific user-observable correctness requirement demands it,
  and scope it to the narrowest possible boundary.
- Distributed-when-justified. Default to message-driven, asynchronous communication
  *between bounded contexts*. Within a single context, simpler patterns (direct calls,
  shared memory, local transactions) are acceptable when the context is small enough.
  The question is not "is this distributed?" but "where are the context boundaries, and
  what properties must the boundaries exhibit?"
- Data structures dominate. Choose data representations that make algorithms self-evident
  (Pike Rule 5). Prefer event logs, immutable streams, and aggregate state over mutable
  shared tables. "Write stupid code that uses smart objects."
- Simplicity scales. Prefer simple algorithms with predictable performance over sophisticated
  ones with large constant factors (Pike Rules 3-4). Brute force with good data structures
  beats clever algorithms with poor ones. More components is not always better than fewer
  well-structured ones. Justify every boundary, coordination mechanism, and infrastructure
  component with a domain or measured-scale rationale.
- Reactive properties at boundaries. Communication across bounded context boundaries must be:
  * Message-Driven: asynchronous, with explicit schemas and back-pressure.
  * Resilient: failure in one context doesn't cascade into others.
  * Responsive: latency bounded at each boundary crossing.
  * Elastic: no single context becomes a bottleneck for the whole system.
  These properties apply *between* contexts. Within a single small context, they are optional
  overhead unless scale demands them.
- Anti-transactional bias (scoped). When a subagent proposes:
  (a) a shared database as the integration layer between bounded contexts,
  (b) synchronous coupling between contexts that don't share a consistency boundary,
  (c) distributed transactions spanning multiple aggregates, or
  (d) a single write-path bottleneck —
  require explicit justification. But: within a single bounded context, a local database
  with ACID transactions may be the Pike Rule 4 answer (simplest thing that works).
  The bias correction applies at context *boundaries*, not universally.
- Scale-appropriate complexity (Pike Rule 3). Before introducing infrastructure (message
  brokers, saga coordinators, event sourcing, CQRS, circuit breakers), ask: "What is the
  actual event rate? The actual consistency window? The actual failure frequency?"
  If the answer is "dozens per day" not "thousands per second," simpler patterns may be
  correct. Measure first. Architect second.
