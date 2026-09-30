# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Clon del clásico arcade Asteroids implementado en HTML5 Canvas puro. Sin dependencias, sin bundler, sin build step — es un solo archivo de JavaScript (`game.js`) cargado directamente por `index.html`.

## Running

There is no build/lint/test tooling. Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

Then visit `http://localhost:3000`. To verify a change, open the page and play — there are no automated tests.

## Architecture

Everything lives in `game.js` (single file, no modules/imports). The structure, top to bottom:

- **Input**: a raw `keys`/`justPressed` map populated by `keydown`/`keyup` listeners; `pressed(code)` consumes a one-shot press (used for shooting and restart) while `keys[code]` gives continuous state (used for rotation/thrust).
- **Entity classes**: `Bullet`, `Asteroid`, `Ship`, `Particle` — each owns its `update(dt)` and `draw()`. There is no shared base class or entity-component system; the game loop calls these methods directly on arrays of instances.
- **Global mutable state**: `ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state` are module-level `let` bindings, not encapsulated in an object. `state` is one of `'playing' | 'dead' | 'gameover'` and gates most of `update()`.
- **Game loop**: `requestAnimationFrame(loop)` drives `update(dt)` then `draw()` every frame, with `dt` clamped to 0.05s to avoid large jumps on tab-switch.
- **Space is toroidal**: all moving entities wrap via `wrap(v, max)` on both axes (`W=800`, `H=600`), rather than colliding with screen edges.
- **Collision**: brute-force O(n·m) distance checks each frame (bullets×asteroids, ship×asteroids) using `dist()`; no spatial partitioning, which is fine at this entity count.
- **Asteroid splitting**: size follows `3 → 2 → 1` (large/medium/small), each destroyed asteroid spawns two of the next size down via `Asteroid.split()`, with radius/speed/points looked up from the parallel `RADII`/`SPEEDS`/`POINTS` arrays indexed by size.
- **Level progression**: when `asteroids.length === 0`, `nextLevel()` increments `level` and spawns `3 + level` new large asteroids.
- **Ship death/respawn**: `killShip()` decrements `lives` and either ends the game (`state = 'gameover'`) or starts a timed respawn (`state = 'dead'`, `deadTimer`); the ship gets temporary `invincible` time (with blinking in `draw()`) after each `reset()`.
