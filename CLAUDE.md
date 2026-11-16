# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tetris in vanilla JavaScript + HTML5 Canvas + CSS. No dependencies, no build step, no tests, no linter. The README and UI text are in Spanish.

## Running

Open `index.html` directly, or serve statically (e.g. `python3 -m http.server 8000`) and visit `http://localhost:8000`.

## Architecture

Three files, all loaded directly by `index.html` (`game.js` is a plain `<script>`, not a module):

- `index.html` — fixed DOM the script depends on by ID: `board` (300×600 canvas), `next-canvas` (120×120), `score`/`lines`/`level`, and the `overlay` (`overlay-title`, `overlay-score`, `restart-btn`) used for both PAUSA and GAME OVER. Renaming an ID breaks `game.js`, which looks them all up at load time.
- `style.css` — presentation only.
- `game.js` — the entire game, using module-level mutable globals (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, …) declared together and (re)set in `init()`.

### Game logic notes (`game.js`)

- Board is a `ROWS × COLS` (20×10) array of ints; `0` is empty, `1–7` index into both `COLORS` and `PIECES`. A piece's cell values are its own type index, so merging into the board preserves color.
- The loop is `requestAnimationFrame`-driven (`loop`): it accumulates time into `dropAccum` and applies gravity when it exceeds `dropInterval`. Pause/game-over cancel the frame (`cancelAnimationFrame(animId)`); resuming resets `lastTime` and restarts `loop`.
- Piece lifecycle: `lockPiece()` → `merge()` → `clearLines()` → `spawn()`. `spawn()` promotes `next` to `current` and triggers `endGame()` if the new piece immediately collides.
- `clearLines()` owns scoring and leveling: `LINE_SCORES[cleared] * level`, level = `floor(lines/10)+1`, `dropInterval = max(100, 1000 - (level-1)*90)`. Soft drop adds 1 per cell, hard drop 2 per cell (in `softDrop`/`hardDrop`).
- Rotation is clockwise-only via matrix rotation (`rotateCW`) with horizontal wall kicks `[0,-1,1,-2,2]` — not SRS. The O piece is rotated like any other (harmless for a 2×2).
- `collide(shape, ox, oy)` is the single collision check (walls, floor, board); rows with `ny < 0` are allowed above the board.
- Rendering redraws everything each frame in `draw()` (grid, board, ghost at 0.2 alpha via `ghostY()`, current piece). `drawBlock` is shared between the main and "next" canvases.
- HUD is updated imperatively via `updateHUD()`; call it after any state change affecting score/lines/level.
