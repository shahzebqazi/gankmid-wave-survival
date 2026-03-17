# Audio Design Document: Sound & Music

**Date:** 2026-03-17  
**Project:** Dungeon Prototype — Wave Survival  
**Status:** Sourced — all sounds identified from Freesound.org (CC0/CC-BY 4.0)

**Companion docs:**
- `design-docs/audio-asset-gallery.html` — interactive gallery with playable previews for every sound
- `docs/plans/2026-03-17-unified-prd-wave-survival.md` — PRD section 10 (audio system architecture)
- `docs/plans/2026-03-17-cycle-2-handoff-prompt.md` — implementation instructions with download procedure

---

## 1) Music

| ID | Track | Usage | Style | Loop | Priority | Freesound ID | Author | License |
|----|-------|-------|-------|------|----------|-------------|--------|---------|
| `mus_menu` | Main Menu Theme | Main menu background | Dark ambient synth, low BPM, brooding dungeon atmosphere | Yes | P0 | [516566](https://freesound.org/s/516566/) | Kinoton | CC0 |
| `mus_combat_01` | Combat Loop A | In-game during active waves | Driving percussion, minor key, escalating tension, 120-140 BPM | Yes | P0 | [138681](https://freesound.org/s/138681/) | haydensayshi123 | CC0 |
| `mus_wave_clear` | Wave Clear Sting | Plays on wave_cleared signal, crossfades back to combat loop | Short triumphant brass/strings sting, 2-4 seconds | No | P1 | [706753](https://freesound.org/s/706753/) | xkeril | CC0 |
| `mus_game_over` | Game Over Sting | Plays on game_over | Descending somber melody, 3-5 seconds | No | P1 | [270467](https://freesound.org/s/270467/) | LittleRobotSoundFactory | CC-BY 4.0 |

---

## 2) Player Sound Effects

| ID | File | Trigger | Description | Freesound ID | Author | License |
|----|------|---------|-------------|-------------|--------|---------|
| `sfx_player_footstep_01` | `footstep_01.ogg` | Every ~0.35s while Running_A plays and grounded | Stone/dirt footstep, soft, alternating L/R | [208103](https://freesound.org/s/208103/) | Phil25 | CC0 |
| `sfx_player_footstep_02` | `footstep_02.ogg` | Alternated with footstep_01 | Slight variant for natural feel | [712084](https://freesound.org/s/712084/) | areniporgen | CC0 |
| `sfx_player_attack_swing` | `sword_swing.ogg` | On attack_window start (0.3s into attack) | Metallic whoosh, medium pitch | [700220](https://freesound.org/s/700220/) | Artninja | CC-BY 4.0 |
| `sfx_player_attack_hit` | `sword_hit.ogg` | When take_damage is called on an enemy by player | Meaty impact + slight clang | [547036](https://freesound.org/s/547036/) | CogFireStudios | CC0 |
| `sfx_player_block` | `shield_block.ogg` | When player takes damage while blocking | Heavy metallic clang, shield rattle | [97792](https://freesound.org/s/97792/) | CGEffex | CC-BY 4.0 |
| `sfx_player_hurt` | `player_hurt.ogg` | When player takes unblocked damage | Short grunt/pain vocalization | [561555](https://freesound.org/s/561555/) | luminousfridge | CC0 |
| `sfx_player_jump` | `jump.ogg` | On jump input | Soft cloth whoosh + effort grunt | [465299](https://freesound.org/s/465299/) | SoftDistortionFX | CC0 |
| `sfx_player_land` | `land.ogg` | On landing (was_on_floor transition) | Thud on stone | [187136](https://freesound.org/s/187136/) | 000600 | CC0 |

---

## 3) Skeleton Sound Effects

| ID | File | Trigger | Description | Freesound ID | Author | License |
|----|------|---------|-------------|-------------|--------|---------|
| `sfx_skeleton_spawn` | `skeleton_spawn.ogg` | On Skeletons_Spawn_Ground animation start | Rattling bones emerging, brief | [202102](https://freesound.org/s/202102/) | spookymodem | CC0 |
| `sfx_skeleton_footstep` | `skeleton_step.ogg` | Every ~0.5s while Skeletons_Walking plays | Light bone/claw on stone, quieter than player | [381859](https://freesound.org/s/381859/) | cribbler | CC0 |
| `sfx_skeleton_attack` | `skeleton_attack.ogg` | On _try_attack | Sharp bone scrape/swipe | [815763](https://freesound.org/s/815763/) | xkeril | CC0 |
| `sfx_skeleton_hit` | `skeleton_hit.ogg` | On take_damage (non-lethal) | Bone crack + rattle | [442769](https://freesound.org/s/442769/) | qubodup | CC0 |
| `sfx_skeleton_death` | `skeleton_death.ogg` | On _die | Bone collapse scatter, rattling fade | [202091](https://freesound.org/s/202091/) | spookymodem | CC0 |
| `sfx_skeleton_aggro` | `skeleton_aggro.ogg` | On first transition from SPAWNING to CHASE | Low undead groan/hiss | [416060](https://freesound.org/s/416060/) | giddster | CC-BY 4.0 |

---

## 4) UI Sound Effects

| ID | File | Trigger | Description | Freesound ID | Author | License |
|----|------|---------|-------------|-------------|--------|---------|
| `sfx_ui_navigate` | `ui_navigate.ogg` | Menu focus change (button hover) | Soft click/tick | [253168](https://freesound.org/s/253168/) | suntemple | CC0 |
| `sfx_ui_confirm` | `ui_confirm.ogg` | Menu button press (Start Game, Resume) | Crisp select chime | [220206](https://freesound.org/s/220206/) | GameAudio | CC0 |
| `sfx_ui_back` | `ui_back.ogg` | ESC / back action | Soft reverse click | [366104](https://freesound.org/s/366104/) | Breviceps | CC0 |
| `sfx_ui_pause` | `ui_pause.ogg` | Pause menu open | Low rumble + mute effect | (trim 516566) | Kinoton | CC0 |

---

## 5) Ambient / Environment

| ID | File | Trigger | Description | Freesound ID | Author | License |
|----|------|---------|-------------|-------------|--------|---------|
| `sfx_ambient_dungeon` | `ambient_dungeon.ogg` | Constant background during gameplay | Wind through stone, distant drips, low hum. Very quiet. | [516566](https://freesound.org/s/516566/) | Kinoton | CC0 |
| `sfx_wave_start` | `wave_horn.ogg` | On wave_started signal | Distant war horn or bell toll, announces danger | [512490](https://freesound.org/s/512490/) | DeVern | CC0 |

---

## 6) Audio Bus Layout

| Bus | Purpose | Default Volume |
|-----|---------|---------------|
| Master | Global output | 0 dB |
| Music | All mus_ tracks | -6 dB |
| SFX | All sfx_ tracks | -3 dB |
| UI | All sfx_ui_ tracks | -3 dB |
| Ambient | sfx_ambient_ tracks | -12 dB |

---

## 7) Implementation Notes

- All sound files should be `.ogg` (Ogg Vorbis) for Godot import efficiency.
- Music tracks use `AudioStreamPlayer` (non-positional, global).
- Player/skeleton SFX use `AudioStreamPlayer3D` attached to their respective scenes for spatial audio.
- UI SFX use `AudioStreamPlayer` (non-positional).
- Footstep timing should be driven by animation events or a timer synced to movement speed, not frame-based.
- Volume ducking: reduce `Music` bus by -6 dB during pause menu.
- All audio should be sourced from free/CC0 packs or generated. Recommended sources: Kenney Audio, Freesound.org, OpenGameArt.

---

## 7.5) Options Menu — Volume Sliders

The Options menu (accessible from Main Menu and Pause Menu) provides real-time volume control.

### Slider Spec

| Slider | Maps to Bus | Range | Default | Godot API |
|--------|------------|-------|---------|-----------|
| Master Volume | `Master` | 0–100 | 80 | `AudioServer.set_bus_volume_db(0, linear_to_db(v/100))` |
| Music Volume | `Music` | 0–100 | 70 | `AudioServer.set_bus_volume_db(idx, linear_to_db(v/100))` |
| SFX Volume | `SFX` | 0–100 | 80 | `AudioServer.set_bus_volume_db(idx, linear_to_db(v/100))` |

- UI and Ambient buses are derived from SFX: UI tracks SFX at 1:1, Ambient tracks SFX at 0.5x.
- Changes apply immediately for real-time preview.
- Play `sfx_ui_navigate` on slider value change as audible feedback.
- "Reset Defaults" button restores all sliders to defaults.
- Settings persist to `user://settings.cfg` via `ConfigFile`.
- Mockup: see `design-docs/menu-hud-mockups.html` → Options Menu panel (interactive, drag sliders).

### Scene Structure

```
OptionsMenu (Control)
├── PanelContainer
│   ├── Title: "OPTIONS"
│   ├── MasterSlider (HSlider) + Label "Master Volume" + ValueLabel
│   ├── MusicSlider (HSlider) + Label "Music Volume" + ValueLabel
│   ├── SFXSlider (HSlider) + Label "SFX Volume" + ValueLabel
│   ├── FullscreenToggle (CheckButton)
│   ├── ResetButton (Button) "Reset Defaults"
│   └── BackButton (Button) "Back"
```

### Settings Persistence

```gdscript
# Save
var config := ConfigFile.new()
config.set_value("audio", "master", master_slider.value)
config.set_value("audio", "music", music_slider.value)
config.set_value("audio", "sfx", sfx_slider.value)
config.save("user://settings.cfg")

# Load (in AudioManager._ready())
var config := ConfigFile.new()
if config.load("user://settings.cfg") == OK:
    set_bus_volume("Master", config.get_value("audio", "master", 80))
    set_bus_volume("Music", config.get_value("audio", "music", 70))
    set_bus_volume("SFX", config.get_value("audio", "sfx", 80))
```

---

## 7.6) AudioManager Autoload

Register as autoload: `AudioManager` → `res://scripts/autoload/audio_manager.gd`

### API

```gdscript
# Music
func play_music(stream_id: String) -> void     # crossfade to new track
func stop_music() -> void                       # fade out current track

# Stings (ducks music, plays one-shot, restores)
func play_sting(stream_id: String) -> void

# UI SFX (non-positional)
func play_sfx_ui(stream_id: String) -> void

# Bus control
func set_bus_volume(bus_name: String, linear_0_100: float) -> void
func load_settings() -> void                    # load from user://settings.cfg
func save_settings() -> void                    # persist to user://settings.cfg
```

### Audio Stream Registry

AudioManager holds a `Dictionary` mapping sound IDs to preloaded `AudioStream` resources:

```gdscript
var _streams: Dictionary = {
    "mus_menu": preload("res://assets/audio/music/menu_theme.ogg"),
    "mus_combat_01": preload("res://assets/audio/music/combat_loop_01.ogg"),
    "sfx_ui_confirm": preload("res://assets/audio/sfx/ui/ui_confirm.ogg"),
    # ... all 24 entries
}
```

---

## 8) File Structure

```
project/assets/audio/
├── music/
│   ├── menu_theme.ogg
│   ├── combat_loop_01.ogg
│   ├── wave_clear_sting.ogg
│   └── game_over_sting.ogg
├── sfx/
│   ├── player/
│   │   ├── footstep_01.ogg
│   │   ├── footstep_02.ogg
│   │   ├── sword_swing.ogg
│   │   ├── sword_hit.ogg
│   │   ├── shield_block.ogg
│   │   ├── player_hurt.ogg
│   │   ├── jump.ogg
│   │   └── land.ogg
│   ├── skeleton/
│   │   ├── skeleton_spawn.ogg
│   │   ├── skeleton_step.ogg
│   │   ├── skeleton_attack.ogg
│   │   ├── skeleton_hit.ogg
│   │   ├── skeleton_death.ogg
│   │   └── skeleton_aggro.ogg
│   ├── ui/
│   │   ├── ui_navigate.ogg
│   │   ├── ui_confirm.ogg
│   │   ├── ui_back.ogg
│   │   └── ui_pause.ogg
│   └── environment/
│       ├── ambient_dungeon.ogg
│       └── wave_horn.ogg
```

---

## 9) Attribution Requirements

CC-BY 4.0 sounds require attribution in the game credits or a `LICENSES.txt` file. Full attribution template is available in `design-docs/audio-asset-gallery.html` (Attribution section at bottom).

**CC-BY 4.0 sounds used:**
- `mus_game_over` — Jingle_Lose_00 by LittleRobotSoundFactory
- `sfx_player_attack_swing` — Sword_whoosh_sound by Artninja
- `sfx_player_block` — Metal Impact by CGEffex
- `sfx_skeleton_aggro` — Death by Zombie by giddster

All other sounds are CC0 (no attribution required).
