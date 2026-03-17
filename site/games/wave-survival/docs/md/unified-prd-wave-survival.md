# Unified PRD: Wave Survival Vertical Slice

**Date:** 2026-03-17  
**Project:** Godot asset-driven prototype  
**Status:** Approved planning baseline for implementation  
**Compiled from:**  
- `docs/plans/game-design-document.md`  
- `docs/plans/2026-03-17-game-design-expert-handoff.md`  
- `docs/plans/2026-03-17-expert-critique.md`  
- `docs/plans/2026-03-17-vertical-slice-roadmap.md`  
- `docs/plans/2026-03-17-external-expert-prompt.md`  
- `docs/plans/2026-03-17-agent-swarm-handoff.md`  
- `docs/plans/2026-03-17-menu-direction-brief.md`  
- `docs/plans/2026-03-17-zombies-wave-map-mockups-design.md`

---

## 1) Product Vision

Build a small, replayable, Zombies-style survival prototype where the player defends and rotates through compact greybox arenas while waves of `KayKit_Skeletons_1.1_FREE` spawn from random entrances.  

The first production goal is a reliable vertical slice, not content volume:
- deterministic map flow,
- deterministic spawn logic with controlled randomness,
- stable UI shell and HUD,
- clear go/no-go gates per slice.

---

## 2) Experience Pillars

1. **Readable Combat Space**  
   Mixed-neutral prototype textures clearly communicate safe zones, danger lanes, and vertical routes.

2. **Escalating Pressure**  
   Every few waves, more entrances become active and spawn counts increase in predictable tiers.

3. **Fair Randomness**  
   Doors are random, but algorithmically constrained to avoid impossible spikes or degenerate repetition.

4. **Short Session Replayability**  
   Each map supports a 5-10 minute loop with quick restart and visible progression pressure.

---

## 3) Scope

### In Scope (this PRD)
- Wave survival mode on 3 small demo maps:
  - 1 entrance
  - 2 entrances
  - 3 entrances
- Playable character selection from `KayKit_Adventurers_2.0_FREE` (single-player choose-one flow).
- Skeleton-only enemy roster for first combat loop.
- Spawn algorithm driven by wave index and entrance count.
- UI baseline: main/pause/options + in-game HUD.
- Debug overlays for entrances, active door set, and wave diagnostics.

### Out of Scope (for this milestone)
- Full narrative or quest chains.
- Broad enemy taxonomy beyond skeletons.
- Large progression economy and deep inventory.
- Final art polish.
- Character class-specific skill trees or deep RPG stat systems.

---

## 4) Canonical Mockups To Implement

### Gameplay Map Mockups
- `design-docs/zombies-wave-map-mockups.html`
  - **Map A:** Single Breach Holdout (1 entrance)
  - **Map B:** Dual-Pinch Warehouse (2 entrances)
  - **Map C:** Triple-Front Atrium (3 entrances)
  - Includes floor plans + vertical level diagrams and gameplay annotations.

### UI Mockups
- `design-docs/menu-hud-mockups.html`
  - Main menu, pause, options, HUD layout.
  - Current prompt mode is text-only and should become prompt-provider driven in implementation.

---

## 5) Architecture Constraints

1. **Map Owns Static Authored Data**
   - geometry/collision
   - entrance markers
   - spawn points
   - trigger volumes

2. **Entities Own Runtime State/Behavior**
   - player
   - skeleton enemies
   - interactables
   - pickups/projectiles (later)

3. **Thin Per-Map Director**
   - allowed: orchestration and trigger routing
   - not allowed: becoming a combat/AI/UI God object

4. **Central Trigger + Spawn Contracts**
   - typed IDs
   - load-time validation
   - smoke tests for critical paths

---

## 6) Wave Spawn Algorithm (Skeletons)

This algorithm satisfies your requested behavior:
- wave 1 has exactly 1 skeleton,
- in maps with `n` entrances, doors ramp from `1..n`,
- at wave `c * n`, each door has exactly 1 skeleton in that wave.

### 6.1 Variables

- `w` = 1-based wave index
- `n` = number of entrances on current map (`1`, `2`, or `3` for initial demo maps)
- `c` = ramp constant (positive integer; recommended default `c = 3`)
- `E = [0..n-1]` entrance indices

### 6.2 Active door count

`k(w) = min(n, ceil(w / c))`

Properties:
- `k(1) = 1`
- `k` increases by 1 every `c` waves
- `k(c*n) = n`

### 6.3 Random contiguous active doors

Pick random start door `s` from `[0..n-1]`.  
Active door set:

`A(w) = { (s + i) mod n | i = 0..k(w)-1 }`

This implements: random door + next `1,2,3..n` doors around the ring.

### 6.4 Skeletons per active door

Base per active door:

`q(w) = 1 + floor((w - 1) / (c * n))`

Total skeletons:

`S(w) = k(w) * q(w)`

Guarantees:
- `w = 1`: `k = 1`, `q = 1`, `S = 1`
- `w = c*n`: `k = n`, `q = 1` -> each door gets exactly 1 skeleton
- Later cycles increase pressure without changing the fairness rule.

### 6.5 Spawn scheduler (within-wave pacing)

Do not drop all skeletons in one frame.  
For each door `d` in `A(w)`:
- spawn `q(w)` skeletons,
- stagger by `spawn_interval` (ex: `0.65s`, curve down slowly over waves),
- apply slight per-door jitter (`+/- 0.1s`) for natural pacing.

### 6.6 Spawn timing algorithm (when skeletons spawn)

This section defines exact timing so spawn cadence is deterministic and tunable.

#### Timing parameters
- `base_interval = 0.65` seconds
- `interval_decay = 0.03`
- `min_interval = 0.30` seconds
- `between_wave_delay = 4.0` seconds
- `clear_buffer = 1.5` seconds

#### Effective interval by wave

`I(w) = max(min_interval, base_interval - interval_decay * floor((w - 1) / (c * n)))`

#### Wave start time

Let `T(1) = 0`. For `w > 1`:

`T(w) = T(w-1) + D(w-1) + between_wave_delay + clear_buffer`

Where:
- `D(w-1)` is measured runtime duration from first spawn to last kill in wave `w-1`.

#### Spawn timestamp per door and unit

For each active door `d in A(w)` and each unit index `j in [0..q(w)-1]`:

`t_spawn(w, d, j) = T(w) + lane_offset(d) + j * I(w) + jitter(j)`

Where:
- `lane_offset(d)` is a small door offset (example: `0.0`, `0.12`, `0.24`) to avoid perfect synchronization,
- `jitter(j)` is random in `[-0.1, +0.1]`.

#### Alive-cap guard

At each scheduled spawn tick:
- if `alive_count >= max_alive_cap`, postpone this spawn by `0.25s` and retry.

### 6.7 Pseudocode

```text
function build_wave(w, n, c, rng):
    k = min(n, ceil(w / c))
    s = rng.int(0, n - 1)
    active_doors = []
    for i in 0..k-1:
        active_doors.append((s + i) % n)

    q = 1 + floor((w - 1) / (c * n))
    interval = max(min_interval, base_interval - interval_decay * floor((w - 1) / (c * n)))
    plan = []
    for door in active_doors:
        for j in 0..q-1:
            t = wave_start_time + lane_offset(door) + (j * interval) + rng.range(-0.1, 0.1)
            plan.append({ door: door, enemy: "skeleton", delay: t })
    return plan
```

### 6.8 Example (`c = 3`)

- **Map A (`n=1`)**  
  - Wave 1: 1 skeleton from entrance 0  
  - Wave 3: 1 skeleton from entrance 0  
  - Wave 4: 2 skeletons from entrance 0

- **Map B (`n=2`)**  
  - Wave 1-3: 1 active door, 1 skeleton  
  - Wave 6 (`c*n`): both doors active, 1 skeleton each

- **Map C (`n=3`)**  
  - Wave 1-3: 1 door active  
  - Wave 4-6: 2 contiguous doors active  
  - Wave 9 (`c*n`): all 3 doors active, 1 skeleton each

---

## 7) Functional Requirements

1. **Entrance System**
   - Map defines named entrances with transforms and ring-order index.
   - Director resolves entrance list and validates contiguous ordering.

2. **Wave Director**
   - Computes `k(w)`, `A(w)`, `q(w)`, and spawn schedule.
   - Emits wave lifecycle events: `wave_start`, `wave_spawn_done`, `wave_clear`.

3. **Spawner**
   - Spawns skeleton scenes from `KayKit_Skeletons_1.1_FREE`.
   - Supports delayed queued spawn requests per entrance.

4. **Skeleton AI (baseline archetype)**
   - Skeletons continuously move toward the player using simple chase behavior.
   - Movement speed is intentionally slow to preserve readability and allow kiting.
   - Skeletons only attack when player distance is within a short melee range.
   - Attack damage is intentionally low; pressure comes from numbers, not burst damage.
   - Skeletons return to chase state immediately when target leaves attack range.

5. **Player Character and Combat Baseline**
   - Player can select one playable character from `KayKit_Adventurers_2.0_FREE` before starting a run.
   - Player combat animation clips are sourced from `KayKit_Character_Animations_1.1`.
   - Player primary attack is intentionally strong for this prototype: 1 hit kills 1 skeleton.
   - Player attack uses a medium melee range and medium hitbox volume for forgiving early-slice feel.
   - One-hit behavior is prototype-only and can be tuned later after wave pacing is validated.

6. **UI/HUD**
   - Show wave number, alive enemy count, and in-game wave timer.
   - Optional debug panel: active doors + planned spawn counts.

7. **Map-specific tuning**
   - Shared algorithm, map-specific constants allowed:
     - `c`
     - spawn interval
     - max alive cap
     - safe delay between waves
     - skeleton movement speed
     - skeleton attack range
     - skeleton attack damage
     - player attack range
     - player attack hitbox size

---

## 8) Balancing Constants (Initial Defaults)

- `c = 3`
- `spawn_interval = 0.65s`
- `min_interval = 0.30s`
- `interval_decay = 0.03`
- `between_wave_delay = 4.0s`
- `clear_buffer = 1.5s`
- `max_alive_cap`:
  - Map A: `8`
  - Map B: `12`
  - Map C: `16`
- Skeleton AI baseline:
  - `skeleton_move_speed = 1.6 m/s` (slow walk)
  - `skeleton_attack_range = 1.2 m` (short melee reach)
  - `skeleton_attack_damage = 4` per hit (low damage)
  - `skeleton_attack_cooldown = 1.1 s`
- Player baseline:
  - `player_selectable_roster = KayKit_Adventurers_2.0_FREE`
  - `player_animation_source = KayKit_Character_Animations_1.1`
  - `player_attack_damage_vs_skeleton = 100` (targeting guaranteed 1-hit kill with current skeleton health baseline)
  - `player_attack_range = 2.2 m` (medium range)
  - `player_attack_hitbox_radius = 1.1 m` (medium hitbox)

Tune only after deterministic behavior is validated.

---

## 9) Test and Quality Gates

### Deterministic Tests
- Wave 1 always yields exactly 1 skeleton.
- For each map `n`, wave `c*n` yields all `n` doors active and exactly 1 skeleton per door.
- No entrance outside `A(w)` spawns during wave `w`.
- Spawn schedule length equals `S(w)`.
- Spawn event timestamps are strictly non-decreasing within each door queue.
- Skeleton AI state transitions obey: `Chase -> Attack` only when `distance <= skeleton_attack_range`.
- Skeleton AI transitions obey: `Attack -> Chase` when `distance > skeleton_attack_range`.
- Character select yields exactly one spawned adventurer character per run start.
- Player basic attack always kills a full-health skeleton in one landed hit.
- Attack hit registration occurs only when skeleton is inside `player_attack_range` and `player_attack_hitbox_radius`.

### Runtime Smoke
- 20/20 run reliability:
  - start map
  - survive/clear wave 1
  - progress through wave `c*n`
  - return to menu/restart

### Debug Validation
- On-screen debug can print:
  - `w`, `n`, `c`, `k`, `A(w)`, `q`, `S`, `I(w)`, and current wave timer
- Optional AI debug print can show:
  - `skeleton_state`, `distance_to_player`, and `last_attack_time`
- Trigger/entrance IDs validated at map load with hard error on mismatch.

---

## 10) Audio System

### 10.1 Overview

Audio is integral to game feel and must be wired by Cycle 2. The system consists of:
- An `AudioManager` autoload for centralized music/sting playback and bus management.
- Positional `AudioStreamPlayer3D` nodes on player and skeleton scenes for spatial SFX.
- Non-positional `AudioStreamPlayer` nodes in UI scripts for menu sounds.

### 10.2 Audio Bus Layout

| Bus | Purpose | Default Volume |
|-----|---------|---------------|
| Master | Global output | 0 dB |
| Music | All `mus_` tracks | -6 dB |
| SFX | All `sfx_` combat/entity tracks | -3 dB |
| UI | All `sfx_ui_` tracks | -3 dB |
| Ambient | `sfx_ambient_` tracks | -12 dB |

### 10.3 Required Audio Files

All audio sourced from Freesound.org under CC0 or CC-BY 4.0 licenses. See `design-docs/audio-asset-gallery.html` for playable previews and download links.

| ID | Target File | Source | Freesound ID | License |
|----|-------------|--------|-------------|---------|
| `mus_menu` | `music/menu_theme.ogg` | Dark Dungeon Ambience — Kinoton | 516566 | CC0 |
| `mus_combat_01` | `music/combat_loop_01.ogg` | battle-march action loop — haydensayshi123 | 138681 | CC0 |
| `mus_wave_clear` | `music/wave_clear_sting.ogg` | Victory (short sting) — xkeril | 706753 | CC0 |
| `mus_game_over` | `music/game_over_sting.ogg` | Jingle_Lose_00 — LittleRobotSoundFactory | 270467 | CC-BY 4.0 |
| `sfx_player_footstep_01` | `sfx/player/footstep_01.ogg` | Stone Steps — Phil25 | 208103 | CC0 |
| `sfx_player_footstep_02` | `sfx/player/footstep_02.ogg` | Steps on concrete — areniporgen | 712084 | CC0 |
| `sfx_player_attack_swing` | `sfx/player/sword_swing.ogg` | Sword_whoosh_sound — Artninja | 700220 | CC-BY 4.0 |
| `sfx_player_attack_hit` | `sfx/player/sword_hit.ogg` | Hit Impact Sword 2 — CogFireStudios | 547036 | CC0 |
| `sfx_player_block` | `sfx/player/shield_block.ogg` | Metal Impact — CGEffex | 97792 | CC-BY 4.0 |
| `sfx_player_hurt` | `sfx/player/player_hurt.ogg` | jump-grunt — luminousfridge | 561555 | CC0 |
| `sfx_player_jump` | `sfx/player/jump.ogg` | Concrete Footsteps — SoftDistortionFX | 465299 | CC0 |
| `sfx_player_land` | `sfx/player/land.ogg` | BodyFall — 000600 | 187136 | CC0 |
| `sfx_skeleton_spawn` | `sfx/skeleton/skeleton_spawn.ogg` | Rattling Bones — spookymodem | 202102 | CC0 |
| `sfx_skeleton_footstep` | `sfx/skeleton/skeleton_step.ogg` | Skeleton bones (game) — cribbler | 381859 | CC0 |
| `sfx_skeleton_attack` | `sfx/skeleton/skeleton_attack.ogg` | Sword hit (low metal) — xkeril | 815763 | CC0 |
| `sfx_skeleton_hit` | `sfx/skeleton/skeleton_hit.ogg` | Sword Hit — qubodup | 442769 | CC0 |
| `sfx_skeleton_death` | `sfx/skeleton/skeleton_death.ogg` | Falling Bones — spookymodem | 202091 | CC0 |
| `sfx_skeleton_aggro` | `sfx/skeleton/skeleton_aggro.ogg` | Death by Zombie — giddster | 416060 | CC-BY 4.0 |
| `sfx_ui_navigate` | `sfx/ui/ui_navigate.ogg` | SFX UI Button Click — suntemple | 253168 | CC0 |
| `sfx_ui_confirm` | `sfx/ui/ui_confirm.ogg` | Game Audio UI SFX — GameAudio | 220206 | CC0 |
| `sfx_ui_back` | `sfx/ui/ui_back.ogg` | Clicks, Buttons & UI — Breviceps | 366104 | CC0 |
| `sfx_ui_pause` | `sfx/ui/ui_pause.ogg` | (derived from Kinoton drone) | 516566 | CC0 |
| `sfx_ambient_dungeon` | `sfx/environment/ambient_dungeon.ogg` | Dark Dungeon Ambience — Kinoton | 516566 | CC0 |
| `sfx_wave_start` | `sfx/environment/wave_horn.ogg` | Distant War Horn — DeVern | 512490 | CC0 |

### 10.4 AudioManager Autoload

The `AudioManager` autoload script (`scripts/autoload/audio_manager.gd`) provides:

- `play_music(stream_id: String)` — crossfades between music tracks.
- `play_sfx_ui(stream_id: String)` — one-shot UI sound.
- `play_sting(stream_id: String)` — ducks music bus, plays sting, restores music.
- `set_bus_volume(bus_name: String, linear: float)` — maps 0.0–1.0 linear to dB. Called by options menu sliders.
- Volume ducking: reduce Music bus by -6 dB when pause menu is visible.

### 10.5 Options Menu — Audio Sliders

The Options menu provides volume sliders for each audio bus:

| Slider | Bus | Range | Default | Persistence |
|--------|-----|-------|---------|-------------|
| Master Volume | Master | 0–100 | 80 | `user://settings.cfg` |
| Music Volume | Music | 0–100 | 70 | `user://settings.cfg` |
| SFX Volume | SFX | 0–100 | 80 | `user://settings.cfg` |

UI and Ambient buses are controlled as sub-ratios of SFX to keep the options screen simple (3 sliders). UI bus tracks SFX at 1:1 ratio. Ambient bus tracks SFX at 0.5x ratio (always quieter).

Slider behavior:
- Horizontal `HSlider` with label showing current value (0–100).
- Changes apply immediately (real-time preview while adjusting).
- Play `sfx_ui_navigate` on slider value change for audible feedback.
- Settings persist to `user://settings.cfg` via `ConfigFile`.
- "Reset Defaults" button restores all sliders to default values.
- Accessible from both Main Menu and Pause Menu via the existing "Options" button.
- Mockup reference: `design-docs/menu-hud-mockups.html` (Options Menu with Sliders panel).

### 10.6 Audio Trigger Wiring

| Script | Triggers |
|--------|----------|
| `main_menu.gd` | `mus_menu` on `_ready()`, `sfx_ui_confirm` on Start Game, `sfx_ui_navigate` on focus change |
| `pause_menu.gd` | `sfx_ui_pause` on pause open, `sfx_ui_confirm` on Resume, duck Music bus while paused |
| `player.gd` | `sfx_player_footstep_01/02` alternating on run, `sfx_player_attack_swing` on attack, `sfx_player_attack_hit` on enemy damage, `sfx_player_block` on blocked hit, `sfx_player_hurt` on damage taken, `sfx_player_jump` on jump, `sfx_player_land` on landing |
| `skeleton_enemy.gd` | `sfx_skeleton_spawn` on spawn anim, `sfx_skeleton_footstep` while walking, `sfx_skeleton_attack` on attack, `sfx_skeleton_hit` on non-lethal damage, `sfx_skeleton_death` on die, `sfx_skeleton_aggro` on first chase |
| `wave_director.gd` | `sfx_wave_start` on wave_started, `mus_wave_clear` sting on wave_cleared |
| `game_state.gd` | `mus_combat_01` on Phase.PLAYING, `mus_game_over` sting on Phase.GAME_OVER |
| `map_base.gd` | `sfx_ambient_dungeon` loop on `_ready()` |

---

## 11) Vertical Slice Implementation Order

1. **Slice 0 - UI Foundation**
   - Implement `design-docs/menu-hud-mockups.html` structure in Godot UI scenes.
   - Includes Options menu with audio volume sliders (Master, Music, SFX).

2. **Slice 1 - Map Interaction Loop**
   - Implement Map A with entrance + trigger contracts.

3. **Slice 2 - Combat Micro-loop**
   - Integrate skeleton archetype and wave director algorithm.

4. **Slice 3 - Audio and Settings**
   - Download and convert all audio files from Freesound sources (see section 10.3).
   - Create audio bus layout and AudioManager autoload.
   - Wire all sound triggers per section 10.6.
   - Implement options menu settings persistence (`user://settings.cfg`).

5. **Slice 4 - Progression and Save/Load**
   - Add minimal round-trip persistence for map ID + wave state baseline.

---

## 12) Risks and Mitigations

1. **Wave spikes feel unfair**
   - Mitigate with `max_alive_cap` and per-wave pacing.

2. **Door randomness repeats excessively**
   - Add anti-repeat guard (do not reuse same start door >2 waves in a row).

3. **Map-specific script drift**
   - Keep one shared wave director and per-map config data only.

4. **UI/gameplay coupling**
   - HUD reads from wave state adapter, not direct deep node paths.

---

## 13) Implementation Backlog (Execution-ready)

1. Create shared `WaveDirector` spec + config resource (`n`, `c`, pacing constants).
2. Define entrance marker node type and map validation script.
3. Implement Map A entrance setup + smoke test.
4. Implement skeleton spawn queue and delayed spawn scheduler.
5. Integrate wave/HUD state binding.
6. Implement Map B and Map C with same algorithm and map-tuned caps.
7. Add automated checks for the `w=1` and `w=c*n` invariants.
8. Download audio files from Freesound (section 10.3), convert to `.ogg`, place in `project/assets/audio/`.
9. Create `AudioManager` autoload and audio bus layout per section 10.2/10.4.
10. Build Options menu scene with volume sliders per section 10.5 mockup.
11. Wire all sound triggers per section 10.6.
12. Persist audio settings to `user://settings.cfg`.

---

## 14) Definition of Done

PRD execution is complete when:
- all 3 maps run the same wave algorithm with different `n`,
- invariants pass (`w=1`, `w=c*n`) on each map,
- menu + HUD slice is functional and stable,
- wave progression is readable and fair in playtests,
- audio plays for: menu music, combat music, sword swing, skeleton death, UI confirm, wave horn,
- options menu volume sliders work and persist settings,
- no blocker-level soft-locks or spawn-contract errors remain.
