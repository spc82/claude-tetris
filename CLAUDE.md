# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris. No dependencies, no build step, no package.json, no tests. Three files: `index.html`, `style.css`, `game.js` (~300 lines, all game logic).

## Running

Open directly or serve statically:

```bash
open index.html          # macOS, just opens the file
python3 -m http.server 8000   # or: npx serve .
```

No lint/build/test commands exist in this repo.

## Architecture (`game.js`)

- **Board model**: `ROWS × COLS` matrix, each cell `0` (empty) or color index `1-7`.
- **Pieces**: square matrices; rotation is transpose + row-reverse (`rotateCW`).
- **Collision**: `collide()` checks board bounds and overlap with locked cells.
- **Wall kicks**: `tryRotate()` shifts the piece ±1/±2 columns if rotation collides.
- **Game loop**: `requestAnimationFrame`-driven `loop()`, accumulates delta time, drops piece one row when `dropInterval` elapsed.
- **Line clear**: `clearLines()` scans bottom-up, removes full rows, unshifts empty rows at top.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` × level; hard drop = 2 pts/cell, soft drop = 1 pt/row.
- **Level/speed**: level up every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
- **Ghost piece**: `ghostY` projects current piece straight down, drawn at `globalAlpha = 0.2`.

Flow: `init()` → `createBoard()` + first `spawn()` → `requestAnimationFrame(loop)`. `spawn()` moves `next` into `current` and generates a new `next`; if the new piece immediately collides, `endGame()` fires.

Tunable constants live at the top of `game.js`: `COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`. Changing `COLS`/`ROWS`/`BLOCK` requires updating the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS×BLOCK` by `ROWS×BLOCK`).
