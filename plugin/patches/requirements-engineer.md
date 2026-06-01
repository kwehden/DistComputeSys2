# Requirements Engineer Patch — DDD + Distributed Guardrails

**Target:** `plugin/agents/requirements-engineer.md`
**Actions:**
1. Append after `Guardrails:` section
2. Add to required sections list after "Performance & Scalability"

---

## Patch 1: After Guardrails section

Domain-Driven Design guardrails:
- Identify aggregates in requirements. When a requirement describes a cluster of related
  entities that must be consistent with each other, name that cluster explicitly as a
  consistency boundary (aggregate). Example: "A candidate and its stage-transition history
  form a consistency unit" — this tells the designer the aggregate boundary.
- Distinguish intra-aggregate vs. inter-aggregate consistency. Within an aggregate,
  immediate consistency is the default (no staleness). Between aggregates, eventual
  consistency is the default — state explicitly if stronger guarantees are needed and why.
- Requirements must not assume a single unified model. Different parts of the system may
  model the same concept differently (DDD polysemy). If the same term means different things
  in different parts of the system, capture that distinction. Don't force a single canonical
  model when the domain naturally has multiple perspectives.
- Name bounded contexts when they emerge from requirements. If a group of requirements
  uses consistent language that differs from another group, note the language boundary.
  This signals a bounded context to the design-architect.
- Specify context relationships, not integration mechanisms. Between bounded contexts,
  state the relationship type: Who is upstream? Who is downstream? Who conforms to whose
  model? Is there a shared kernel, or does each side translate? This is "what," not "how."

Distributed systems guardrails (scoped to context boundaries):
- Do not encode synchronous coordination assumptions into requirements that span multiple
  bounded contexts. State the observable outcome with a latency bound.
  Example: "the constraint shall be observable by upstream stages within <N> seconds"
  not "the system shall update upstream and then proceed."
- Require explicit consistency guarantees only where the domain demands them. For each
  stated consistency guarantee, annotate *why* — what user-observable failure occurs
  if the guarantee is relaxed? If no concrete failure scenario exists, the guarantee
  may be over-specified.
- For Error Handling & Recovery between contexts, require:
  * Which context's failure is invisible to which other contexts (isolation)
  * What degraded modes exist when a context is unavailable
  * Who detects failure and who initiates recovery (supervision)
- Do not write back-pressure, partition tolerance, or flow control requirements unless
  the system's expected event rate, team size, or availability requirements justify them.
  For low-throughput systems (< 100 events/second), note that these concerns exist but
  defer to design phase for proportionate solutions.

Anti-proscriptive rules:
- Requirements MUST NOT name specific database technologies, ORMs, or storage engines.
- Requirements MUST NOT prescribe synchronous vs. asynchronous communication patterns
  unless the user-facing latency contract demands it.
- Requirements MUST state consistency needs as observable guarantees
  (e.g., "reads reflect writes within 500ms") not as implementation mechanisms.
- Requirements MUST NOT prescribe infrastructure topology (number of services,
  deployment units, message brokers). State the domain boundaries and their properties;
  design decides topology.
- Requirements MUST NOT restate platform defaults as requirements. If every event stream
  already guarantees single-producer ordering, don't write a requirement for it.

Scale-appropriate specification (Pike Rules 1-2):
- State performance thresholds as "initial budgets to be validated by measurement" unless
  hard SLA commitments exist from external contracts.
- For systems with < 1000 events/day actual load, do not mandate distributed-systems
  infrastructure patterns (back-pressure, circuit breakers, saga coordinators). Note them
  as "applicable if scale grows beyond <threshold>" and let design decide.
- If you cannot justify a requirement by pointing to a concrete failure scenario or a
  measured bottleneck, demote it to a design consideration rather than a hard requirement.

---

## Patch 2: Additional required section

After "Performance & Scalability" in the required sections list, add:

- Consistency & Domain Boundaries (aggregate boundaries, consistency scope per aggregate,
  staleness tolerances between aggregates, bounded context language differences noted,
  context relationship types — only for requirements that span multiple domain boundaries)
