# Code Reviewer Patch — DDD + Distributed Review Criteria

**Target:** `plugin/agents/code-reviewer.md`
**Actions:**
1. Append to `Review checklist:`
2. Append to `Surface-area delta:`
3. Append to `Future-change probe:`

---

## Patch 1: Review checklist additions

- Domain model integrity (DDD):
  * Bounded context discipline: does code in one context directly reference internals
    of another context? Flag any import that crosses a context boundary without going
    through a published interface or Anti-Corruption Layer.
  * Aggregate boundary violations: does a transaction, database query, or state mutation
    span multiple aggregates? Flag cross-aggregate transactions. The fix is domain events
    or decomposition, not a bigger transaction.
  * Ubiquitous language drift: are the same domain terms used with different meanings
    in different files without explicit context boundaries? Flag polysemy without ACL.
  * Shared database between contexts: do multiple bounded contexts read/write the same
    tables? Flag — each context owns its own persistence.
  * Aggregate size: is the aggregate growing to encompass unrelated concerns for
    convenience? Flag aggregates that contain entities with independent lifecycles.

- Distributed systems hygiene (at context boundaries):
  * Data structure fitness: are data representations chosen so algorithms are self-evident?
    Flag clever algorithms compensating for poor data structure choices (Pike Rule 5).
  * Message-driven communication at boundaries: is inter-context communication async with
    explicit schemas? Flag synchronous RPC between bounded contexts without timeout/fallback.
  * Failure isolation: can one context's failure cascade into another? Flag missing bulkheads,
    missing timeouts, and shared-fate dependencies between contexts.
  * Idempotency at boundaries: are message handlers between contexts idempotent?
    Flag state mutations on retry/replay that produce different results.
  * Timeout discipline: does every call crossing a context boundary have an explicit timeout?
    Flag default or infinite timeouts on cross-boundary calls.
  * Back-pressure (when applicable): for boundaries with > 100 messages/second measured
    throughput, is flow control present? For lower-throughput boundaries, don't flag absence.
  * Location transparency: do cross-context calls hardcode endpoints without discovery
    or configuration? Flag in distributed deployments; acceptable in monolith deployments.

- Scale-appropriate complexity (Pike Rules 3-4):
  * Is the infrastructure proportionate to the actual load? Flag circuit breakers on
    boundaries called once per hour. Flag saga coordinators for 2-step operations that
    could be retried atomically. Flag event sourcing in contexts that only need CRUD.
  * Count the coordination mechanisms: locks, distributed transactions, consensus,
    saga orchestrators. Each must be justified by a specific aggregate invariant or
    measured contention. Flag any that exist "for safety" without a stated invariant.
  * Would a simpler approach work? If the diff introduces a new service, message broker,
    or coordination mechanism, ask: could this be a module boundary within an existing
    service? The burden is on complexity to justify itself.

---

## Patch 2: Surface-area delta additions

- Bounded context boundaries added/changed/removed
- Aggregate boundaries added/changed
- Cross-context coupling points added/removed
- Coordination mechanisms added/removed (locks, transactions, consensus, sagas)
- Anti-Corruption Layers added/modified

---

## Patch 3: Future-change probe additions

- Assess whether the diff respects or violates existing bounded context boundaries.
- Identify any new cross-context coupling that would make independent deployment harder.
- Assess whether new aggregates are sized correctly (not too large, not split across contexts).
- If a new bounded context is introduced: is it justified by language change in the domain,
  or is it infrastructure enthusiasm? Would a module boundary suffice?
