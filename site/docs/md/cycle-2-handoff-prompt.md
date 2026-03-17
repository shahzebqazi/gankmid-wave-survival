# Cycle 2: Polish, Audio, and Expansion Handoff Prompt

Use this prompt to run the next development cycle after the MVP vertical slice.

---

## Prompt

You are a **Godot 4.6 game development agent** continuing work on a wave-survival dungeon prototype. The MVP vertical slice is complete and playable. Your job is to review the current codebase, fix known issues, integrate audio, and expand toward the full vertical slice scope.

### Context

The project is at `/home/sqazi/godot-game/project/` (Godot 4.6.1). Godot is installed at `/usr/bin/godot`.

---

## Required Inputs (read all before starting)

### Planning docs
- `docs/plans/2026-03-17-unified-prd-wave-survival.md` — canonical PRD with wave algorithm, constants, functional requirements
- `docs/plans/game-design-document.md` — original GDD

### Design docs
- `design-docs/audio-design-document.md` — **NEW** full audio spec with sound IDs, triggers, bus layout, file structure, and Freesound source mappings
- `design-docs/audio-asset-gallery.html` — **NEW** interactive gallery with playable Freesound previews for every sound; open in browser to audition
- `design-docs/zombies-wave-map-mockups.html` — map floor plans for all 3 maps
- `design-docs/menu-hud-mockups.html` — UI mockup reference (includes Options menu with volume slider mockups)

### Current codebase (read all scripts)
```
project/
├── project.godot
├── scenes/
│   ├── entities/player.tscn, skeleton_enemy.tscn
│   ├── maps/map_a.tscn
│   └── ui/main_menu.tscn, hud.tscn, pause_menu.tscn
├── scripts/
│   ├── autoload/game_state.gd, wave_config.gd
│   ├── entities/player.gd, skeleton_enemy.gd
│   ├── maps/map_base.gd
│   ├── systems/entrance_marker.gd, wave_director.gd
│   └── ui/main_menu.gd, hud.gd, pause_menu.gd
└── assets/
    ├── characters/adventurers/ (6 GLB characters)
    ├── characters/skeletons/ (4 GLB skeletons)
    ├── animations/ (8 GLB animation packs)
    ├── weapons/ (sword_1handed.gltf, shield_round.gltf + knight_texture.png)
    ├── textures/prototype/ (Kenney prototype textures, symlinked)
    └── fonts/monogram.ttf (symlinked)
```

### Asset packs available
- `KayKit_Adventurers_2.0_FREE` — 6 playable characters + weapons + basic animations
- `KayKit_Skeletons_1.1_FREE` — 4 skeleton variants + skeleton-specific animations
- `KayKit_Character_Animations_1.1` — full animation library (combat, movement, special, tools)
- `kenney_prototypeTextures` — greybox textures (Dark, Light, Green, Red, Orange, Purple)
- `monogram/` — pixel font

---

## Known Issues To Fix First

These are confirmed bugs from MVP playtesting. Fix them before adding features:

1. **Animation root_node inconsistency** — `player.gd` uses default `root_node` (`..`) while `skeleton_enemy.gd` explicitly sets `root_node = get_path_to(instance)`. Both should use the same approach. Verify animations actually play on the character mesh (bones move) by testing in-game. If they don't, the root_node is wrong.

2. **KayKit model orientation** — KayKit models face +Z (Blender convention). Godot's forward is -Z. The current code uses `atan2(direction.x, direction.z)` for yaw which naturally points models toward +Z (correct for the mesh). The attack hitbox uses `model.basis.z` (model's +Z forward). Verify this is consistent: the character should visually face the direction of movement AND the attack should hit in front of the visual character.

3. **Weapon texture loading** — Weapons were changed from symlinks to real file copies in `assets/weapons/`. Verify `knight_texture.png` is found by the GLTF importer and the sword/shield render with textures, not white/pink.

4. **Death tween "det == 0" error** — The skeleton death tween scales to `Vector3.ZERO` which causes a basis inversion error. Scale to `Vector3(0.01, 0.01, 0.01)` instead.

5. **Nav mesh warning** — `map_base.gd` bakes nav mesh from visual meshes at runtime. Switch to using collision shapes as source geometry by setting `NavigationMesh.geometry_parsed_geometry_type` to `PARSED_GEOMETRY_STATIC_COLLIDERS`.

---

## Development Tracks (execute in order)

### Track 1: Bug Fixes and Code Cleanup
- Fix all 5 known issues above.
- Ensure all scripts use consistent patterns (animation loading, model orientation).
- Test: run game, verify movement direction matches visual facing, attack hits forward, animations play and loop.

### Track 2: Audio Integration
- Read `design-docs/audio-design-document.md` fully — it contains Freesound source IDs for every sound.
- Create the audio bus layout in Godot (Master, Music, SFX, UI, Ambient) per section 6 of the audio doc.
- Create an `AudioManager` autoload (`scripts/autoload/audio_manager.gd`) for centralized music/sting playback with crossfade and bus volume control.
- **Download audio files** from Freesound.org using the IDs below. Convert all to `.ogg` (Ogg Vorbis) and place in `project/assets/audio/` per the file structure in section 8 of the audio doc.

#### Audio File Source Map (Freesound IDs)

| Target File | Freesound ID | Author | License |
|-------------|-------------|--------|---------|
| `music/menu_theme.ogg` | [516566](https://freesound.org/s/516566/) | Kinoton | CC0 |
| `music/combat_loop_01.ogg` | [138681](https://freesound.org/s/138681/) | haydensayshi123 | CC0 |
| `music/wave_clear_sting.ogg` | [706753](https://freesound.org/s/706753/) | xkeril | CC0 |
| `music/game_over_sting.ogg` | [270467](https://freesound.org/s/270467/) | LittleRobotSoundFactory | CC-BY 4.0 |
| `sfx/player/footstep_01.ogg` | [208103](https://freesound.org/s/208103/) | Phil25 | CC0 |
| `sfx/player/footstep_02.ogg` | [712084](https://freesound.org/s/712084/) | areniporgen | CC0 |
| `sfx/player/sword_swing.ogg` | [700220](https://freesound.org/s/700220/) | Artninja | CC-BY 4.0 |
| `sfx/player/sword_hit.ogg` | [547036](https://freesound.org/s/547036/) | CogFireStudios | CC0 |
| `sfx/player/shield_block.ogg` | [97792](https://freesound.org/s/97792/) | CGEffex | CC-BY 4.0 |
| `sfx/player/player_hurt.ogg` | [561555](https://freesound.org/s/561555/) | luminousfridge | CC0 |
| `sfx/player/jump.ogg` | [465299](https://freesound.org/s/465299/) | SoftDistortionFX | CC0 |
| `sfx/player/land.ogg` | [187136](https://freesound.org/s/187136/) | 000600 | CC0 |
| `sfx/skeleton/skeleton_spawn.ogg` | [202102](https://freesound.org/s/202102/) | spookymodem | CC0 |
| `sfx/skeleton/skeleton_step.ogg` | [381859](https://freesound.org/s/381859/) | cribbler | CC0 |
| `sfx/skeleton/skeleton_attack.ogg` | [815763](https://freesound.org/s/815763/) | xkeril | CC0 |
| `sfx/skeleton/skeleton_hit.ogg` | [442769](https://freesound.org/s/442769/) | qubodup | CC0 |
| `sfx/skeleton/skeleton_death.ogg` | [202091](https://freesound.org/s/202091/) | spookymodem | CC0 |
| `sfx/skeleton/skeleton_aggro.ogg` | [416060](https://freesound.org/s/416060/) | giddster | CC-BY 4.0 |
| `sfx/ui/ui_navigate.ogg` | [253168](https://freesound.org/s/253168/) | suntemple | CC0 |
| `sfx/ui/ui_confirm.ogg` | [220206](https://freesound.org/s/220206/) | GameAudio | CC0 |
| `sfx/ui/ui_back.ogg` | [366104](https://freesound.org/s/366104/) | Breviceps | CC0 |
| `sfx/ui/ui_pause.ogg` | (trim from 516566) | Kinoton | CC0 |
| `sfx/environment/ambient_dungeon.ogg` | [516566](https://freesound.org/s/516566/) | Kinoton | CC0 |
| `sfx/environment/wave_horn.ogg` | [512490](https://freesound.org/s/512490/) | DeVern | CC0 |

#### Download procedure
1. For each Freesound ID, download via: `wget "https://freesound.org/people/AUTHOR/sounds/ID/download/ID__author__name.wav"` (requires Freesound login) OR use `freesound-client` CLI OR download manually from the browser.
2. Convert to `.ogg`: `ffmpeg -i input.wav -c:a libvorbis -q:a 5 output.ogg`
3. For multi-hit recordings (footsteps, sword_swing), trim to a single hit: `ffmpeg -i input.wav -ss START -t DURATION -c:a libvorbis -q:a 5 output.ogg`
4. Place files in `project/assets/audio/` per the file structure in the audio doc.

- Wire sound triggers into existing scripts per the trigger column in the audio doc.
- Add `AudioStreamPlayer3D` nodes to player and skeleton scenes for positional SFX.
- Add `AudioStreamPlayer` to UI scripts for menu sounds.
- Music: play `mus_menu` on main menu, crossfade to `mus_combat_01` on game start, play stings on wave_cleared and game_over.

### Track 2.5: Options Menu (Audio Settings)
- Create `scenes/ui/options_menu.tscn` with volume sliders per the mockup in `design-docs/menu-hud-mockups.html`.
- 3 horizontal sliders: Master Volume (default 80), Music Volume (default 70), SFX Volume (default 80).
- Slider changes call `AudioManager.set_bus_volume()` in real-time.
- Play `sfx_ui_navigate` on slider value change for audible feedback.
- Add "Reset Defaults" button and "Back" button (plays `sfx_ui_back`).
- Persist settings to `user://settings.cfg` using `ConfigFile`.
- Load saved settings on game launch in `AudioManager._ready()`.
- Wire the "Options" button in both `main_menu.gd` and `pause_menu.gd` to show this menu (currently prints "not yet implemented").

### Track 3: Character Select Screen
- Replace the stubbed "Start Game → Knight" flow with an actual character selection screen.
- Show all 6 adventurer models in a carousel/grid.
- Player picks one, then starts on Map A.
- Use `GameState.selected_character` to drive which GLB loads.

### Track 4: Map B and Map C
- Build `map_b.tscn` (Dual-Pinch Warehouse, 2 entrances) per mockup.
- Build `map_c.tscn` (Triple-Front Atrium, 3 entrances) per mockup.
- Add map selection to the character select or main menu flow.
- Reuse `map_base.gd` and `WaveDirector` with `num_entrances=2/3`.

### Track 5: Game Over Screen and Restart
- When `GameState.game_over()` fires, show a Game Over overlay with stats (wave reached, time survived, enemies killed).
- Options: Restart (same map/character), Return to Menu.
- Track `enemies_killed` counter in GameState.

---

## Hard Constraints

1. Godot 4.6.1, GDScript only.
2. All audio files must be `.ogg` format.
3. Do not modify asset pack source files — only use files in `project/assets/`.
4. Keep the wave algorithm unchanged — it is validated and correct per PRD section 6.
5. All constants must remain in `wave_config.gd` — no magic numbers in gameplay scripts.
6. KayKit models face +Z — account for this in all rotation math.
7. Animations must be imported from separate GLB library files, not embedded in character GLBs (characters have no AnimationPlayer). Set `loop_mode = LOOP_LINEAR` on locomotion/idle animations.
8. Audio files must be sourced from Freesound.org using the IDs in the audio source map table above. Do not generate placeholder tones.
9. Options menu volume sliders must use `AudioServer.set_bus_volume_db()` with `linear_to_db()` conversion. Settings must persist via `ConfigFile` to `user://settings.cfg`.

---

## Quality Gates

Do not mark complete until:
- All 5 known bugs are fixed and verified.
- Audio plays for at minimum: menu music, combat music, sword swing, skeleton death, UI confirm, wave horn.
- Options menu has working volume sliders that persist across sessions.
- Character select shows all 6 characters and loads the selected one.
- Map A, B, and C all run the wave algorithm correctly.
- Game over screen appears and restart works.
- No script errors in the Godot output console during a full play session.

---

## Definition of Done

Cycle 2 is complete when:
- Bug fixes verified by playtest.
- Audio system is wired and playing with real Freesound-sourced sounds (not placeholder tones).
- Options menu with Master/Music/SFX sliders is functional from both main menu and pause menu.
- Audio settings persist to `user://settings.cfg`.
- All 3 maps playable with the wave algorithm.
- Character select functional.
- Game over → restart loop works.
- CC-BY attribution is included in a `LICENSES.txt` or credits screen.
- Codebase is clean, consistent, and documented.
