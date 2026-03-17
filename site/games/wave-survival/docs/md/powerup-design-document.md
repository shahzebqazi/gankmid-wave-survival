# Power-Up Design Document

**Date:** 2026-03-17  
**Project:** Dungeon Prototype — Wave Survival  
**Status:** Designed, implementation in Cycle 3  
**Depends on:** Cycle 2 (bug fixes, audio, maps)

---

## 1) Overview

Skeletons have a 22% chance to drop a power-up on death. Power-ups are timed buffs (30 seconds). Only one instance of each type can be active at a time — picking up a second of the same type refreshes the timer. Power-ups appear as glowing pickups at the skeleton's death position.

---

## 2) Power-Up Definitions

| ID | Name | Icon | Duration | Effect | Visual Cue |
|----|------|------|----------|--------|------------|
| `powerup_double_jump` | Double Jump | `res://assets/icons/powerup_double_jump.svg` | 30s | Player can jump once more while airborne (one extra jump per airborne phase). Resets on landing. | Cyan glow, up-arrows |
| `powerup_regen` | Regeneration | `res://assets/icons/powerup_regen.svg` | 30s | Player regenerates +2 HP/sec. Stops if health reaches 100. | Green glow, plus/cross |
| `powerup_attack_speed` | Attack Speed | `res://assets/icons/powerup_attack_speed.svg` | 30s | Player attack cooldown reduced by 20% (0.5s → 0.4s), attack lock duration reduced by 20% (0.5s → 0.4s). | Red glow, speed arrows |

---

## 3) Drop Mechanics

- **Drop chance:** 22% per skeleton death, rolled once on `_die()`.
- **Drop selection:** Equal weight (1/3 each) among the three types.
- **Drop position:** Skeleton's `global_position` at moment of death, raised +0.5 Y to float above ground.
- **Pickup lifetime:** 15 seconds. If not picked up, fades out and despawns.
- **Pickup radius:** 1.5m — player walks into it, auto-collects (Area3D trigger on layer 6, mask 2).
- **Max drops on ground:** 5 (oldest despawns if exceeded).

---

## 4) Active Buff Tracking

Tracked in `GameState` or a new `PowerUpManager` autoload:

```
var active_buffs: Dictionary = {}  # { "powerup_id": remaining_seconds }
```

- Tick down in `_process()` while `Phase.PLAYING`.
- When timer hits 0, remove buff.
- Picking up a duplicate refreshes the timer to 30s.
- On game over or return to menu, clear all buffs.

---

## 5) HUD Integration

- Active buffs shown as a horizontal row of icons above the health bar (bottom-left).
- Each icon shows the power-up SVG with a radial timer overlay (pie chart countdown) or a numeric seconds remaining.
- Icons appear on pickup, disappear on expiry.
- Max 3 icons (one per type).

---

## 6) Player Script Changes

### Double Jump
- Track `_extra_jumps_remaining: int` (0 normally, 1 when buff active).
- On jump input while airborne AND `_extra_jumps_remaining > 0`: allow jump, decrement counter.
- On landing: reset `_extra_jumps_remaining` to 1 if buff active, else 0.

### Regeneration
- In `_physics_process`, if buff active and `GameState.player_health < 100`:
  - `GameState.player_health += 2.0 * delta`

### Attack Speed
- Multiply `ATTACK_LOCK_DURATION` and `attack_cooldown.wait_time` by 0.8 when buff active.
- Restore to 1.0 when buff expires.

---

## 7) Pickup Scene Structure

```
PowerUp (Area3D) — layer 6, mask 2
├── CollisionShape3D (SphereShape3D radius 1.5)
├── MeshInstance3D (small quad or sphere with icon texture, billboard mode)
├── OmniLight3D (color matches power-up, energy 0.5, range 3)
├── Timer "LifetimeTimer" (one_shot, 15s)
└── AnimationPlayer (bob + rotate loop)

Script: res://scripts/entities/power_up.gd
```

---

## 8) Power-Up Constants (add to `wave_config.gd`)

```gdscript
@export_group("Power-Ups")
@export var powerup_drop_chance: float = 0.22
@export var powerup_duration: float = 30.0
@export var powerup_pickup_radius: float = 1.5
@export var powerup_lifetime: float = 15.0
@export var powerup_max_on_ground: int = 5
@export var powerup_regen_rate: float = 2.0
@export var powerup_attack_speed_multiplier: float = 0.8
```

---

## 9) Icon Assets

Pre-created SVG icons at:
- `project/assets/icons/powerup_double_jump.svg` — cyan circle, double up-arrows
- `project/assets/icons/powerup_regen.svg` — green circle, plus/cross with healing aura
- `project/assets/icons/powerup_attack_speed.svg` — red circle, speed arrows

These are 64x64 SVGs. Godot imports SVGs natively as textures.

---

## 10) Tuning Note

All damage, range, health, regen, hitbox, and timing values are designed as starting points. The player will tune these values manually after the system is implemented. All constants live in `wave_config.gd` with `@export` for editor tweaking.
