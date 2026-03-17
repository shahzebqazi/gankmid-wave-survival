# MVP Status Report

**Date:** 2026-03-17  
**Build:** Cycle 1 — Vertical Slice MVP  
**Verdict:** PLAYABLE — ready for Cycle 2

---

## What Was Built

| Component | Status | Files |
|-----------|--------|-------|
| Godot project scaffold | Done | `project.godot`, folder structure, asset symlinks |
| Main menu | Done | `main_menu.tscn`, `main_menu.gd` |
| Pause menu | Done | `pause_menu.tscn`, `pause_menu.gd` |
| HUD | Done | `hud.tscn`, `hud.gd` — wave, timer, enemies, health, stamina, debug panel |
| Player controller | Done | `player.tscn`, `player.gd` — WASD, mouse look, jump, attack, block |
| Weapon attachment | Done | Sword + shield via BoneAttachment3D on handslot bones |
| Skeleton enemy | Done | `skeleton_enemy.tscn`, `skeleton_enemy.gd` — SPAWNING/CHASE/ATTACK/DEAD states |
| Wave director | Done | `wave_director.gd` — full PRD algorithm, respawn on kill |
| Entrance system | Done | `entrance_marker.gd` — Marker3D with debug gizmo |
| Map A | Done | `map_a.tscn` — 20x20m arena, 1 entrance, greybox geometry |
| Animation system | Done | Runtime GLB import with loop mode and retargeting |
| Game state | Done | `game_state.gd` — phase machine, signals, timers |
| Wave config | Done | `wave_config.gd` — all PRD constants, pure algorithm functions |

---

## Controls

| Input | Action |
|-------|--------|
| WASD | Move (camera-relative) |
| Mouse | Look (3rd person orbit) |
| Left Click | Attack (sword swing, sphere overlap hitbox) |
| Right Click | Block (75% damage reduction, slow movement) |
| Space | Jump |
| Escape | Pause menu |
| F3 | Debug overlay (wave stats) |

---

## Known Bugs (for Cycle 2)

1. Animation root_node may not resolve correctly — player.gd and skeleton_enemy.gd use different approaches
2. Death tween `Vector3.ZERO` scale causes "det == 0" basis error
3. Nav mesh bakes from visual meshes at runtime (cosmetic warning)
4. Character select not implemented (defaults to Knight)
5. Options menu stubbed (prints to console)
6. No audio at all
7. Maps B and C not built
8. No game over screen

---

## Architecture Notes for Next Developer

**Model orientation:** KayKit models face +Z. `atan2(direction.x, direction.z)` gives correct yaw. Attack forward = `model.basis.z`.

**Animation loading:** Characters have NO embedded AnimationPlayer. Animations come from separate GLB library files (`Rig_Medium_*.glb`). At runtime: create AnimationPlayer, parent to character instance, import animations from libraries with `duplicate()`, set `loop_mode = LOOP_LINEAR` on idle/walk/run anims.

**Wave algorithm:** Fully implemented per PRD section 6. `WaveConfig.build_wave_plan()` returns sorted `Array[Dictionary]` of `{door: int, delay: float}`. WaveDirector processes this with a time accumulator.

**Respawn:** When a skeleton dies it emits `died` signal. WaveDirector queues a 10-second respawn timer per kill.
