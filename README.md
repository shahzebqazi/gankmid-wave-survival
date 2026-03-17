# gankmid-wave-survival

Design documentation site for the **Dungeon Prototype — Wave Survival** game built with Godot 4.6.

## Site Structure

```
site/
├── index.html                    # Landing page
├── css/style.css                 # Shared styles
└── docs/
    ├── ui-assets-gallery.html    # 3D Asset Gallery (iframe wrapper)
    ├── audio-asset-gallery.html  # Audio Asset Gallery (iframe wrapper)
    ├── menu-hud-mockups.html     # Menu & HUD Mockups (iframe wrapper)
    ├── wave-map-mockups.html     # Wave Map Mockups (iframe wrapper)
    ├── audio-design.html         # Audio Design Spec (markdown rendered)
    ├── powerup-design.html       # Power-Up Design Spec (markdown rendered)
    ├── references.html           # All planning docs & external links
    ├── embed/                    # Original self-contained HTML design docs
    └── md/                       # Source markdown files
```

## Hosting

Designed to be served from `dev.gankmid.co/` on Digital Ocean with proper COOP/COEP headers for future Godot web exports.

## Serving Locally

```bash
cd site
python3 -m http.server 8000
```

Then open http://localhost:8000
