# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Gherkins War** is a mobile-first browser game (also packaged as an iOS app via Capacitor). It is a tile-based shooter where a player fights bosses on a grid map. The entire game logic and rendering live in a single IIFE in `game.js`.

## Running the Game

There is no build step. Open `index.html` directly in a browser or serve with `npx serve .`.
For the iOS app, `capacitor.config.json` points to a live GitHub Pages URL — no local native build is needed.
Deploy: pushing to `main` publishes to GitHub Pages via `.github/workflows/deploy.yml`, and the iOS app loads that live site. Treat `main` as production.

## Architecture

All game code is in `game.js` as one large IIFE with no modules or bundler. Key sections:

- **Constants** (top of file): sprite coordinates (`SPRITES`), timing constants (move speed, shot timing, flash durations), direction mappings.
- **`game` object**: single mutable state object holding board state, character positions, shot state, animation handles, cached canvases, and timers.
- **Level format** (`levels/*.csv`): 8×16 (portrait) or 16×8 (landscape) CSV grids. Tokens: `t` (floor), `w` (wall), `s` (water/swamp), `_` (void), `p` (player start), `b` / `c` / `m` (bosses). Portal wall tokens use directional syntax like `/w\`, `<w>`, etc. — parsed in `parseLevel()`.
- **Rendering pipeline**: `drawField()` composites a cached `terrainCanvas` (floor/water tiles, drawn once) with per-frame depth-sorted character and wall layers. Wall depth is `y * 4 + 3`; characters are `y * 4 + 2` for painter's-algorithm ordering.
- **Movement**: joystick (touch drag) and arrow keys move the player on a timer (`JOYSTICK_STEP_MS`). Diagonal moves are prohibited. Aim direction is locked separately via WASD or aim pad buttons.
- **Sprites**: single sprite sheet at `assets/combined.png`. All sprite coordinates are defined in the `SPRITES` constant. The sheet is prepared in `start()`: red (255,0,0) is chroma-keyed to transparent on a canvas, so keep that step until the PNG has real alpha.
- **Level editor**: `redactor.html` is a standalone local level editor (no server needed); edit CSV files directly and reload the game.

## Controls (keyboard)

- **Arrow keys**: move player
- **WASD**: aim direction
- **Space / fire button**: shoot

## Code Rules

- **Scope**: one concern per branch and PR. No drive-by reformatting, renames, or unrelated cleanups. No new dependencies and no build step.
- **Cache busting**: when changing `game.js` or `style.css`, bump the matching `?v=` query string in `index.html`.
- **Canvas rendering**: keep `imageSmoothingEnabled = false` and snap draw coordinates to whole pixels, to avoid blur and bleeding between atlas frames.
- **Magic numbers**: new numeric values become named constants at the top of the file (entity y-offset `0.3`, boss height `1.3`, z-layer `y * 4 + n` are existing examples).
- **Timers and state**: any new timer must be pausable. Pause on `visibilitychange` and while the motion prompt is open. Reset teleport flags (`isPlayerTeleporting`, `isBossTeleporting`) whenever the level re-renders.
- **Rotation**: `rotateLevel` must keep walls, entities, water, portals, and locked aim direction consistent. Re-check it after any change to level state.
- **Touch and iOS**: keep `touch-action: none` on controls, avoid layout-triggering properties in per-frame code, and never rely on zoom or scroll gestures.
- **Accessibility**: the field is canvas or decorative. Meaningful status goes in an `aria-live` region, not in invalid `role="grid"` markup.
- **Dead code**: list candidates and confirm with me before deleting anything.
- **Git**: work on a branch named `feat/...`, `fix/...`, or `chore/...`. Never push to `main`. Open a PR with a one-line summary.

## Token & Execution Rules

- **No Python / shell scripts for images or edits**: Do NOT write or run Python scripts to inspect `assets/combined.png` or to modify `game.js`. Rely on the existing `SPRITES` coordinate layout in `game.js` and edit files directly using native file editing tools.
- **Targeted reads**: `game.js` is a single large file—read only the relevant sections (e.g., `SPRITES`, `parseLevel`, rendering/AI loops) instead of dumping the entire file repeatedly.
- **No verification servers**: Never start `npx serve` or headless browsers to test changes unless explicitly requested. I verify visually on a device.
- **Verify before changing**: if a task assumes code that no longer exists (for example DOM sprite styling after the canvas move), report that and stop instead of improvising.
- **Plan first for big changes**: for refactors, deploy or workflow changes, and gameplay features, show a short plan and wait for approval before editing.
- **Minimal output**: Keep responses strictly concise (1–3 sentences max). Do not explain code line-by-line or summarize obvious edits.