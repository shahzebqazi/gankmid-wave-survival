# Play Map A — Web Build

This folder holds the Godot HTML5 export for Map A.

## Export from Godot

1. **Install Web export templates** (one-time): Godot Editor → Editor → Manage Export Templates → Download and Install
2. Open the game project (`godot-game/project/`)
3. **Project → Export** → Select the "Web" preset
4. Click **Export Project** (output: `project/builds/web/`)
5. Copy files to this folder. From `godot-game/` run: `./scripts/copy-web-build.sh`

## Deploy

Push to `main` — the GitHub Actions workflow deploys the `site/` folder to GitHub Pages.
