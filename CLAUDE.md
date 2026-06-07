# Doof In Space — Project Memory

This file is automatically maintained to preserve context across sessions.
Always read this file at the start of any session working on this project.

---

## Project Overview
- **Repo**: danhgottlieb/doof-in-space
- **Local path**: C:\Users\dagottl\Projects\doof-in-space
- **Live URL**: https://doof-in-space.vercel.app
- **Vercel project**: doof-in-space (under danhgottliebs-projects)
- **Concept**: Standalone deployment of the "Doof In Space" asteroids game from the doofs-adventures universe
- **Tech**: Single HTML file (`index.html`), vanilla JS, Canvas 2D, all assets in root directory
- **Deploy**: `npx vercel --prod` from repo root, also auto-deploys on push via GitHub

## Architecture
- Single `index.html` contains all game code in one `<script>` tag (~4100 lines)
- Image assets (PNGs) and audio (WAV/MP3) in root alongside index.html
- No build step, no framework, no dependencies
- Vercel serves the root directory as a static site

## Game Features
- Pug-themed Asteroids clone with full state machine: title → difficulty select → playing → boss
- Easy/Hard difficulty modes
- Special weapons: laser gun, pulse wave, homing missiles
- Enemy UFOs (green from level 2, red aces from level 3)
- Multi-phase boss fights every 3 levels
- Shield pickups, Web Audio SFX, background music
- Konami code level select
- localStorage high score persistence
- Mobile-first touch controls (keyboard: arrows/WASD + Space/F)

## Key Technical Notes
- **Variable ordering matters**: `let` declarations used throughout; the `resize()` call during init triggers `generateTitleAsteroids()` which needs `difficulty` to already be initialized. Keep game state variables ABOVE the resize call.
- **Keydown handler flow**: Uses `if/else if` chains for state transitions to prevent one keypress from triggering multiple state changes in the same event.
- **Ship sprites**: PNGs were pre-processed with Pillow (cockpit dome fix). No runtime alpha cleanup — images are used as-is.
- **frameCount**: Incremented each frame inside `draw()`, used for all animation timing throughout the game.

## Session History

### Session — 2026-06-06
- **Fixed site-down crash**: `let difficulty = 'hard'` was declared AFTER the `resize()` call that triggered `generateTitleAsteroids()` → `getAsteroidSpeed()` → accesses `difficulty`. Temporal dead zone crash killed the entire game on page load. Moved declaration above `resize()`.
- **Fixed difficulty select skip**: Keydown handler used sequential `if` blocks so pressing Space on title both opened difficulty select AND immediately confirmed it. Changed to `else if`.
- **Redesigned difficulty select UI**: Replaced plain text overlay with sci-fi cockpit panel — dark vignette, glowing card borders, sparkle particles, pulsing selected option with scale animation.
- **Fixed ship sprite halo**: Cockpit dome area had full-alpha bright blue/white pixels from a bad background removal. Processed PNGs with Python/Pillow to convert dome to dark semi-transparent glass. Also added runtime `cleanSpriteAlpha()` for edge fringe cleanup.
- All changes deployed to https://doof-in-space.vercel.app and pushed to GitHub.

### Session — 2026-06-07
- **Fixed ship too transparent**: Removed the `cleanSpriteAlpha()` runtime function entirely. It was double-processing PNGs that had already been fixed with Pillow, making the ship look like a ghost. Ship now renders at full opacity using the raw PNG assets.
- Deployed to https://doof-in-space.vercel.app and pushed to GitHub.

## Key Technical Notes Update
- **Ship sprite cleanup** is NO LONGER performed at runtime. The PNGs (`pug-ship.png`, `pug-ship-shooting.png`) were pre-processed with Pillow and are used as-is.

## Open Items
- [ ] The `doofs-adventures` repo has an older version of this game without difficulty modes — consider syncing
- [ ] Mobile touch controls not yet verified with difficulty select screen
