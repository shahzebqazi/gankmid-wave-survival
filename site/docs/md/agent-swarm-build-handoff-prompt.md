# Agent Swarm Build Handoff Prompt (Copy/Paste)

Use this prompt to run a planning+alignment swarm before implementation.

---

## Prompt

You are an **Agent Swarm Lead** preparing this project for implementation.  
Your job is to review all planning artifacts, resolve inconsistencies, update/create design docs as needed, and publish a **final implementation PRD** that an agent swarm can execute without ambiguity.

### Mission

Produce a build-ready planning package for a Godot wave-survival game prototype that includes:
- finalized PRD,
- updated design docs,
- unresolved-risk list,
- execution backlog suitable for parallel agent work.

Do not write gameplay code in this pass. This is planning/documentation finalization only.

---

## Required Inputs (read all)

- `docs/plans/2026-03-17-unified-prd-wave-survival.md`
- `docs/plans/game-design-document.md`
- `docs/plans/2026-03-17-game-design-expert-handoff.md`
- `docs/plans/2026-03-17-expert-critique.md`
- `docs/plans/2026-03-17-vertical-slice-roadmap.md`
- `docs/plans/2026-03-17-agent-swarm-handoff.md`
- `docs/plans/2026-03-17-menu-direction-brief.md`
- `docs/plans/2026-03-17-zombies-wave-map-mockups-design.md`
- `docs/plans/2026-03-17-external-expert-prompt.md`

Reference mockups:
- `design-docs/zombies-wave-map-mockups.html`
- `design-docs/menu-hud-mockups.html`

Asset constraints in scope:
- `KayKit_Skeletons_1.1_FREE`
- `KayKit_Adventurers_2.0_FREE`
- `KayKit_Character_Animations_1.1`
- `kenney_prototypeTextures`

---

## Hard Requirements

1. Preserve these baseline design decisions unless a contradiction is found:
   - Wave-based skeleton spawning from random entrances with deterministic rules.
   - Wave 1 has exactly 1 skeleton.
   - At wave `c * n` (constant `c`, entrances `n`), each door has 1 skeleton.
   - Skeleton AI: slow chase, short range, low damage.
   - Player: selectable adventurer character, one-hit-kill baseline vs skeletons, medium range/hitbox.
   - HUD includes in-game timer.

2. Resolve inconsistencies across docs (naming, formulas, constants, scope).

3. If a required design doc is missing, create it.

4. If an existing design doc is incomplete, update it in place.

5. Final output must be implementation-ready for a multi-agent build pass.

---

## Swarm Work Plan

Assign parallel agents to the following tracks, then merge results:

### Track A: PRD Consistency Review
- Validate all formulas, constants, and functional requirements.
- Check for conflicts between PRD and roadmap docs.
- Output: issues list with severity and exact proposed corrections.

### Track B: UX + Map Mockup Alignment
- Verify PRD matches menu/HUD and map mockups.
- Ensure map variants (1/2/3 entrances) and timer HUD are reflected in requirements/tests.
- Output: alignment report and doc edits.

### Track C: Combat + AI Design Spec Quality
- Validate skeleton AI behavior spec and player combat baseline for clarity/testability.
- Ensure one-hit kill and medium range/hitbox are measurable and not ambiguous.
- Output: finalized archetype requirements and tuning defaults.

### Track D: Execution Packaging
- Convert finalized requirements into an execution backlog grouped by independent workstreams.
- Mark parallelizable vs sequential tasks.
- Output: swarm-ready build plan with handoff boundaries.

---

## Deliverables (must produce all)

1. **Final PRD**
   - Path: `docs/plans/final-prd-wave-survival.md`
   - Must supersede previous drafts and be internally consistent.

2. **Design Doc Changelog**
   - Path: `docs/plans/final-design-doc-changelog.md`
   - Include what was changed, why, and which source doc was affected.

3. **Swarm Execution Backlog**
   - Path: `docs/plans/final-swarm-execution-backlog.md`
   - Include:
     - workstream ID
     - objective
     - required input files
     - output files
     - dependencies
     - acceptance tests

4. **Open Risks and Decisions Log**
   - Path: `docs/plans/final-open-risks-and-decisions.md`
   - Any unresolved items must include owner + unblock criteria.

---

## Quality Gates Before Completion

Do not mark complete until all checks pass:

- No conflicting formulas for spawn count/timing.
- No missing requirement for timer HUD.
- Character selection and player combat baseline explicitly specified.
- Skeleton AI state behavior is testable with deterministic criteria.
- All constants used in tests exist in final PRD.
- Vertical slice order and go/no-go gates are present.
- Final PRD can be handed to an implementation swarm with zero additional clarification.

---

## Required Response Format

Return your response in this structure:

1. **Summary of what changed** (max 10 bullets)
2. **Conflicts found and resolved** (table)
3. **Files created/updated**
4. **Final PRD readiness verdict**: `READY` or `NOT READY`
5. **If NOT READY**: exact blockers

Include explicit references to output files created.

---

## Definition of Done

Planning is complete only when:
- `docs/plans/final-prd-wave-survival.md` exists and is consistent,
- supporting design docs are updated/created,
- backlog is split for swarm execution,
- unresolved risks are documented with owners and unblock paths,
- readiness verdict is `READY`.
