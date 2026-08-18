# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-page Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no package.json — the repo is just `index.html`, `style.css`, and `game.js`.

## Running / testing

There is no build or test suite. To run the game, serve the directory and open it in a browser:

```bash
python3 -m http.server 8000   # or: npx serve .
```

Then open `http://localhost:8000`. Opening `index.html` directly via `file://` also works since there are no modules or fetch calls.

Since there's no test harness, verify changes manually in the browser: check piece movement/rotation near walls (wall kicks), line clearing at the bottom rows, pause/game-over overlays, and the next-piece preview.

## Architecture

Everything lives in `game.js` (~300 lines, single file, no modules, top-to-bottom procedural style with module-level mutable state: `board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.).

- **Board model**: `ROWS × COLS` (20×10) matrix; each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
- **Pieces**: `PIECES` array holds the 7 standard tetrominoes as square matrices; `COLORS` maps the same indices to hex colors. Rotation (`rotateCW`) is a transpose + row-reverse, not a lookup table of rotation states.
- **Collision** (`collide`): checks board bounds and existing locked cells for a given shape/offset.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns until one doesn't collide, else the rotation is discarded.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time (`dropAccum`) and advances the piece one row once `dropInterval` is exceeded, otherwise calls `lockPiece()`.
- **Locking a piece** (`lockPiece`): `merge()` writes the piece into `board` → `clearLines()` → `spawn()` a new piece. If the newly spawned piece immediately collides, `endGame()` fires.
- **Line clears** (`clearLines`): scans bottom-up, splices out full rows and unshifts empty ones at the top; awards score via `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`, and recomputes `level`/`dropInterval`.
- **Scoring/speed**: level increases every 10 lines; `dropInterval = max(100, 1000 - (level-1)*90)` ms. Hard drop awards 2 pts/row dropped, soft drop 1 pt/row.
- **Ghost piece**: `ghostY()` projects the current piece straight down to its landing row; drawn at `globalAlpha = 0.2`.
- **Rendering**: `draw()` redraws the full board canvas each frame (grid → locked blocks → ghost → current piece); `drawNext()` renders the preview onto the separate `#next-canvas`.
- **Input**: a single `keydown` listener switches on `e.code` (arrows, `KeyX` for rotate, `Space` for hard drop, `KeyP` for pause), gated by `paused`/`gameOver` flags.

`index.html` just wires up the two `<canvas>` elements (`#board` 300×600, `#next-canvas` 120×120), the HUD (`#score`, `#lines`, `#level`), and the pause/game-over `#overlay`. `style.css` provides the dark/retro visual theme (flexbox layout, `backdrop-filter` on overlays).

## Tunable constants (in `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval` (initial value, set in `init()`). If `COLS`/`ROWS`/`BLOCK` change, the `#board` canvas `width`/`height` in `index.html` must be updated to match (`COLS×BLOCK` by `ROWS×BLOCK`).
