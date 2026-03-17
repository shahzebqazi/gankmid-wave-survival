# Game Design Expert Handoff

**Date:** 2026-03-17  
**Project:** Asset-driven Godot game (pre-production)  
**Owner goal:** Validate architecture and define test tooling before coding full game systems.

---

## 1) Why this handoff exists

I want an expert review of my current game architecture and planning before implementation. I also want a practical tooling and test strategy so I can build **vertical slices** and de-risk systems one-by-one.

**Immediate priority:** start vertical slices with **UI and menus** before gameplay-heavy systems.

---

## 2) Current planning baseline

Primary design doc: `docs/plans/game-design-document.md`

Current assumptions from that doc:
- Map owns static level data (geometry, bounds, spawn points, trigger regions).
- Entities own behavior/state (player, enemies, pickups, interactables).
- Map/entity interaction is event-driven (spawn + trigger + map-level flow).
- UI includes menus, HUD, prompts; Monogram font and PS4 prompt icons are planned.

Asset/source manifest: `manifest/README.md`

---

## 3) Asset constraints and implications

- **KayKit Character Animations + KayKit Skeletons**: broad movement/combat animation coverage, good for behavior prototyping.
- **Kenney Prototype Textures**: ideal for greybox maps and readability in early slices.
- **Monogram**: compact font for HUD/menus.
- **PS4 Buttons**: prompt icons for controller-first UX (license requires attribution).

Implication: prioritize functionality and readability over final art polish during slices.

---

## 4) What I need the expert to review

### A. Architecture review
- Validate map/entity ownership boundaries and event flow.
- Identify coupling risks (map logic leakage into entities, UI coupled to gameplay internals, etc.).
- Recommend minimum architecture that is scalable but not over-engineered for solo/small-team development.

### B. Tooling and test strategy
- Define tools/methods to test:
  1. Animation correctness and blend quality
  2. Map playability and trigger reliability
  3. Core game loops and regressions
- Recommend a practical mix of editor tooling, runtime debug overlays, and automated smoke tests.

### C. Vertical slice sequencing
- Validate sequence, scope, and exit criteria for slices.
- Ensure each slice is independently testable and yields decision-making insight.

---

## 5) Proposed pre-production tooling (for review)

## 5.1 Test pyramid for this project

- **Manual design validation (high value early):** moment-to-moment feel, readability, UX friction.
- **Instrumentation and debug views:** fast feedback while tuning maps/animations/UI.
- **Automation (targeted):** smoke tests for scene loads, menu navigation, trigger events, and save/load sanity.

## 5.2 Animation tooling and methods

- **Animation compatibility matrix** (document):
  - Character rig -> animation set compatibility
  - Required states: idle, locomotion, jump/fall, attack, hit, death
- **Animation test scene(s):**
  - One scene cycles all key clips on a reference character
  - Optional second scene tests transitions and blend timings
- **Animation QA checklist:**
  - Foot sliding acceptable/unacceptable thresholds
  - Pose snapping at transitions
  - Attack timing windows (windup/active/recovery)
  - Root motion policy consistency (on/off and why)
- **Debug overlays:**
  - Current state name
  - Transition reason
  - Playback speed and blend time

## 5.3 Map tooling and methods

- **Map validation checklist per level:**
  - Spawn points exist and are reachable
  - Critical triggers are reachable and fire exactly once/when intended
  - No dead-end soft locks on critical path
  - Camera and collision boundaries are valid
- **Map test harness scene:**
  - Spawn player + one reference enemy + one interactable
  - Scripted route to verify start -> objective -> exit flow
- **Debug views:**
  - Toggle trigger volumes
  - Show spawn markers and IDs
  - Show nav/collision diagnostics (if applicable)

## 5.4 Game/system tooling and methods

- **Play mode presets:**
  - Fast start in test map
  - Start from menu flow
  - Resume from checkpoint/save
- **Telemetry light layer (local):**
  - Scene load time
  - Death count/time-to-fail
  - Trigger event counts
  - Menu navigation funnels (where users back out)
- **Regression smoke suite (small):**
  - Launch -> main menu -> options -> back -> start game
  - Pause/resume loop
  - Basic save/load round-trip
  - Critical map trigger path

---

## 6) Vertical slice plan (starting with UI and menus)

## Slice 0: UI Foundation + Menus (first)

**Goal:** validate UX shell and interaction model before gameplay complexity.

**Scope:**
- Main Menu
- Pause Menu
- Options Menu (at least audio + display + controls placeholders)
- Credits (including attribution placeholders)
- Input prompt framework (PS4 icon mapping for key actions)
- Basic style system (Monogram typography, spacing, focus/selection behavior)

**Out of scope:** combat, AI, map progression logic.

**Exit criteria:**
- Can navigate all menus with controller and keyboard (if supported)
- Focus states are always visible and deterministic
- Options persist between sessions (or persistence stub is clearly defined)
- At least one in-game overlay can be entered/exited from a dummy scene
- No blocker-level UX issues in heuristic pass

**Tests:**
- Manual: navigation, edge cases, back/cancel paths
- Smoke automation: menu flow open/close, settings apply/reopen checks

---

## Slice 1: Map Interaction Loop

**Goal:** prove spawn/trigger/exit flow on one small greybox map.

**Scope:**
- One map blockout
- Player spawn
- One interactable trigger and one exit trigger
- Minimal game director flow (trigger -> state change -> map transition or success state)

**Exit criteria:**
- Start-to-exit loop works reliably for repeated runs
- Trigger events are deterministic and logged
- No soft-lock path in the level

---

## Slice 2: Animation + Combat Micro-loop

**Goal:** validate animation state flow and combat readability.

**Scope:**
- One enemy archetype
- Player basic attack and hit reaction loop
- Health/death for both sides (minimal)
- Combat HUD indicators

**Exit criteria:**
- State transitions are stable under rapid input
- Hits and damage feedback are readable
- Combat loop can be completed without desync/logic dead-ends

---

## Slice 3: Progression + Save/Load Sanity

**Goal:** prove continuity across sessions and slices.

**Scope:**
- Minimal progression flag(s)
- Save/load round-trip
- Return to menu and resume flow

**Exit criteria:**
- Save data integrity preserved across restarts
- Restored state is deterministic for tested scenarios

---

## 7) Definition of done for pre-production

Pre-production can be considered complete when:
- UI shell is stable and testable.
- One complete map loop is proven.
- One combat/animation loop is proven.
- Minimal save/load is reliable.
- Tooling exists to quickly catch regressions in those loops.

---

## 8) Key risks to discuss with expert

- Overbuilding architecture before validating feel.
- Under-testing animation transitions (hidden state machine bugs).
- Trigger logic becoming map-specific spaghetti without clear ownership.
- UI built without deterministic input/focus model.
- Scope creep from content production before systems stabilize.

---

## 9) Questions for the expert (requested answers)

1. Is the proposed map/entity boundary right for a small Godot project?
2. What is the minimum debug tooling you would require before gameplay implementation?
3. What should be automated first vs kept manual in early vertical slices?
4. Is Slice 0 (UI first) the right order, or should any gameplay scaffold come earlier?
5. What measurable thresholds should gate each slice (performance, bug count, UX criteria)?
6. What anti-patterns should I actively avoid in Godot scene architecture for this scope?

---

## 10) Expected deliverables from expert review

- Revised architecture notes (1-2 pages max)
- Tooling checklist with priority labels (P0/P1/P2)
- Vertical slice adjustments (scope, order, exit criteria)
- Risk mitigation actions tied to each slice
- A go/no-go checklist for starting full production

---

## 11) References

- `docs/plans/game-design-document.md`
- `manifest/README.md`
