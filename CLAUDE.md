# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Gherkins War** is a mobile-first browser game (also packaged as an iOS app via Capacitor). It is a tile-based shooter where a player fights a boss on a grid map. The entire game logic and rendering live in a single IIFE in `game.js`.

## Running the Game

There is no build step. Open `index.html` directly in a browser or serve it with any static file server:

```
npx serve .
```

For the iOS app, the Capacitor config in `capacitor.config.json` points to a live GitHub Pages URL — no local native build is needed for web development.

## Architecture

All game code is in `game.js` as one large IIFE with no modules or bundler. Key sections:

- **Constants** (top of file): sprite coordinates, timing constants (move speed, shot timing, flash durations), direction mappings.
- **`game` object**: single mutable state object holding board state, character positions, shot state, animation handles, cached canvases, and timers.
- **Level format** (`levels/*.csv`): 8×16 (portrait) or 16×8 (landscape) CSV grids. Tokens: `t` (floor), `w` (wall), `s` (water/swamp), `_` (void), `p` (player start), `b` (boss start). Portal wall tokens use directional syntax like `/w\`, `<w>`, etc. — parsed in `parseLevel()`.
- **Rendering pipeline**: `drawField()` composites a cached `terrainCanvas` (floor/water tiles, drawn once) with per-frame depth-sorted character and wall layers. Wall depth is `y * 4 + 3`; characters are `y * 4 + 2` for painter's-algorithm ordering.
- **Movement**: joystick (touch drag) and arrow keys move the player on a timer (`JOYSTICK_STEP_MS`). Diagonal moves are prohibited. Aim direction is locked separately via WASD or aim pad buttons.
- **Sprites**: single sprite sheet at `assets/combined.png`. All sprite coordinates are defined in the `SPRITES` constant.
- **Level editor**: `redactor.html` is a standalone local level editor (no server needed); edit CSV files directly and reload the game.

## Controls (keyboard)

- **Arrow keys**: move player
- **WASD**: aim direction
- **Space / fire button**: shoot