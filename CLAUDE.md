# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone built with vanilla HTML5 Canvas and ES6+ JavaScript — no frameworks, no bundler, no dependencies, no package.json. The entire game logic lives in a single file: `game.js`.

## Running

Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

There is no build step, no linter, and no test suite — there is nothing to compile or check beyond loading the page and playing the game. Verify changes by opening `index.html` (or the local server URL) in a browser and testing gameplay directly.

## Architecture

`game.js` is organized top-to-bottom as: input handling, small math utils, entity classes, game state, update loop, draw loop, and the `requestAnimationFrame` main loop at the bottom. There are no modules — everything is defined at file scope in load order, and `initGame()` / `requestAnimationFrame(loop)` at the end of the file kick things off.

Key pieces:

- **Coordinate space & wraparound**: the playfield is a fixed `W × H` (800×600) toroidal space. Any entity with position must wrap via the `wrap(v, max)` util in both `update()` methods — ship, bullets, and asteroids all wrap independently.
- **Entity classes** (`Bullet`, `Asteroid`, `Ship`, `Particle`): each has its own `update(dt)` and `draw()`, and a `dead` flag used for filtering out of the corresponding array (`bullets`, `asteroids`, `particles`) each frame rather than splicing in place.
- **Asteroid sizing**: size is an integer 3 (large) → 1 (small), indexing into parallel arrays `RADII`, `SPEEDS`, `POINTS`. `Asteroid.split()` produces two smaller asteroids at the same position; size 1 asteroids don't split.
- **Game state machine**: the module-level `state` variable is one of `'playing' | 'dead' | 'gameover'`, checked at the top of `update(dt)` to branch behavior (respawn timer on death, restart-on-Space on game over).
- **Input**: `keys` tracks held state, `justPressed`/`pressed()` implements edge-triggered presses (used for firing and restart) so a single keydown doesn't repeat every frame.
- **Collision detection**: plain circle-circle distance checks (`dist()` + radius sums) for bullet-vs-asteroid and ship-vs-asteroid; no spatial partitioning since entity counts are small.
- **Frame loop**: `dt` is delta time in seconds, clamped to 0.05s max (`loop()`) to avoid large jumps after tab-switch/lag; all movement and timers are `dt`-scaled.

When adding new entity types or behaviors, follow the existing pattern: a class with `update(dt)`/`draw()`/`dead`, pushed into a module-level array, filtered each frame in `update()`.
