# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A clone of the arcade game **Asteroids**, written in pure HTML5 Canvas + ES6 JavaScript. No frameworks, no bundler, no dependencies, no build step.

## Running

Open `index.html` directly in a browser, or serve the directory:

```bash
npx serve .
```

There are no tests, no linter, and no package.json — the project is three files: `index.html`, `game.js`, `favicon.svg`.

## Architecture

All game logic lives in `game.js` (~420 lines). It follows a classic fixed-structure game loop:

- **Entity classes** (`Bullet`, `Asteroid`, `Ship`, `Particle`) each own `update(dt)` and `draw()`, plus a `dead` flag used for removal. Asteroids also have `split()`. Everything is time-based via `dt` (seconds), so behavior is framerate-independent.
- **Global game state** is held in module-level `let` variables (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`, `deadTimer`) rather than a container object. `state` is a string machine: `'playing' | 'dead' | 'gameover'`.
- **The loop** (`loop` → `update(dt)` → `draw()`) runs on `requestAnimationFrame`; `dt` is clamped to 0.05s max to survive tab-switches. `update` branches on `state` first, then handles input, movement, collisions, and level progression.
- **Dead-entity removal pattern**: entities set `this.dead = true`, then `update` filters them out with `array.filter(e => !e.dead)`. Newly spawned asteroids from splits are collected separately and concatenated after the collision pass to avoid mutating the array mid-iteration.
- **Input** is captured into a `keys` map (held state) and a `justPressed` map consumed via `pressed(code)` for edge-triggered actions like shooting and restarting.

### Key conventions

- The playfield is **toroidal** — `wrap(v, max)` wraps positions across both edges for ships, bullets, and asteroids (but not particles).
- Canvas is a fixed `800×600` (`W`, `H` constants in both `game.js` and the `<canvas>` element).
- Asteroid behavior is table-driven by `size` (1=small, 2=medium, 3=large): `RADII`, `SPEEDS`, `POINTS` are indexed by size. Large asteroids split into two of the next size down; size-1 asteroids are destroyed outright.
- All rendering is immediate-mode canvas drawing in white (`#fff`) on black, using `ctx.save()`/`ctx.translate()`/`ctx.rotate()`/`ctx.restore()` per entity. Ships and asteroids are stroked polygons defined by local-space vertex coordinates.

## Note on README drift

`README.md` (in Spanish) still describes **power-ups** and a **shooting-star (estrella fugaz)** asteroid type. These were removed (commit `13e713f`) and no longer exist in `game.js`. Treat `game.js` as the source of truth; update the README if you touch those areas.



