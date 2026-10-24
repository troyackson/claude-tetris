# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas). No `package.json`, no build, no tests, no linter. README and UI text are in Spanish.

## Run

Open `index.html` directly, or serve statically: `python -m http.server 8000` → `http://localhost:8000`.

## Architecture

Three files, all logic in `game.js` (script-global state, no modules):

- `index.html` — fixed-size canvases: `#board` (300×600) and `#next-canvas` (120×120), HUD spans (`#score`, `#lines`, `#level`), `#overlay` for PAUSA / GAME OVER.
- `game.js` — module-level `let` state (`board`, `current`, `next`, `score`, ...) reset by `init()`. Loop is `requestAnimationFrame` → `loop(ts)` accumulates `dropAccum` against `dropInterval`, then `draw()`. Input handled in one `keydown` listener.
- Piece/board cells store color index 1–7 (`0` = empty); `PIECES` and `COLORS` are indexed by that same number, so adding a piece means updating both and the `Math.random()*7` in `randomPiece`.

Gotchas:

- `COLS`/`ROWS`/`BLOCK` must match the `<canvas id="board">` `width`/`height` in `index.html` (`COLS*BLOCK` × `ROWS*BLOCK`). Next-piece preview uses its own hardcoded 30px block and 4×4 grid.
- Game over/pause stop the loop via `cancelAnimationFrame(animId)`; resume/restart must reset `lastTime` to avoid a huge `dt` drop.
- `spawn()` calls `endGame()` on collision at spawn; `lockPiece` = `merge` → `clearLines` → `spawn`.
- Score: line table `LINE_SCORES × level`; soft drop +1/row, hard drop +2/row. Speed: `max(100, 1000 - (level-1)*90)` ms, level up every 10 lines.
- Rotation (`tryRotate`) is simple CW with horizontal kicks `[0,-1,1,-2,2]`, not SRS.
