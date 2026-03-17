# Game Design Document

**Project:** Godot game (asset-driven)  
**Focus:** Architecture and planning — maps, domain model (map ↔ entities), UI, and remaining work. No implementation yet.

---

## 1. Game concept (asset-informed)

The game is driven by the available packs:

- **KayKit Character Animations** + **KayKit Skeletons** → 3D humanoid/skeleton characters, combat and movement animations, weapons (melee, bow, staff, shield).
- **Kenney Prototype Textures** → Levels built from prototype/grid textures (greybox style).
- **Monogram** → UI and in-world text.
- **PS4 Buttons** → Controller input prompts (e.g. “Press [X] to interact”).

**Implications:** 3D levels (or 2.5D); characters that move, fight, and interact; prototype-visual maps; clear UI and controller feedback. Exact genre (action RPG, dungeon crawler, arena, etc.) can be chosen later; the architecture below supports any of these.

---

## 2. Maps

### 2.1 What a map is

A **map** is a playable space the player and other entities inhabit. It provides:

- **Geometry** — Walkable and non-walkable surfaces (floors, walls, platforms, hazards). Built from meshes using Kenney (or other) textures.
- **Bounds** — Where the playable area starts and ends (kill zones, camera limits, loading boundaries).
- **Spawn and placement data** — Where the player, enemies, NPCs, and interactables are initially placed (and optionally respawn).
- **Triggers and regions** — Invisible volumes or surfaces that fire events (e.g. “player entered zone”, “lever activated”, “door opened”, “load next map”).
- **Lighting and atmosphere** — Lights, fog, ambient settings (visual only; no domain logic).
- **Optional:** Props, decoration, and “gameplay furniture” (doors, levers, chests) that can be pure art or wired into the domain model as entities or map-owned objects.

Maps are **authored** in the editor (Godot scenes or a level format) and **loaded** at runtime. They do not contain game rules; they describe space and initial state.

### 2.2 Map types and flow

- **Level / scene** — One self-contained playable map (e.g. one dungeon, one arena, one hub).
- **Flow** — How the game moves between maps: direct scene change, additive loading, or “world” made of multiple chunks. Decide up front so the domain model and save system can assume one flow.

### 2.3 What you need to do for maps

- Define **map format** (e.g. single scene per level, or scene + optional JSON/data for spawns and triggers).
- Build **first playable map(s)** (blockout with Kenney textures, then add spawns and triggers).
- Define **naming and folder convention** (e.g. `maps/level_01`, `maps/hub`).
- Decide **who owns what**: map owns geometry and trigger volumes; entities own position, state, and behaviour.

---

## 3. Domain model: map and entities

### 3.1 Core idea

- **Map** = space + static level data (geometry, bounds, spawn points, trigger volumes, maybe static props).
- **Entities** = things that have **state**, **behavior**, and **position** in that space (player, enemies, NPCs, pickups, moving platforms, doors, etc.).
- **Interactions** = how map and entities affect each other (events, queries, and rules), without the map “knowing” entity logic or entities “knowing” map layout details.

### 3.2 Map (domain responsibilities)

- **Owns:**  
  - Geometry and collision.  
  - Spawn points (named or typed: “player_start”, “enemy_skeleton_01”, “pickup_health”).  
  - Trigger regions (volumes or areas) and their IDs or types.  
  - Optional: list of “level-owned” interactables (e.g. doors, levers) if you keep them out of the generic entity list.
- **Provides:**  
  - “Give me spawn transform for type X.”  
  - “Is point P inside a trigger? Which one?”  
  - “Spawn the player / enemies / pickups at their points.”  
- **Does not:** Run AI, handle input, or hold health/inventory; that stays in entities or systems.

### 3.3 Entities (domain responsibilities)

- **Player** — Input, movement, combat, interaction, camera (if not a separate system). Consumes “spawn at X” from map; reports “I entered/exited trigger T” or “I used interactable I”.
- **Enemies / NPCs** — AI, movement, combat, dialogue. Spawned by map (or by spawners that the map places). Can react to map events (e.g. “trigger T activated → open door”).
- **Pickups / collectibles** — Position, collision, “on collected” behaviour (e.g. add health, key). Often spawned by map; removed or disabled when collected.
- **Interactables** — Doors, levers, chests. Have state (open/closed, on/off). Map or a “level script” can define which trigger or entity toggles which interactable.
- **Projectiles / hazards** — Optional; can be entities that the map or spawners create. Map provides collision; entities handle damage and lifetime.

Entities **live in** the map but are **not part of** the map’s static data; they are created at load or at runtime from map spawn data.

### 3.4 Interactions (map ↔ entities)

- **Map → entities**  
  - **Spawn:** Map (or a spawn system reading map data) creates entities at spawn points.  
  - **Trigger events:** When something (usually the player) enters/exits a trigger, the map (or a level script) emits an event; entities or systems subscribe (e.g. “trigger_door_1” → open door entity).  
  - **Queries:** “Is position P walkable?”, “Which trigger contains P?” — used by AI or movement.
- **Entities → map**  
  - **Position / movement:** Entities move; map provides collision and physics. No need for map to “store” entity position; the engine does.  
  - **Trigger reporting:** “Player entered trigger_id X” or “Player pressed interact in region Y”. Map or a **level script / game director** translates that into map-level actions (load next map, open door, enable spawner).
- **Entities ↔ entities**  
  - Combat, dialogue, pickups: handled by entity/systems logic (e.g. “skeleton” vs “player” damage). Map only needs to know if you want triggers to react to entity state (e.g. “all enemies dead → open exit”).

So: **map = space and level-authored data; entities = everything that moves and has behaviour; interactions = spawn, triggers, and optional queries.**

### 3.5 Optional: “Level script” or “game director”

A single per-map object or small script can:

- Listen to trigger and entity events.
- Drive map-level flow: “when trigger_start_combat → enable spawners”; “when all enemies dead → open exit trigger”; “when player touches exit_trigger → load next map”.

This keeps map-specific logic in one place and out of the generic entity systems.

### 3.6 Diagram (conceptual)

```
┌─────────────────────────────────────────────────────────────────┐
│ MAP                                                              │
│  • Geometry (collision, visuals)                                 │
│  • Spawn points (player, enemies, pickups)                       │
│  • Trigger regions (IDs / types)                                  │
│  • Optional: level-owned interactables (doors, levers)            │
└──────────────┬──────────────────────────────────┬───────────────┘
               │                                    │
       spawn   │                                    │  trigger
       points  │                                    │  events
               ▼                                    ▼
┌──────────────────────────────────────────────────────────────────┐
│ ENTITIES (live in map, created from spawn data or at runtime)     │
│  • Player (input, movement, combat, interact)                     │
│  • Enemies / NPCs (AI, combat)                                    │
│  • Pickups, interactables (doors, levers), projectiles            │
└──────────────┬──────────────────────────────────┬────────────────┘
               │                                    │
               │  position / collision              │  "entered trigger X"
               │  (physics)                         │  "interact in region Y"
               ▼                                    ▼
┌──────────────────────────────────────────────────────────────────┐
│ LEVEL SCRIPT / GAME DIRECTOR (per map, optional)                  │
│  • Map-specific flow: trigger → open door, enable spawner,        │
│    load next map, win/lose conditions                             │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4. UI

### 4.1 Screens (out of gameplay)

- **Main menu** — Start game, options, credits, quit. Uses Monogram for text; optional PS4 button hints for “confirm/cancel”.
- **Pause** — Resume, options, back to main menu. Same font and button style.
- **Options** — Volume (Master/Music/SFX sliders with real-time preview), controls (and display of controller layout using PS4 button assets), resolution/fullscreen. Save/cancel. See interactive mockup in `design-docs/menu-hud-mockups.html`.
- **Game over / victory** — Message + restart / next level / main menu. Monogram + button prompts.
- **Credits** — Attribution (especially PS4 Buttons, CC BY 4.0) and thanks. Text-only or simple layout.

### 4.2 In-game HUD

- **Health / resource bars** — Position (e.g. top-left or corner), style (flat/prototype to match Kenney).
- **Crosshair / aim** — If ranged combat; can be minimal (dot or cross).
- **Interaction prompt** — “Press [X] to interact” using PS4 Buttons asset; show only when player can interact. Use Monogram for short labels (“Open door”, “Talk”).
- **Objectives / hints** — Optional; short lines of text (Monogram), e.g. top or bottom of screen.
- **Minimap** — Optional; can be added later without changing the domain model.

All HUD elements should be in a **single UI layer** (one scene or canvas) so layout and resolution scaling stay in one place.

### 4.3 Input prompts

- Use **PS4 Buttons** art for any “press [button]” hints (interact, jump, attack, menu).
- Plan for **one source of truth** for “current input type” (keyboard vs gamepad) so prompts can switch between keys and controller buttons if you add PC support later.

### 4.4 UI asset usage

- **Monogram** — All text (menus, HUD labels, prompts, objectives).
- **PS4 Buttons** — Controller button icons only; keep a small mapping table (action → icon name) for design and future localization.

---

## 5. What else you need to do (checklist)

Beyond maps, domain model, and UI, you will need to define or implement:

### 5.1 Core loop and feel

- [ ] **Core loop** — What the player does minute-by-minute (explore → fight → loot → progress). Lock this before building many levels.
- [ ] **Camera** — Follow player, collision with geometry, optional look-around; decide 3rd-person vs fixed etc.
- [ ] **Game feel** — Movement speed, jump/gravity, attack feedback, hit reactions, screen shake, sound placeholders. Affects tuning, not just architecture.

### 5.2 Systems (no code here, just scope)

- [ ] **Combat** — Hitboxes, damage, health, death (player and enemies). Who applies damage (entity vs dedicated combat system).
- [ ] **Inventory / progression** — If any: what is persistent (keys, gold, upgrades) and how it’s stored (save data).
- [ ] **Save / load** — What to save (current map, player state, world flags). When to save (checkpoints, manual, auto). Format and location (file path, optional cloud).
- [ ] **Audio** — Music per map or per state; SFX for movement, combat, UI, triggers. All sounds sourced from Freesound.org — see `design-docs/audio-design-document.md` for full spec and `design-docs/audio-asset-gallery.html` for playable previews.
- [ ] **Settings** — Persist volume (Master/Music/SFX sliders), controls, resolution; hook options screen to these. See options mockup in `design-docs/menu-hud-mockups.html`.

### 5.3 Content and balance

- [ ] **Level design** — Build and test maps; place spawns and triggers; playtest flow and difficulty.
- [ ] **Enemy and encounter design** — Types, placement, and difficulty curve.
- [ ] **Narrative / goals** — If any: simple story, objectives, or “clear all rooms” only. Enough to drive level order and win condition.

### 5.4 Technical and release

- [ ] **Project setup** — Godot project, folders (e.g. `maps/`, `entities/`, `ui/`, `assets/`), first runnable scene (e.g. main menu).
- [ ] **Input mapping** — Action names (e.g. `move`, `jump`, `attack`, `interact`, `pause`) and default bindings (gamepad + optional keyboard).
- [ ] **Build and export** — Target platform(s), export presets, and a “first playable” build.
- [ ] **Attribution** — Credits screen and/or README with PS4 Buttons (CC BY 4.0) and any other required attributions.

---

## 6. Suggested order of work (planning only)

1. **Lock high-level loop** — Genre, win condition, and one-sentence core loop.
2. **Define map format and folder structure** — One scene per map (or chunks); where spawn and trigger data live.
3. **Implement domain boundaries** — Map vs entities vs level script (on paper or in a one-page “architecture summary”).
4. **Design first map** — One small level: blockout, one player spawn, one trigger (e.g. “exit”), one enemy spawn. Prove map ↔ entity flow.
5. **UI list** — Finalize screens and HUD list; then implement in order (main menu → HUD → pause → options → game over).
6. **Then** — Combat, save/load, audio, more content, polish.

---

## 7. Summary

- **Maps** = geometry, bounds, spawn points, triggers; authored in editor; no game logic inside.
- **Domain model** = map owns space and level data; entities (player, enemies, pickups, interactables) own state and behaviour; interactions = spawn, triggers, and optional level script.
- **UI** = main menu, pause, options, game over, credits; in-game HUD and prompts; Monogram for text, PS4 Buttons for controller icons.
- **What else** = core loop and feel, camera, combat, save/load, audio, settings, level design, narrative/goals, project layout, input mapping, build, attribution.

No code in this doc; next step is to turn this into a concrete implementation plan (e.g. scene structure, node names, and task list) when you’re ready to build.
