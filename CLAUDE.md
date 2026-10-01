# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Vanilla JS Tetris on HTML5 Canvas. No dependencies, no `package.json`, no build, no tests, no linter. UI text and README are in Spanish.

## Run

```bash
open index.html                 # or:
python3 -m http.server 8000     # then http://localhost:8000
```

## Architecture

All logic is in `game.js` (single script, global mutable state, `'use strict'`). `index.html` provides the DOM elements it grabs by id at load time (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`) — renaming an id breaks the script.

- **Cell values = piece type = color index.** Board cells and piece shape matrices store `0` or `1–7`; that number indexes both `PIECES` and `COLORS`. Adding a piece means adding to both arrays and updating the `* 7` in `randomPiece()`.
- **Loop**: `init()` → `spawn()` → `requestAnimationFrame(loop)`. `loop` accumulates `dt` into `dropAccum` and drops one row per `dropInterval`; landing goes through `lockPiece()` → `merge()` → `clearLines()` → `spawn()`. `spawn()` colliding on arrival triggers `endGame()`.
- **Rotation**: `rotateCW` (clockwise only) + `tryRotate` wall kicks `[0, -1, 1, -2, 2]` columns. Not SRS.
- **Scoring/level** live in `clearLines()`: `LINE_SCORES[n] * level`, level = `floor(lines/10)+1`, `dropInterval = max(100, 1000 - (level-1)*90)`. Soft drop +1/row, hard drop +2/row.
- Pause/game-over reuse one overlay (`#overlay`, toggled via the `hidden` class).
- Canvas size is hardcoded in `index.html` (300×600 = `COLS×BLOCK` × `ROWS×BLOCK`); change it together with `COLS`/`ROWS`/`BLOCK`. The next-piece preview assumes a 4×4 grid of 30px on a 120×120 canvas.

## Known quirks

- Unpausing doesn't re-add `hidden` to the overlay, so the "PAUSA" overlay stays visible after resuming.
- When game over is triggered from inside `loop` (gravity lock), `endGame()` cancels the frame but `loop` then schedules a new one, so the loop keeps running after game over.
