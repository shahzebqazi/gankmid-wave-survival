# Agent Swarm Handoff: Game Design Expert Deliverables

**Date:** 2026-03-17  
**Project:** Asset-driven Godot game (pre-production)  
**Owner intent:** Execute a parallel expert-style pass and produce actionable planning outputs before full gameplay implementation.

---

## Mission

Deliver a validated pre-production package that includes:

1. Architecture critique (boundaries, coupling risks, minimum viable structure).
2. Vertical-slice roadmap (Slices 0-3) with measurable gates.
3. External expert outreach prompt in copy/paste form.

---

## Mandatory Inputs (Read First)

## Local docs
- `docs/plans/2026-03-17-game-design-expert-handoff.md`
- `docs/plans/game-design-document.md`
- `manifest/README.md`

## Required web sources
- https://kaylousberg.itch.io/kaykit-character-animations
- https://kaylousberg.itch.io/kaykit-skeleton-pack
- https://godotengine.org/asset-library/asset/2566
- https://github.com/KayKit-Game-Assets/KayKit-Character-Pack-Skeletons-1.0
- https://www.kenney.nl/assets/prototype-textures
- https://kenney-assets.itch.io/prototype-textures
- https://godotengine.org/asset-library/asset/780
- https://datagoblin.itch.io/monogram
- https://arks.itch.io/ps4-buttons

For each source, extract:
- License/attribution requirements
- Engine-format implications for Godot import flow
- Risks/caveats relevant to slice scope, tooling, or tests

Do not finalize outputs until this ingestion is complete.

---

## Confirmed Findings Baseline

- The map/entity boundary is sound for small-team scope if kept strict:
  - Map = static authored data (geometry, bounds, spawns, triggers)
  - Entities/systems = runtime state/behavior
- Remove ambiguity around "map-owned interactables"; model doors/levers/chests as entities with map placement references.
- Add a thin per-map director for trigger orchestration only; prevent it from becoming a gameplay/system God object.
- UI-first ordering is valid, but Slice 0 must include a dummy in-game scene integration to avoid menu/runtime divergence.
- Deterministic automation should start with menu flow, trigger reliability, and save/load sanity; feel/readability remains manual early.
- Asset constraints:
  - Most packs are CC0.
  - PS4 Buttons is CC BY 4.0 and must be credited in release-facing credits/docs.
  - KayKit skeleton page is legacy/remaster-noted; pin package version explicitly.

---

## Swarm Execution Plan

## Agent A: Architecture and coupling
**Objective:** Validate and tighten domain boundaries.  
**Produce:** Concise architecture memo with top coupling risks and mitigations.  
**Output file:** `docs/plans/2026-03-17-expert-critique.md`

## Agent B: Tooling and test strategy
**Objective:** Define practical P0/P1/P2 tooling and what to automate first.  
**Produce:** Priority checklist and measurable quality gates per slice.  
**Output file:** `docs/plans/2026-03-17-expert-critique.md` (integrated section)

## Agent C: Vertical-slice sequencing and gates
**Objective:** Build all-slice roadmap with dependency flow and go/no-go criteria.  
**Produce:** Slice 0-3 roadmap, risk mitigations, and explicit pass/fail thresholds.  
**Output file:** `docs/plans/2026-03-17-vertical-slice-roadmap.md`

## Agent D: External expert packet
**Objective:** Produce a polished prompt for external review.  
**Produce:** Copy/paste expert request with strict response structure.  
**Output file:** `docs/plans/2026-03-17-external-expert-prompt.md`

## Agent E: Menu direction alignment (user-gated)
**Objective:** Lock menu implementation direction with the user before UI/menu build decisions are finalized.  
**Produce:** A short "Menu Direction Brief" capturing chosen UX direction and constraints.  
**Output file:** `docs/plans/2026-03-17-menu-direction-brief.md`

**Mandatory user prompt before implementation:**
- This agent must stop and ask the user for direction on menu design before any menu implementation details are finalized.
- At minimum, collect:
  - Visual style direction (minimal/prototype vs stylized target)
  - Navigation model preferences (controller-first behavior, wrap rules, confirm/cancel mapping)
  - Information architecture (screen hierarchy and priority actions)
  - Accessibility preferences (font scale, contrast, motion, input forgiveness)
  - Persistence expectations (which options must persist immediately)
- If direction is ambiguous, the agent must ask follow-up questions and wait for user confirmation.

---

## Acceptance Criteria

- Architecture boundaries are explicit, non-overlapping, and solo-team practical.
- Coupling risks are ranked and mapped to concrete mitigations.
- Tooling is prioritized P0/P1/P2 with deterministic, testable thresholds.
- Slice roadmap includes clear scope, out-of-scope, tests, and go/no-go gates.
- External expert prompt is immediately usable and requests structured output.
- Menu implementation direction is explicitly user-confirmed and documented before UI/menu decisions proceed.
- Licensing obligations are reflected, especially PS4 Buttons attribution.

---

## Existing Deliverables (Current State)

- `docs/plans/2026-03-17-expert-critique.md`
- `docs/plans/2026-03-17-vertical-slice-roadmap.md`
- `docs/plans/2026-03-17-external-expert-prompt.md`

Use these as canonical outputs unless superseded by a new review cycle.

