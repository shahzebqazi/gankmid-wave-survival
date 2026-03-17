# Expert Critique: Architecture, Tooling, and Slice Sequencing

**Date:** 2026-03-17  
**Scope:** Review of pre-production architecture and execution strategy for an asset-driven Godot project.

---

## 1) Verdict on current architecture

Your baseline is strong for a solo/small-team project. The map/entity boundary is directionally correct and supports vertical slicing well.

- **Keep:** map as static authored world data (geometry, bounds, spawn points, trigger regions).
- **Keep:** entities/systems as runtime behavior/state owners.
- **Adjust now:** remove ambiguity around "optional map-owned interactables." Treat all interactables (doors, levers, chests) as entities with state, and let map data only reference placement/IDs.
- **Add:** a thin per-map director that wires trigger events to actions without absorbing gameplay system logic.

This gives you enough structure to scale to multiple maps without overbuilding.

---

## 2) Highest-risk coupling points (and mitigation)

### Risk A: Trigger contract drift
Map-authored trigger IDs can silently diverge from runtime handlers.

**Mitigation**
- Define one trigger schema file (ID naming rules + enum-like list).
- Validate trigger IDs at map load and log unknown/missing handlers as errors.
- Add a smoke test that runs the critical trigger path per slice.

### Risk B: "Level script" becoming a God object
Map directors can become dumping grounds for AI, combat, progression, and UI.

**Mitigation**
- Restrict map director responsibilities to orchestration only:
  - trigger -> event routing
  - level-local sequencing
  - map transition decisions
- Keep combat, inventory, save, UI state in dedicated systems/components.

### Risk C: UI coupled directly to gameplay internals
Menu/HUD reading mutable gameplay nodes directly creates brittle dependencies.

**Mitigation**
- UI reads from a small presentation/state adapter (or signal-fed view-model).
- Avoid direct node-path reaches from UI scenes into gameplay scene internals.

### Risk D: Animation state machine regressions
Fast input transitions can create untested state edges (snap, lock, invalid blend).

**Mitigation**
- Required animation test scene + transition matrix.
- Debug overlay with state, transition reason, blend time, playback rate.
- Add a repeatable rapid-input test script in Slice 2.

---

## 3) Minimum viable architecture (recommended)

Use this minimum architecture before deeper gameplay implementation:

1. **Map scene + data contract**
   - Static geometry/collision
   - Spawn points
   - Trigger volumes with IDs/types
2. **Spawn/trigger router system**
   - Loads map spawns
   - Emits typed events on trigger enter/exit/interact
3. **Entity layer**
   - Player, enemies, interactables, pickups
4. **Thin map director (per-map)**
   - Subscribes to trigger events
   - Applies map-local flow decisions
5. **UI layer**
   - Menus/HUD/input prompts from shared state, not deep scene coupling
6. **Persistence baseline**
   - Save: map ID, key progression flags, minimal player state

---

## 4) Tooling and testing priorities (P0/P1/P2)

## P0 (must exist before gameplay-heavy work)
- Menu navigation smoke path (`launch -> main -> options -> back -> start -> pause -> resume`).
- Focus-state visibility checks for keyboard/controller.
- Trigger debug visualization (volumes + IDs) and event log counts.
- Map validation checklist (spawn reachability, critical trigger reachability, no soft-lock path).
- Credits/attribution gate ensuring CC BY 4.0 notice for PS4 Buttons.

## P1 (add during Slice 1-2)
- Animation compatibility matrix (rig/set coverage for required states).
- Animation transition test scene (idle, locomotion, jump/fall, attack, hit, death).
- Basic telemetry layer (scene load time, trigger counts, death count/time-to-fail).
- Save/load smoke round-trip with deterministic assertions.

## P2 (after loops stabilize)
- Expanded automated regression paths across multiple maps.
- Controller prompt abstraction for additional gamepad families.
- Performance budget checks integrated into release checklist.

---

## 5) Manual vs automation: what to automate first

Early pre-production should bias manual feel checks, but automate deterministic paths immediately.

- **Manual first:** readability, feel, UX friction, animation polish judgments.
- **Automate first:** menu traversal, trigger determinism, scene load, save/load sanity.
- **Automate later:** nuanced animation quality and combat readability (these need stable criteria first).

---

## 6) Slice order recommendation

**Keep Slice 0 first** (UI/menu foundation), with one small guardrail:

- Include a tiny dummy gameplay scene during Slice 0 that can open/close pause and overlay prompts from runtime input context.

This keeps UI-first strategy while preventing menu architecture from diverging from in-game reality.

---

## 7) Measurable gates per slice (baseline thresholds)

- **Slice 0:** 100% deterministic focus traversal in defined menu paths across 10 repeated runs; 0 blocker UX issues in heuristic pass.
- **Slice 1:** critical trigger path succeeds in 20/20 runs; 0 soft-locks on critical route.
- **Slice 2:** no animation state dead-end in rapid-input stress run (5-minute loop); hit feedback readable in checklist pass.
- **Slice 3:** save/load round-trip restores tested flags/state identically in 20/20 runs.

---

## 8) Asset-source review outcomes that affect planning

- **KayKit Character Animations (itch.io):** CC0; FBX/GLTF; retargeting expected for non-matching rigs.
- **KayKit Skeleton Pack (itch.io + Godot Asset Library + GitHub):** CC0; 90+ animations, accessories; legacy itch page points to newer remaster, so version pinning matters.
- **Kenney Prototype Textures (kenney.nl + itch.io + Godot Asset Library):** CC0; 75 textures at 1024x1024; good for greybox readability.
- **Monogram (itch.io):** CC0; TTF and bitmap variants; in practice, Godot projects may prefer explicit bitmap import settings at small sizes.
- **PS4 Buttons (itch.io):** CC BY 4.0; attribution required; PNG+SVG supports flexible prompt rendering pipelines.

---

## 9) Godot anti-patterns to avoid for this scope

- Putting gameplay rules directly in map scene scripts.
- Letting UI scenes traverse gameplay node trees by hardcoded deep paths.
- Trigger logic encoded as ad-hoc strings with no centralized schema.
- Deferring save/load contract definition until after multiple slices.
- Building generalized framework code before proving one map loop and one combat loop.

