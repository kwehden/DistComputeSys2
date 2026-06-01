# Spec Coordinator Patch — Domain Model & Scale Context

**Target:** `plugin/agents/spec-coordinator.md`
**Actions:**
1. Append step 5 after `Before writing:` step 4
2. Add sections to `spec/context.md must include these sections` list
3. Append to `Style requirements:`
4. Add to Constraints & Invariants guidance

---

## Patch 1: Before-writing step 5

5) Assess the domain and scale characteristics:
   - What are the distinct "languages" in this problem? (e.g., do stakeholders use the
     same terms with different meanings?)
   - What are the natural model boundaries? (Where do stakeholders stop agreeing on
     definitions?)
   - What is the expected scale: events/day, users, concurrent operations?
   - What are the latency expectations at user-facing boundaries?
   - What failure modes must the system tolerate?
   - How many teams will maintain this? (Conway's Law: team boundaries constrain
     context boundaries.)
   If the system appears to have multiple domain models or multiple stakeholder languages,
   capture this in the Domain Model Sketch. Do not resolve it — surface it for requirements
   and design to address.

---

## Patch 2: Additional required sections for spec/context.md

- Domain Model Sketch (preliminary bounded contexts if apparent from problem structure,
  key aggregate candidates, ubiquitous language terms that may mean different things to
  different stakeholders, and known integration boundaries with external systems.
  This is a sketch — design-architect refines it. But the spec-coordinator surfaces
  early language conflicts and model boundaries before requirements are written.)
- Communication & Scale Intent (default communication model — sync vs. async,
  expected scale trajectory with actual numbers, known failure scenarios the system
  must tolerate, and explicit statement of the system's operational scope: is this a
  single-team research tool, a multi-team platform, or a production service with SLAs?)

---

## Patch 3: Style requirements additions

- When different stakeholders use the same term differently, call out the polysemy in the
  Glossary and note that this likely signals a bounded context boundary. Don't force a
  single definition — capture both and mark the boundary.
- State scale expectations numerically: concurrent users, events/second, data growth rate.
  "Scalable" is not a specification. Include the team size maintaining the system —
  this constrains architectural complexity (you can't operate more services than you have
  people to maintain them).
- When describing system interactions, prefer event/outcome language over
  request/response language unless the interaction is genuinely synchronous.
  But: if the system is small (single team, < 100 events/day), don't force distributed
  framing onto what might be a well-structured monolith.

---

## Patch 4: Constraints & Invariants template addition

Domain and distribution constraints (include when applicable):
- Domain model: [single unified model | multiple models with boundaries at <X, Y> |
  unclear — needs design-phase resolution]
- Operational scope: [single-team research tool | multi-team platform | production service with SLAs]
- Communication default between boundaries: [async message-driven | sync RPC | TBD based on scale]
- Consistency model: [per-aggregate immediate, inter-aggregate eventual with <N>ms budget |
  TBD — design-phase decision]
- Scale reality: [<N> events/day current, <M> projected 12-month | actual measurements available: yes/no]
- Team capacity: [N people maintaining this system — constrains operational complexity]
