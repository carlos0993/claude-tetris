# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A Tetris clone in vanilla JavaScript with HTML5 Canvas. It has no dependencies, no build step, no package.json, no tests and no linter. The README and the in-game UI text are in Spanish (`lang="es"`). Keep user-facing strings in Spanish.

## Running

Serve the folder statically and open http://localhost:8000:

    python3 -m http.server 8000

`open index.html` also works because there are no modules or fetches. To check a change, reload the browser. There is no automated test suite.

## Architecture

Everything lives in `game.js`. It is a single classic script (`'use strict'`, no modules) that `index.html` loads at the end of `<body>`. It grabs its DOM elements by id at load time, so renaming an id in `index.html` breaks it.

- **State** is a set of module-level `let` globals: `board, current, next, score, lines, level, paused, gameOver, lastTime, dropAccum, dropInterval, animId`. `init()` resets all of them, and it is also the restart-button handler.
- **Board** is a `ROWS × COLS` matrix of `0` (empty) or a piece type index `1–7`. That same index selects the color from `COLORS[]` and the shape from `PIECES[]`, so the three arrays must stay aligned. Index 0 is `null` in both.
- **Pieces** are square matrices whose cells hold their type index. Rotation is `rotateCW` (transpose + reverse), and `tryRotate` applies simple horizontal kicks `[0,-1,1,-2,2]`. These kicks are not SRS.
- **`collide(shape, x, y)`** is the single legality check. Cells with `y < 0` are allowed (above the board).
- **Lock pipeline:** `lockPiece()` → `merge()` → `clearLines()` (updates lines, score, level and `dropInterval`) → `spawn()`. If a piece collides as it spawns, `spawn()` calls `endGame()`.
- **Loop:** `loop(ts)` runs on `requestAnimationFrame`. It adds frame time to `dropAccum` and applies gravity once that passes `dropInterval`. It redraws the whole board every frame. Pause and game over stop the loop with `cancelAnimationFrame(animId)`. Unpausing restarts it by calling `loop()` directly.
- **Rendering:** `draw()` draws the grid, the locked cells, the ghost piece (`ghostY()`, alpha 0.2) and the current piece. `drawNext()` renders to the separate 120×120 `#next-canvas` and runs only on spawn. The HUD is DOM text that `updateHUD()` sets.
- **Scoring:** `LINE_SCORES[cleared] * level`. A soft drop adds +1 per row and a hard drop adds +2 per cell. The level is `floor(lines/10)+1`. The fall speed is `max(100, 1000 − (level−1)·90)` ms.

## Gotchas

- If you change `COLS`, `ROWS` or `BLOCK`, also update the `<canvas id="board">` `width`/`height` in `index.html` (`COLS×BLOCK` by `ROWS×BLOCK`). `drawNext` hardcodes a 4×4 preview at 30px.
- `togglePause()` shows the overlay when pausing but never adds `hidden` back on resume. The overlay stays visible after unpausing.
