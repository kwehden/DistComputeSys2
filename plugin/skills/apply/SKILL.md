---
name: apply
description: Apply the DistComputeSys2 overlay to the current project's System2 installation. Patches CLAUDE.md and agent persona files with distributed systems guidance.
argument-hint: "[--dry-run] [--phase 1|2|3]"
---

# /distcompute:apply — Install Distributed Computing Overlay

You are executing the /distcompute:apply skill. Follow these steps exactly.

## Arguments

- `--dry-run`: If passed, show what would be changed without modifying files.
- `--phase <N>`: Apply only a specific phase (1=architecture, 2=upstream, 3=enforcement). Default: all phases.

## Prerequisites Check

1. Verify `CLAUDE.md` exists at the project root and contains `## Operating principles`.
2. Verify the System2 plugin is installed by checking for agent persona files. Look for:
   - `plugin/agents/spec-coordinator.md` OR `.claude/agents/spec-coordinator.md`
   - `plugin/agents/design-architect.md` OR `.claude/agents/design-architect.md`
   If neither location has agent files, inform the user: "System2 agent personas not found. Run `/system2:init` first, or specify the agent directory."
3. Determine the agent directory (whichever of the above paths exists).

## Phase 1: Architecture (Orchestrator + Design Architect)

### 1a. Patch CLAUDE.md

Read the orchestrator patch from `plugin/patches/orchestrator.md` in the DistComputeSys2 plugin directory.

Find the `## Operating principles` section in CLAUDE.md. Locate the last bullet point in that section.

If the section already contains "Domain-first boundaries" — skip (already applied).

Otherwise, append the patch content after the last existing bullet.

### 1b. Patch design-architect.md

Read the design-architect patch from `plugin/patches/design-architect.md`.

Apply Patch 1: Find the `Design constraints:` section. Locate the end of the existing content (after the agentic components block or the last existing constraint). If the section already contains "Bounded context identification" — skip. Otherwise, append Patch 1 content.

Apply Patch 2: Find the required sections list for spec/design.md. After "Concurrency, Ordering, and Consistency", insert the two new section requirements. If "Bounded Contexts & Context Map" already exists — skip.

Apply Patch 3: Find "Alternatives Considered" in the required sections list. Append the domain fit / scale profile / simplicity assessment template. If "Domain fit:" already exists — skip.

Apply Patch 4: Find "Simplicity Budget" in the required sections list. Append the bounded context count and coordination mechanism items. If "Bounded context count" already exists — skip.

## Phase 2: Upstream (Spec Coordinator + Requirements Engineer)

### 2a. Patch spec-coordinator.md

Read the spec-coordinator patch from `plugin/patches/spec-coordinator.md`.

Apply Patch 1: Find `Before writing:` numbered list. After step 4, insert step 5 (renumber if needed). If step 5 already contains "domain and scale characteristics" — skip.

Apply Patch 2: Find `spec/context.md must include these sections:` list. After "Open Questions", add "Domain Model Sketch" and "Communication & Scale Intent". If "Domain Model Sketch" already exists — skip.

Apply Patch 3: Find `Style requirements:` section. Append the three new bullets. If "polysemy" is already mentioned — skip.

Apply Patch 4: Add the domain/distribution constraints template to the Constraints & Invariants guidance. If "Domain model:" template already exists — skip.

### 2b. Patch requirements-engineer.md

Read the requirements-engineer patch from `plugin/patches/requirements-engineer.md`.

Apply Patch 1: Find the `Guardrails:` section. Append after the last existing guardrail bullet. If "Domain-Driven Design guardrails:" already exists — skip.

Apply Patch 2: Find the required sections list for spec/requirements.md. After "Performance & Scalability", insert the new section. If "Consistency & Domain Boundaries" already exists — skip.

## Phase 3: Enforcement (Code Reviewer + Executor)

### 3a. Patch code-reviewer.md

Read the code-reviewer patch from `plugin/patches/code-reviewer.md`.

Apply Patch 1: Find `Review checklist:` section. Append after the last existing checklist item. If "Domain model integrity (DDD):" already exists — skip.

Apply Patch 2: Find `Surface-area delta:` section. Append the new items. If "Bounded context boundaries" already exists — skip.

Apply Patch 3: Find `Future-change probe:` section. Append the new items. If "bounded context boundaries" is already mentioned — skip.

### 3b. Patch executor.md

Read the executor patch from `plugin/patches/executor.md`.

Find `## Anti-additive bias` section. Insert the new section AFTER it (before `## Slop catalog` if that exists). If `## Domain-driven implementation discipline` already exists — skip.

## Completion

After all patches are applied (or skipped):

1. Report which patches were applied vs. skipped (already present).
2. If `--dry-run` was passed, report what WOULD have been changed without writing files.
3. Suggest the user run a smoke test: "Try running a small design task through the pipeline to verify the extensions are active."

## Idempotency

This skill is idempotent. Running it multiple times on the same project will not duplicate content — each patch checks for sentinel text before applying.
