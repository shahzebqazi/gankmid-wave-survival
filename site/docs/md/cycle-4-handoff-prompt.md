# Cycle 4: Skybox Art and Visual Polish Handoff Prompt

Use this prompt after Cycle 3 is complete (power-ups working, tuning constants exposed).

---

## Prompt

You are a **Godot 4.6 game development agent** continuing work on a wave-survival dungeon prototype. Cycle 3 delivered power-ups and tuning prep. Your job in Cycle 4 is to integrate custom skybox art the player has downloaded and apply visual polish to the maps.

### Context

The project is at `/home/sqazi/godot-game/project/` (Godot 4.6.1). Godot is installed at `/usr/bin/godot`.

---

## Required Inputs (read before starting)

### Current codebase
- Read `project/scenes/maps/map_a.tscn`, `map_b.tscn`, `map_c.tscn` — each has a `WorldEnvironment` node with a `ProceduralSkyMaterial`.
- Read `project/project.godot` for rendering settings.

### Player-provided art
The player will have downloaded skybox art images before this cycle starts. Ask the player:
1. Where are the skybox image files located?
2. What format are they? (PNG, HDR, EXR, JPG)
3. Are they panoramic (equirectangular) or 6-face cubemap?

---

## Development Tracks

### Track 1: Discover and Import Skybox Art
- Locate the skybox art files the player has downloaded.
- Copy them to `project/assets/textures/skybox/`.
- Determine format:
  - **Equirectangular panorama** (single wide image, 2:1 ratio): Use as `PanoramaSkyMaterial.panorama` texture.
  - **6-face cubemap** (6 separate images: px, nx, py, ny, pz, nz): Compose into a Godot cubemap resource.
  - **HDR/EXR**: Import as-is — Godot handles HDR natively for sky materials.
  - **JPG/PNG**: Import as CompressedTexture2D, use with `PanoramaSkyMaterial`.

### Track 2: Replace Procedural Sky
- For each map scene (`map_a.tscn`, `map_b.tscn`, `map_c.tscn`):
  - Replace the `ProceduralSkyMaterial` sub_resource with a `PanoramaSkyMaterial`.
  - Set the `panorama` property to the imported skybox texture.
  - Adjust `Sky.sky_material` reference.
  - Keep ambient light and fog settings but tune them to match the new sky's color temperature.

### Track 3: Per-Map Sky Variants (if multiple art files)
- If the player provides different skybox images for different maps:
  - Map A: assign skybox A
  - Map B: assign skybox B
  - Map C: assign skybox C
- If only one image: use it for all maps.

### Track 4: Lighting Adjustment
- After applying the skybox, adjust the `DirectionalLight3D` color and energy to match the sky mood.
- Adjust `Environment.ambient_light_color` to complement the new sky.
- Adjust fog color to harmonize with the skybox horizon.
- Test each map visually — characters and greybox geometry should read clearly against the new sky.

### Track 5: Optional Visual Polish
- Add `Environment.glow_enabled = true` with subtle bloom for the power-up pickups and OmniLight3D sources.
- Add subtle `Environment.volumetric_fog` if performance allows (low density, short range).
- Adjust `Environment.tonemap_mode` to ACES or Filmic for richer sky rendering.
- These are optional — only apply if they look good and don't hurt performance.

---

## Hard Constraints

1. Godot 4.6.1, GDScript only (no shader code unless necessary for sky blending).
2. Do not delete or modify the player's original art files — only copy into project.
3. Keep fog and ambient light functional — they serve gameplay readability per the PRD "Readable Combat Space" pillar.
4. Skybox changes must not affect navigation mesh, collision, or gameplay systems.
5. If the player hasn't downloaded art yet, stop and ask before proceeding.

---

## Quality Gates

Do not mark complete until:
- Skybox art is visible in all map scenes.
- Lighting matches the sky mood (no jarring contrast between sky and directional light).
- Characters and geometry remain clearly readable against the new sky.
- No rendering errors or pink/missing textures.
- Performance is stable (no FPS drop from sky material).

---

## Definition of Done

Cycle 4 is complete when:
- Custom skybox art integrated into all map scenes.
- Lighting and fog adjusted to complement the skybox.
- Visual quality noticeably improved over procedural sky.
- No regressions to gameplay or performance.
