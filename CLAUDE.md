# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Gherkins War** is a mobile-first browser game (also packaged as an iOS app via Capacitor). It is a tile-based shooter where a player fights bosses on a grid map. The entire game logic and rendering live in a single IIFE in `game.js`.

## Running the Game

There is no build step. Open `index.html` directly in a browser or serve with `npx serve .`.
For the iOS app, `capacitor.config.json` points to a live GitHub Pages URL — no local native build is needed.

## Architecture

All game code is in `game.js` as one large IIFE with no modules or bundler. Key sections:

- **Constants** (top of file): sprite coordinates (`SPRITES`), timing constants (move speed, shot timing, flash durations), direction mappings.
- **`game` object**: single mutable state object holding board state, character positions, shot state, animation handles, cached canvases, and timers.
- **Level format** (`levels/*.csv`): 8×16 (portrait) or 16×8 (landscape) CSV grids. Tokens: `t` (floor), `w` (wall), `s` (water/swamp), `_` (void), `p` (player start), `b` / `c` / `m` (bosses). Portal wall tokens use directional syntax like `/w\`, `<w>`, etc. — parsed in `parseLevel()`.
- **Rendering pipeline**: `drawField()` composites a cached `terrainCanvas` (floor/water tiles, drawn once) with per-frame depth-sorted character and wall layers. Wall depth is `y * 4 + 3`; characters are `y * 4 + 2` for painter's-algorithm ordering.
- **Movement**: joystick (touch drag) and arrow keys move the player on a timer (`JOYSTICK_STEP_MS`). Diagonal moves are prohibited. Aim direction is locked separately via WASD or aim pad buttons.
- **Sprites**: single sprite sheet at `assets/combined.png`. All sprite coordinates are defined in the `SPRITES` constant.
- **Level editor**: `redactor.html` is a standalone local level editor (no server needed); edit CSV files directly and reload the game.

## Controls (keyboard)

- **Arrow keys**: move player
- **WASD**: aim direction
- **Space / fire button**: shoot

## Token & Execution Rules

- **No Python / shell scripts for images or edits**: Do NOT write or run Python scripts to inspect `assets/combined.png` or to modify `game.js`. Rely on the existing `SPRITES` coordinate layout in `game.js` and edit files directly using native file editing tools.
- **Targeted reads**: `game.js` is a single large file—read only the relevant sections (e.g., `SPRITES`, `parseLevel`, rendering/AI loops) instead of dumping the entire file repeatedly.
- **No verification servers**: Never start `npx serve` or headless browsers to test changes unless explicitly requested.
- **Minimal output**: Keep responses strictly concise (1–3 sentences max). Do not explain code line-by-line or summarize obvious edits.