# Zombies Wave Map Mockups Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a single webpage that presents three wave-survival map mockups with floor plans and vertical level diagrams for 1, 2, and 3 enemy entrances.

**Architecture:** Create one standalone HTML document in `design-docs` with embedded CSS and SVG diagrams. Use card-based sections per map variant and include mixed-neutral texture references from `kenney_prototypeTextures`. Keep the page static and portable (no external dependencies).

**Tech Stack:** HTML5, CSS3, inline SVG, small vanilla JavaScript for view toggles.

---

### Task 1: Create page shell and visual system

**Files:**
- Create: `design-docs/zombies-wave-map-mockups.html`

**Step 1: Build page skeleton**
- Add title, hero section, and legend.
- Add sections for Map A, B, and C.

**Step 2: Add visual language**
- Define mixed-neutral palette tokens.
- Add texture swatches tied to:
  - `../kenney_prototypeTextures/PNG/Dark/texture_01.png`
  - `../kenney_prototypeTextures/PNG/Light/texture_01.png`
  - `../kenney_prototypeTextures/PNG/Purple/texture_10.png`
  - `../kenney_prototypeTextures/PNG/Orange/texture_04.png`
  - `../kenney_prototypeTextures/PNG/Red/texture_03.png`

### Task 2: Add floor plans and vertical levels

**Files:**
- Modify: `design-docs/zombies-wave-map-mockups.html`

**Step 1: Build floor-plan SVGs**
- Map A: single enemy entrance funnel.
- Map B: dual entrance pinch.
- Map C: triple-front rotation loop.

**Step 2: Build vertical-level SVGs**
- Show `L0`, `L1`, and optional `L2`.
- Show stairs/ramps and fallback routes.

### Task 3: Add gameplay annotations and interactions

**Files:**
- Modify: `design-docs/zombies-wave-map-mockups.html`

**Step 1: Add labels**
- Mark spawn, power point, holdout node, training loop, and breach lanes.

**Step 2: Add simple controls**
- Toggle between "Floor Plan" and "Vertical Levels" per map.
- Keep interactions minimal and accessible.

### Task 4: Validate and handoff

**Files:**
- Modify: `design-docs/zombies-wave-map-mockups.html` (if needed)

**Step 1: Sanity check**
- Open in browser and verify diagrams/labels render.
- Verify texture paths resolve from `design-docs`.

**Step 2: Handoff**
- Provide map intent notes and how to convert mockups into playable Godot blockouts.
