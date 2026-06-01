# DistComputeSys2

A [System2](https://github.com/jamesnordlund/System2) overlay that injects distributed computing guidance into the System2 agent pipeline. Composed from three pillars:

1. **Domain-Driven Design (DDD)** — where to draw boundaries
2. **Pike's 5 Rules of Programming** — whether complexity is justified
3. **The Reactive Manifesto** — what properties boundaries must exhibit

## Problem

LLMs (including Claude) bias toward transactional, database-centric architectures. They default to shared databases as integration layers, synchronous RPC everywhere, and monolithic designs that fail to scale. Naive correction produces the opposite problem: over-engineered distributed systems for systems that don't need them.

This overlay corrects both failure modes by injecting guidance at every stage of the System2 pipeline — from context framing through requirements, architecture, implementation, and code review.

## Prerequisites

This is an **overlay** — it patches an existing [System2](https://github.com/jamesnordlund/System2) installation. System2 must be installed first.

### What is System2?

[System2](https://github.com/jamesnordlund/System2) is a Claude Code plugin that provides a structured multi-agent engineering workflow with quality gates:

```
Scope → Context → Requirements → Design → Tasks → Implementation → Verification → Ship
```

It installs 13 specialized subagents (spec-coordinator, requirements-engineer, design-architect, executor, code-reviewer, etc.), validation hooks, file-access allowlists, and an orchestrator persona (CLAUDE.md). Each agent is a Markdown file with YAML frontmatter defining its tools, constraints, and system prompt.

### Install System2 (if not already installed)

```
/plugin marketplace add jamesnordlund/System2
/plugin install system2@jamesnordlund-system2
/reload-plugins
```

Then initialize it in your target project:

```
/system2:init
```

---

## Installation

### Step 1: Add the marketplace

```
/plugin marketplace add kwehden/DistComputeSys2
```

This registers the GitHub repo as a plugin source. Claude Code clones it and reads `.claude-plugin/marketplace.json` to discover available plugins.

### Step 2: Install the plugin

```
/plugin install distcompute@kwehden-DistComputeSys2
```

This installs the skill and patch files to `~/.claude/plugins/cache/kwehden-DistComputeSys2/distcompute/1.0.0/`.

### Step 3: Reload plugins

```
/reload-plugins
```

**This step is required.** Claude Code caches the plugin registry at session start. After installing, you must reload for the new skill to become available. You'll see output like:

```
Reloaded: 4 plugins · 0 skills · 21 agents · 0 hooks · 1 plugin MCP server · 0 plugin LSP servers
```

### Step 4: Apply the overlay

Navigate to your System2-enabled project and run:

```
/distcompute:apply
```

The skill locates your System2 agent files (checks the global plugin cache, `plugin/agents/`, and `.claude/agents/`), reads each patch, checks for sentinel text to avoid duplication, and injects the content at the documented insertion points.

**Options:**

| Flag | Effect |
|------|--------|
| `--dry-run` | Show what would change without writing files |
| `--phase 1` | Apply only Phase 1 (orchestrator + design-architect) |
| `--phase 2` | Apply only Phase 2 (spec-coordinator + requirements-engineer) |
| `--phase 3` | Apply only Phase 3 (code-reviewer + executor) |

The skill is **idempotent** — running it again skips already-applied patches.

### What gets patched

| Phase | Target | What's Added |
|-------|--------|-------------|
| 1a | CLAUDE.md (orchestrator) | Three-pillar operating principles |
| 1b | design-architect.md | DDD constraints, bounded context sections, alternatives template, simplicity budget |
| 2a | spec-coordinator.md | Domain/scale assessment step, new context.md sections, style requirements |
| 2b | requirements-engineer.md | DDD + distributed guardrails, consistency & domain boundaries section |
| 3a | code-reviewer.md | DDD checklist, surface-area delta items, future-change probes |
| 3b | executor.md | Domain-driven implementation discipline section |

### Verifying Installation

```bash
grep "Domain-first boundaries" CLAUDE.md
grep "Bounded context identification" plugin/agents/design-architect.md
grep "Domain-Driven Design guardrails" plugin/agents/requirements-engineer.md
grep "Domain model integrity" plugin/agents/code-reviewer.md
grep "Domain-driven implementation discipline" plugin/agents/executor.md
grep "Domain Model Sketch" plugin/agents/spec-coordinator.md
```

If agents live in the global plugin cache (no local `plugin/agents/`), check:
```bash
grep "Bounded context identification" ~/.claude/plugins/marketplaces/system2-marketplace/plugin/agents/design-architect.md
```

---

## Graduated Adoption

You don't have to apply everything at once:

| Phase | Agents Patched | What It Catches | When to Add |
|-------|---------------|-----------------|-------------|
| **Phase 1** | Orchestrator + design-architect | Architecture-level bias | Start here — most impact per patch |
| **Phase 2** | spec-coordinator + requirements-engineer | Transactional requirements entering the pipeline | After Phase 1 design outputs improve |
| **Phase 3** | code-reviewer + executor | Implementation violations + review enforcement | When you want full pipeline enforcement |

```
/distcompute:apply --phase 1   # Start with architecture
# ... use for a few design cycles, validate improvement ...
/distcompute:apply --phase 2   # Add upstream framing
# ... validate requirements quality ...
/distcompute:apply --phase 3   # Add enforcement
```

---

## How It Works

### Plugin Structure

```
DistComputeSys2/
├── .claude-plugin/
│   └── marketplace.json        # Marketplace discovery (required for /plugin marketplace add)
├── plugin/
│   ├── .claude-plugin/
│   │   └── plugin.json         # Plugin manifest (declares name, skills path)
│   ├── patches/                # The injectable content
│   │   ├── orchestrator.md
│   │   ├── spec-coordinator.md
│   │   ├── requirements-engineer.md
│   │   ├── design-architect.md
│   │   ├── code-reviewer.md
│   │   └── executor.md
│   └── skills/
│       └── apply/
│           └── SKILL.md        # The /distcompute:apply skill definition
├── docs/
│   ├── sources.md              # Full source material (Pike, Reactive, DDD)
│   └── anti-bias-analysis.md   # Both failure modes + self-test indicators
├── README.md
└── LICENSE
```

### How Skill Registration Works

1. `/plugin marketplace add kwehden/DistComputeSys2` clones the repo and reads `.claude-plugin/marketplace.json`, which declares a plugin named `distcompute` at source path `./plugin`.

2. `/plugin install distcompute@kwehden-DistComputeSys2` copies `./plugin` to the cache and registers it in `~/.claude/plugins/installed_plugins.json`.

3. `/reload-plugins` re-scans installed plugins. Claude Code reads `plugin.json` which declares `"skills": ["./skills/"]`, finds `skills/apply/SKILL.md`, and registers it as `/distcompute:apply`.

4. When you invoke `/distcompute:apply`, Claude Code loads the SKILL.md and executes its instructions.

---

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

## Author

Karl Wehden — Karl@Wehden.com

## License

MIT
