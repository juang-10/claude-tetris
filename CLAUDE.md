# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla Tetris — HTML5 Canvas + CSS + JS, no dependencies, no build step, no package.json. Just three files: `index.html`, `style.css`, `game.js`.

## Running / testing

No build/test tooling exists. To try the game, serve the directory statically and open it in a browser:

```bash
python3 -m http.server 8000   # or: npx serve .
```

Then open `http://localhost:8000`. There is no automated test suite — verify changes by playing the game (see the `run` skill for driving the app in a browser).

## Architecture

Everything lives in `game.js` (~300 lines), a single global-scope script (no modules, no classes):

- **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or an integer 1–7 indexing into `COLORS`/`PIECES`.
- **Pieces**: the 7 tetrominoes are hardcoded as square matrices in `PIECES`. Rotation (`rotateCW`) is done by transposing + reversing rows — there's no separate rotation-state table (no SRS).
- **Collision** (`collide`): checks a shape against board bounds and fixed cells at a given offset.
- **Wall kicks** (`tryRotate`): on rotation collision, retries at column offsets ±1/±2 before giving up.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates elapsed time and drops the piece one row once `dropInterval` is exceeded. `dropInterval` shrinks as `level` increases (`max(100, 1000 - (level-1)*90)` ms).
- **Line clears** (`clearLines`): scans bottom-up, removes full rows, unshifts empty rows at the top; scoring uses `LINE_SCORES = [0,100,300,500,800]` × level.
- **Ghost piece**: `current`'s landing position is projected downward and drawn at `globalAlpha = 0.2`.
- **State**: a handful of module-level `let` variables (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, ...) hold all game state — no framework, no external state container.

`index.html` provides two canvases (`#board` 300×600, `#next-canvas` 120×120 for the preview) and the score/lines/level panel; `style.css` is a dark/retro-arcade theme.

When changing `COLS`, `ROWS`, or `BLOCK` in `game.js`, also update the `#board` canvas `width`/`height` in `index.html` to match (`COLS × BLOCK`, `ROWS × BLOCK`).

The README (in Spanish) has a more detailed walkthrough of the game flow and tunable constants if deeper context is needed.
