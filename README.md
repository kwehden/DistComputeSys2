# DistComputeSys2

A System2 overlay that injects distributed computing guidance into the System2 agent pipeline. Composed from three pillars:

1. **Domain-Driven Design (DDD)** — where to draw boundaries
2. **Pike's 5 Rules of Programming** — whether complexity is justified
3. **The Reactive Manifesto** — what properties boundaries must exhibit

## Problem

LLMs (including Claude) bias toward transactional, database-centric architectures. They default to shared databases as integration layers, synchronous RPC everywhere, and monolithic designs that fail to scale. Naive correction produces the opposite problem: over-engineered distributed systems for systems that don't need them.

This overlay corrects both failure modes by injecting guidance at every stage of the System2 pipeline — from context framing through requirements, architecture, implementation, and code review.

## Installation

This overlay is designed to be applied on top of an existing System2 installation via the provided skill:

```
/distcompute:apply
```

Or manually: copy the patch files from `plugin/patches/` into your project's System2 agent definitions and CLAUDE.md at the documented insertion points.

## How It Works

The overlay modifies System2 agent personas to include distributed systems guidance at their natural decision points:

| Agent | What Changes |
|-------|-------------|
| **Orchestrator** (CLAUDE.md) | Adds three-pillar operating principles |
| **spec-coordinator** | Adds domain model sketch + scale intent to context.md |
| **requirements-engineer** | Adds aggregate-scoped consistency, anti-proscriptive rules |
| **design-architect** | Adds bounded context identification, context maps, scale-gated reactive properties |
| **executor** | Adds aggregate discipline, boundary-scoped async |
| **code-reviewer** | Adds DDD violation detection + scale-proportionality checks |

## Three Pillars Composition

```
DDD says:     "Here is where the boundaries are"
              (domain language change, aggregate lifecycle)

Pike says:    "Here is whether each boundary is justified"
              (measured scale, actual complexity needs)

Reactive says: "Here is what the justified boundaries must do"
              (message-driven, resilient, elastic, responsive)
```

- Without DDD: boundaries are drawn by infrastructure enthusiasm
- Without Pike: every boundary gets full distributed treatment regardless of load
- Without Reactive: justified boundaries lack specification of runtime properties

## Source Principles

### Pike's 5 Rules
1. Bottlenecks occur in surprising places — don't optimize without measurement
2. Measure before tuning
3. Fancy algorithms are slow for small n
4. Use simple algorithms and simple data structures
5. Data dominates — right data structures make algorithms self-evident

### Reactive Manifesto (v2.0)
- **Responsive**: timely responses with reliable upper bounds
- **Resilient**: responsive in failure via containment, isolation, delegation
- **Elastic**: responsive under load via sharding, no central bottlenecks
- **Message-Driven**: async message-passing, back-pressure, flow control

### Domain-Driven Design
- **Bounded Contexts**: divide where ubiquitous language changes
- **Aggregates**: smallest consistency unit, transactions don't cross boundaries
- **Context Maps**: relationship patterns between contexts drive integration
- **Anti-Corruption Layers**: translate at alien model boundaries

## Anti-Bias Self-Check

| Failure Mode | What It Looks Like | Correction |
|---|---|---|
| Transactional bias | Shared DB, sync RPC everywhere, no failure isolation | Reactive + DDD context separation |
| Distributed enthusiasm | 9 services for 50 events/day, circuit breakers on hourly calls | Pike Rules 3-4 + DDD aggregate sizing |

## When NOT to Use

- Single-process CLI tools or libraries with no integration boundaries
- CRUD web apps with a single database and no external system integration
- Teams < 2 people who will never operate more than one deployment unit

## License

MIT
