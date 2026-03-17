# Cycle 3: Power-Ups and Tuning Prep Handoff Prompt

Use this prompt after Cycle 2 is complete (bugs fixed, audio wired, all 3 maps playable, character select, game over screen).

---

## Prompt

You are a **Godot 4.6 game development agent** continuing work on a wave-survival dungeon prototype. Cycle 2 delivered audio, all 3 maps, character select, and game over. Your job in Cycle 3 is to implement the power-up drop system and prepare the game for manual balance tuning.

### Context

The project is at `/home/sqazi/godot-game/project/` (Godot 4.6.1). Godot is installed at `/usr/bin/godot`.

---

## Required Inputs (read all before starting)

### Design docs
- `design-docs/powerup-design-document.md` — **PRIMARY INPUT** — full power-up spec with drop mechanics, buff tracking, HUD integration, scene structure, constants, and icon assets
- `docs/plans/2026-03-17-unified-prd-wave-survival.md` — canonical PRD
- `docs/plans/2026-03-17-mvp-status-report.md` — architecture notes
- `docs/plans/2026-03-17-cycle-2-handoff-prompt.md` — context on what Cycle 2 built

### Icon assets (already created)
- `project/assets/icons/powerup_double_jump.svg` — cyan, double up-arrows
- `project/assets/icons/powerup_regen.svg` — green, healing cross
- `project/assets/icons/powerup_attack_speed.svg` — red, speed arrows

### Current codebase
Read all scripts in `project/scripts/` before starting. Key files:
- `scripts/entities/skeleton_enemy.gd` — `_die()` is where drop rolls happen
- `scripts/entities/player.gd` — receives buff effects
- `scripts/autoload/wave_config.gd` — all tuning constants go here
- `scripts/autoload/game_state.gd` — buff state tracking
- `scripts/ui/hud.gd` — buff icon display
- `scripts/systems/wave_director.gd` — spawning context

---

## Development Tracks (execute in order)

### Track 1: Power-Up Constants and State
- Add all power-up constants from the design doc section 8 to `wave_config.gd` under a new `@export_group("Power-Ups")`.
- Add buff tracking to `GameState`:
  - `var active_buffs: Dictionary = {}` mapping power-up ID string to remaining seconds.
  - Tick down in `_process()` while PLAYING.
  - Methods: `activate_buff(id: String)`, `has_buff(id: String) -> bool`, `get_buff_time(id: String) -> float`, `clear_all_buffs()`.
  - Signal: `buff_changed(id: String, active: bool, remaining: float)`.
  - Clear all buffs on `game_over()` and `return_to_menu()`.

### Track 2: Power-Up Pickup Scene
- Create `scenes/entities/power_up.tscn` and `scripts/entities/power_up.gd` per design doc section 7.
- Area3D on layer 6 (triggers), mask 2 (player).
- Floating mesh with billboard icon texture, colored OmniLight3D, bob + rotate animation.
- On body_entered (player): call `GameState.activate_buff(powerup_id)`, play pickup SFX, queue_free.
- Lifetime timer: 15s, then fade out and despawn.
- `@export var powerup_id: String` to configure which type.

### Track 3: Drop System
- In `skeleton_enemy.gd` `_die()`: roll `randf() < WaveConfig.powerup_drop_chance`.
- If drop: pick random type from `["powerup_double_jump", "powerup_regen", "powerup_attack_speed"]`.
- Instance power-up scene, set `powerup_id`, position at skeleton death location +0.5 Y.
- Add to "PowerUps" node in map (create if missing). Enforce `powerup_max_on_ground`.

### Track 4: Buff Effects in Player
- **Double Jump:** Track `_extra_jumps_remaining`. On jump input while airborne + remaining > 0: apply jump velocity, decrement. On landing: reset to 1 if buff active.
- **Regeneration:** In `_physics_process`, if `GameState.has_buff("powerup_regen")`: add `WaveConfig.powerup_regen_rate * delta` to `GameState.player_health`.
- **Attack Speed:** When buff active, scale `attack_cooldown.wait_time` and `ATTACK_LOCK_DURATION` by `WaveConfig.powerup_attack_speed_multiplier`. Restore on expiry. Listen to `buff_changed` signal.

### Track 5: HUD Buff Display
- Add a horizontal HBoxContainer above the health bar in `hud.tscn`.
- On `buff_changed` signal: add/remove TextureRect icons with timer labels.
- Show remaining seconds as overlay or label below icon.

### Track 6: Tuning Preparation
- Ensure ALL gameplay constants use `@export` in `wave_config.gd`:
  - Player: health, move speed, jump velocity, attack damage, attack range, attack hitbox, attack cooldown, attack lock, block damage reduction.
  - Skeleton: health, move speed, attack range, attack damage, attack cooldown.
  - Wave: ramp constant, intervals, delays, max alive caps.
  - Power-ups: drop chance, duration, regen rate, attack speed multiplier, pickup radius, lifetime.
- Confirm all these values are editable from the Godot inspector by selecting the WaveConfig autoload node.
- Do NOT change any balance values — the player will tune manually.

---

## Hard Constraints

1. Godot 4.6.1, GDScript only.
2. Do NOT change balance numbers — only expose them as `@export`.
3. Icon assets are SVG files already created — do not regenerate.
4. Power-up pickup uses Area3D collision (layer 6 triggers, mask 2 player).
5. All new constants in `wave_config.gd`, all new state in `game_state.gd`.
6. KayKit models face +Z — same rotation conventions as existing code.

---

## Quality Gates

Do not mark complete until:
- Skeletons drop glowing pickups ~22% of the time on death.
- Each pickup type works: double jump allows one air jump, regen heals over time, attack speed visibly speeds up swings.
- HUD shows active buff icons with countdown timers.
- Buffs expire after 30s and effects are removed cleanly.
- Picking up same buff refreshes timer.
- All constants are `@export` in wave_config.gd.
- No script errors during full play session with pickups.

---

## Definition of Done

Cycle 3 is complete when:
- Power-up drop, pickup, and buff system fully functional.
- All three buff types work correctly with visual feedback.
- HUD displays active buffs.
- All tuning constants exposed as `@export` for manual adjustment.
- Codebase clean and consistent.
