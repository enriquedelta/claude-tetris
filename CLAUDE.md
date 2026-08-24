# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Tetris implemented in vanilla JavaScript (HTML5 Canvas + CSS). No dependencies, no build step, no `package.json`.

## Running / testing

There is no build or test suite. To run the game, serve the directory and open `index.html`:

```bash
python3 -m http.server 8000     # or: npx serve .
```

Then open `http://localhost:8000`. Opening `index.html` directly via `file://` also works.

Since there is no compilation step, verify changes to `game.js` by reloading the page in a browser and playing.

## Architecture

Three files, no modules/bundler — everything is global scope in `game.js`, loaded via a single `<script>` tag in `index.html`.

- **`index.html`** — DOM structure: main `<canvas id="board">` (300×600, i.e. `COLS×BLOCK` by `ROWS×BLOCK`), a side panel canvas `#next-canvas` for the next-piece preview, HUD spans (`#score`, `#lines`, `#level`), and a shared `#overlay` used for both PAUSE and GAME OVER states.
- **`style.css`** — dark/retro arcade look; no JS-driven class logic beyond toggling `.hidden` on `#overlay`.
- **`game.js`** — all game logic, structured around a `requestAnimationFrame` loop operating on module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc. — declared once, reset in `init()`).

Key mechanics, if you need to change gameplay behavior:

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a piece-color index `1–7`.
- **Pieces**: `PIECES` are square matrices; rotation is done by `rotateCW` (transpose + reverse), not by precomputed rotation states.
- **Collision**: `collide(shape, ox, oy)` checks board bounds and cell overlap; used both for movement and for the ghost-piece projection.
- **Wall kicks**: `tryRotate()` rotates then retries at x-offsets `[0, -1, 1, -2, 2]` until one doesn't collide.
- **Locking a piece**: `lockPiece()` → `merge()` (bake shape into `board`) → `clearLines()` → `spawn()` (promote `next` to `current`, generate new `next`; if the new piece immediately collides, `endGame()` fires).
- **Line clearing / scoring**: `clearLines()` scans bottom-up, splices full rows, unshifts empty rows at top; score uses `LINE_SCORES = [0,100,300,500,800]` × `level`. Level increments every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
- **Ghost piece**: `ghostY()` projects `current` straight down via repeated `collide` checks; drawn at `globalAlpha = 0.2`.
- **Rendering**: `draw()` clears and redraws the full board every frame (grid → locked blocks → ghost → current piece); `drawNext()` renders the preview canvas independently.

Tunable constants live at the top of `game.js`: `COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, the `#board` canvas `width`/`height` in `index.html` must be updated to match (`COLS×BLOCK`, `ROWS×BLOCK`).

Controls (handled in a single `keydown` listener): arrows to move/rotate/soft-drop, `Space` for hard drop, `P` to pause/resume.
