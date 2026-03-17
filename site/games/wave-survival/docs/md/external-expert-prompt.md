# External Expert Review Request (Copy/Paste Prompt)

Use this prompt when requesting an external game design/architecture review.

---

## Prompt

I am preparing pre-production architecture for a small/solo Godot game project and want an expert review before full implementation.

### Project context
- Asset-driven project using free packs (KayKit animations/skeletons, Kenney prototype textures, Monogram font, PS4 button icons).
- Current goal is to validate architecture and define a practical testing/tooling strategy before expanding gameplay systems.
- I am deliberately using vertical slices to de-risk one system at a time.
- Immediate priority is **UI and menus first** before gameplay-heavy systems.

### Current architecture assumptions
- **Map owns static authored data:** geometry/collision, bounds, spawn points, trigger regions.
- **Entities own runtime behavior/state:** player, enemies, interactables, pickups, combat state.
- **Map/entity interaction:** event-driven spawn + trigger + map-level flow.
- **UI scope:** menus + HUD + interaction prompts.

### Asset/source constraints
- KayKit Character Animations: CC0, FBX/GLTF, broad movement/combat coverage.
- KayKit Skeletons: CC0, rigged skeletons + 90+ animations, accessories.
- Kenney Prototype Textures: CC0, prototype/greybox texture set.
- Monogram font: CC0, TTF/bitmap variants.
- PS4 Buttons: **CC BY 4.0 (attribution required)**, PNG+SVG.

### Proposed vertical slices
1. **Slice 0: UI Foundation + Menus**
   - Main/Pause/Options/Credits, input prompts, focus behavior, options persistence.
2. **Slice 1: Map Interaction Loop**
   - One map, player spawn, one interactable trigger, one exit trigger, minimal map director flow.
3. **Slice 2: Animation + Combat Micro-loop**
   - One enemy archetype, basic attack/hit/death loop, combat HUD indicators.
4. **Slice 3: Progression + Save/Load Sanity**
   - Minimal progression flags, save/load round-trip, return-to-menu/resume flow.

### What I want reviewed
1. Is this map/entity ownership boundary correct for a small Godot project?
2. What minimum debug tooling must exist before gameplay implementation?
3. What should be automated first vs kept manual in early slices?
4. Is UI-first slice ordering correct, or should I front-load any gameplay scaffold?
5. What measurable thresholds should gate each slice?
6. Which Godot architecture anti-patterns should I actively avoid at this scope?

### Constraints for recommendations
- Keep recommendations lean and implementation-practical for solo/small-team velocity.
- Avoid over-engineering or framework-heavy abstractions.
- Prefer deterministic validation gates over subjective "seems good" checks.

### Required response format

Please answer using this exact structure:

1. **Architecture verdict (max 10 bullets)**
   - What is correct
   - What must change now
   - What can wait

2. **Coupling and failure risks (ranked high -> low)**
   - Risk
   - Why it fails in practice
   - Minimal mitigation

3. **Tooling checklist with priorities**
   - **P0:** must-have before gameplay-heavy implementation
   - **P1:** should-have during slices 1-2
   - **P2:** nice-to-have after loops stabilize

4. **Automation strategy**
   - What to automate immediately
   - What to keep manual initially
   - What to automate later (with trigger conditions for when)

5. **Slice adjustments**
   - Keep/change recommendation for each slice
   - Revised scope if needed
   - Revised exit criteria if needed

6. **Anti-pattern watchlist**
   - Top 5 anti-patterns to avoid in this Godot setup

7. **Go/No-Go production gate**
   - Short checklist to determine if full production should start

8. **30-day action plan**
   - Week-by-week priorities aligned to the slices

---

## Optional attachment list to send with this prompt

- `docs/plans/2026-03-17-game-design-expert-handoff.md`
- `docs/plans/game-design-document.md`
- `manifest/README.md`
- `docs/plans/2026-03-17-expert-critique.md`
- `docs/plans/2026-03-17-vertical-slice-roadmap.md`

