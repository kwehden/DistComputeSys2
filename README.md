# DistComputeSys2

A System2 overlay that injects distributed computing guidance into the System2 agent pipeline. Composed from three pillars:

1. **Domain-Driven Design (DDD)** — where to draw boundaries
2. **Pike's 5 Rules of Programming** — whether complexity is justified
3. **The Reactive Manifesto** — what properties boundaries must exhibit

## Problem

LLMs (including Claude) bias toward transactional, database-centric architectures. They default to shared databases as integration layers, synchronous RPC everywhere, and monolithic designs that fail to scale. Naive correction produces the opposite problem: over-engineered distributed systems for systems that don't need them.

This overlay corrects both failure modes by injecting guidance at every stage of the System2 pipeline — from context framing through requirements, architecture, implementation, and code review.

## Prerequisites

This is an **overlay** — it patches an existing [System2](https://github.com/jamesnordlund/System2) installation. System2 must be installed and initialized before this overlay can be applied.

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
```

Then initialize it in your target project:

```
/system2:init
```

This writes the orchestrator instructions to `CLAUDE.md`. After initialization you should have:

```
your-project/
├── CLAUDE.md                          # Orchestrator persona (System2 writes this)
└── plugin/                            # Or .claude/ depending on project config
    ├── .claude-plugin/plugin.json     # System2 plugin manifest
    ├── agents/                        # 13 agent persona .md files
    │   ├── design-architect.md
    │   ├── requirements-engineer.md
    │   ├── code-reviewer.md
    │   ├── executor.md
    │   ├── spec-coordinator.md
    │   └── ... (8 more)
    ├── hooks/                         # Validation scripts
    ├── allowlists/                    # Per-agent file restrictions
    └── skills/
        └── init/SKILL.md             # The /system2:init skill
```

**Verify System2 is working:** run `/system2:init --force` or check that agent files exist:
```bash
ls plugin/agents/design-architect.md  # or .claude/agents/design-architect.md
```

---

## Installation

### Option A: Plugin Install + Skill (Recommended)

Register this overlay as a Claude Code plugin alongside System2:

```
/plugin marketplace add kwehden/DistComputeSys2
/plugin install distcompute@kwehden-DistComputeSys2
```

This installs the overlay's skill and patch files. The plugin manifest (`plugin/.claude-plugin/plugin.json`) declares `"requires": ["system2"]` — Claude Code will warn if System2 is not present.

After installation, the `/distcompute:apply` skill becomes available. Run it in your project to apply the patches:

```
/distcompute:apply
```

**What the skill does:**
1. Locates your project's CLAUDE.md and agent persona files (checks both `plugin/agents/` and `.claude/agents/`).
2. Reads each patch from its bundled `plugin/patches/` directory.
3. For each patch, checks whether it's already been applied (sentinel text grep).
4. If not already present, injects the patch content at the documented insertion point.
5. Reports what was applied vs. skipped.

**Skill options:**
| Flag | Effect |
|------|--------|
| `--dry-run` | Show what would change without writing files |
| `--phase 1` | Apply only Phase 1 (orchestrator + design-architect) |
| `--phase 2` | Apply only Phase 2 (spec-coordinator + requirements-engineer) |
| `--phase 3` | Apply only Phase 3 (code-reviewer + executor) |

The skill is **idempotent** — running it multiple times won't duplicate content.

### Option B: Manual Patch Application

If you prefer manual control or want to customize before applying:

1. Clone or download this repo
2. Open each file in `plugin/patches/`
3. Each patch file documents:
   - **Target:** which System2 file to modify
   - **Action:** which section to find, where to insert
   - **Sentinel:** text to grep for to check if already applied
4. Copy the patch content into the target file at the specified location

### Option C: Clone and Reference

```bash
git clone https://github.com/kwehden/DistComputeSys2.git
```

Keep it as a reference alongside your projects. Apply patches manually or symlink the skill directory into your project's plugin tree.

---

### How the Skill Gets Registered

When Claude Code loads a plugin, it discovers skills by scanning the path declared in `plugin.json`:

```json
{
  "name": "distcompute",
  "skills": ["./skills/"]
}
```

Claude Code reads `plugin/skills/apply/SKILL.md`, which declares:

```yaml
---
name: apply
description: Apply the DistComputeSys2 overlay to the current project's System2 installation.
---
```

This registers as `/distcompute:apply` (plugin namespace `:` skill name). The skill's Markdown body contains the full procedure that Claude executes when invoked.

---

### Verifying Installation

After applying, confirm the overlay is active:

```bash
# Orchestrator patch applied?
grep "Domain-first boundaries" CLAUDE.md

# Design-architect patch applied?
grep "Bounded context identification" plugin/agents/design-architect.md

# Requirements-engineer patch applied?
grep "Domain-Driven Design guardrails" plugin/agents/requirements-engineer.md

# Code-reviewer patch applied?
grep "Domain model integrity" plugin/agents/code-reviewer.md

# Executor patch applied?
grep "Domain-driven implementation discipline" plugin/agents/executor.md

# Spec-coordinator patch applied?
grep "Domain Model Sketch" plugin/agents/spec-coordinator.md
```

All six should match. If any don't, re-run `/distcompute:apply` or check whether your agent files live in `.claude/agents/` instead of `plugin/agents/`.

---

### Graduated Adoption

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
