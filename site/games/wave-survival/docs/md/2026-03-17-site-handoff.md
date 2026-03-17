# Wave Survival Site — Handoff

**Date:** 2026-03-17  
**Repo:** shahzebqazi/gankmid-wave-survival  
**Live:** https://dev.gankmid.co/

---

## Site Overview

Design documentation site for the **Dungeon Prototype — Wave Survival** Godot 4.6 game. Hosted on GitHub Pages at dev.gankmid.co.

### Structure

```
site/
├── index.html                         # Landing → games/wave-survival/
├── CNAME                              # dev.gankmid.co
└── games/wave-survival/
    ├── index.html                     # Hub (Play, Design Docs)
    ├── css/style.css                  # Shared styles (responsive)
    ├── play/                          # Godot HTML5 export (placeholder until export)
    │   ├── index.html
    │   └── README.md
    └── docs/
        ├── ui-assets-gallery.html     # 3D models (Planets, Kenney, icons load; KayKit .glb not in repo)
        ├── audio-asset-gallery.html   # Freesound embeds
        ├── menu-hud-mockups.html      # Menu/HUD mockups
        ├── wave-map-mockups.html      # Map floor plans
        ├── audio-design.html          # Renders md/audio-design-document.md
        ├── powerup-design.html         # Renders md/powerup-design-document.md
        ├── references.html            # Planning docs index + external links
        ├── embed/                     # Iframe content (self-contained HTML)
        └── md/                        # Source markdown
```

---

## Link Audit (2026-03-17)

**Result: All internal links verified OK.**

### Menu Bar Links (topbar nav)

| Link | Target | Status |
|------|--------|--------|
| Home | `../index.html` or `index.html` | OK |
| Play | `../play/` or `play/` | OK |
| 3D Assets | `ui-assets-gallery.html` | OK |
| Audio | `audio-asset-gallery.html` | OK |
| Menu & HUD | `menu-hud-mockups.html` | OK |
| Map Mockups | `wave-map-mockups.html` | OK |
| Audio Design | `audio-design.html` | OK |
| Power-Ups | `powerup-design.html` | OK |
| References | `references.html` | OK |

### References Page — Planning Docs (md/)

All 14 linked markdown files exist:

- `unified-prd-wave-survival.md`
- `game-design-document.md`
- `expert-critique.md`
- `vertical-slice-roadmap.md`
- `mvp-status-report.md`
- `agent-swarm-handoff.md`
- `agent-swarm-build-handoff-prompt.md`
- `game-design-expert-handoff.md`
- `external-expert-prompt.md`
- `menu-direction-brief.md`
- `zombies-wave-map-mockups-design.md`
- `cycle-2-handoff-prompt.md`
- `cycle-3-handoff-prompt.md`
- `cycle-4-handoff-prompt.md`

### References Page — Design Docs (in-site)

- 3D Asset Gallery, Audio Asset Gallery, Menu & HUD, Wave Map Mockups, Audio Design, Power-Up Design — all OK.

### External Links (references.html)

- KayKit (itch.io, GitHub), Kenney, Monogram, PS4 Buttons, Freesound — external; not validated.

### CSS

- `../css/style.css` from docs pages → OK  
- `css/style.css` from wave-survival index → OK  

### Play Page

- `../index.html` (Back to Wave Survival) → OK  

---

## Known Limitations

1. **3D Asset Gallery** — KayKit `.glb` models (characters, skeletons, weapons, animations) are not in the repo; model-viewer shows empty placeholders. Planets, Kenney textures, and power-up icons load correctly.

2. **Play Map A** — Placeholder until Godot HTML5 export is built and copied to `play/`. See `play/README.md` for export steps.

3. **Deploy** — GitHub Actions deploys `site/` on push to `main`.

---

## Next Steps for Implementers

1. **Game export** — Export Godot project to HTML5 (Web preset), copy build to `site/games/wave-survival/play/`, push.

2. **3D gallery** — To show models on the site, either host KayKit `.glb` files in the repo (large) or link to an external CDN.

3. **Agent swarm** — Use `agent-swarm-build-handoff-prompt.md` for planning/alignment before implementation. Paths in that prompt reference the godot-game repo; adapt to `site/games/wave-survival/docs/md/` when working from this repo.
