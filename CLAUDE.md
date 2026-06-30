# Doof In Space — Project Memory

This file is automatically maintained to preserve context across sessions.
Always read this file at the start of any session working on this project.

---

## Project Overview
- **Repo**: danhgottlieb/doof-in-space
- **Local path**: C:\Users\dagottl\Projects\doof-in-space
- **Live URL**: https://doof-in-space.vercel.app
- **Vercel project**: doof-in-space (under danhgottliebs-projects)
- **Concept**: Standalone deployment of the "Adventures in Space" (formerly "Doof In Space") asteroids game from the doofs-adventures universe. Two playable characters: Doof (pug) and Devyn, chosen on a front-page character select.
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

### Session — 2026-06-23
- **Added real mobile touch controls.** Previously the "mobile touch controls" claim was false — the game was keyboard-only (no `touchstart`/`touchend` handlers, no on-screen buttons). Added a full on-screen control layer:
  - HTML overlay `#touchUI` with cyan sci-fi glowing buttons: rotate ◄/► (bottom-left), thrust ▲ + FIRE (bottom-right), pause ⏸ (top-right), plus menu pills (TAP TO START, EASY, HARD).
  - **Hold buttons** drive `keys[]` directly via pointer events (`ArrowLeft/ArrowRight/ArrowUp/Space`) so they reuse the existing input model. One FIRE button covers both normal and special weapons (the game already routes special-weapon fire through `Space`).
  - **Tap buttons** for menus dispatch synthetic `KeyboardEvent`s (START→Space, pause→KeyP) or call `confirmDifficultySelection()` directly (EASY sets `difficultySelectChoice=0`, HARD sets `1`).
  - `updateTouchUI()` runs each frame in `loop()` and shows/hides the right button group by `gameState` (title / difficultySelect / playing+boss states); releases held keys on every group change so no input sticks.
  - Touch layer only activates on touch-capable devices (`ontouchstart` / `maxTouchPoints` / `pointer: coarse`); hidden entirely on desktop.
  - Verified end-to-end with agent-browser (headless Chrome reports coarse pointer): title→difficulty→play transitions, hold-to-rotate, hold-to-fire (5 bullets), and pause toggle all work.

### Session — 2026-06-23 (part 2) — Second character "Devyn" + character select
- **Renamed game to "Adventures in Space"** (`<title>` + character-select heading). The per-character splash still shows "DOOF IN SPACE" / "DEVYN IN SPACE".
- **Added a 2nd playable character, Devyn**, driven by a `CHARACTERS` config object (`doof`, `devyn`) with `CHARACTER_ORDER`. Each entry holds `ship`/`shipShooting`/`ejected`/`portrait` images plus `label`, `titleWord`, `pauseLabel` (Doof="PAWS", Devyn="PAUSE"), `pauseHint`, `hintPause`. Helpers: `activeChar()`, `activePortrait()`, `applyCharacterSelection()` (clears ALL title caches — titlePortraitCache, portraitGlowCache, titleTextSurface, titleFrameCache — since portrait + wave text differ per character).
- **New front-page state `'characterSelect'`** (initial `gameState`). Flow: characterSelect → title → difficultySelect → playing. ESC walks back (title→characterSelect). `drawCharacterSelectScreen()` renders heading, two glowing portrait cards (selected pulses), and a prompt.
- **All player visuals are now character-driven**: ship, firing sprite, ejected sprite, lives icon, title portrait, pause portrait/label/hint.
- **Devyn assets** (repo root, transparent PNGs, normalized to match pug framing so the shared render constants `SHIP_IMG_WIDTH/OFFSET_X/GUN_TIP_*` work and firing doesn't jump): `devyn-ship.png`, `devyn-ship-shooting.png`, `devyn-ejected.png`, `devyn-portrait.png`.
- **Touch:** added `character` group with `btnCharDoof`/`btnCharDevyn` pills; prompt text is touch-aware ("TAP A HERO TO LAUNCH").
- **Bug fixed during review:** the two character pills were both stuck centered (stacked) because `#touchUI .tpill { left:50% }` out-specifies `#btnCharDoof { left:28% }` — DOOF was un-tappable on mobile. Fixed by raising selector specificity to `#touchUI #btnCharDoof/#btnCharDevyn`.
- **Verified end-to-end with agent-browser** (desktop + 390×844 mobile): both characters' select → title → play → fire (no size jump) → pause (correct label/portrait) → ejected; both touch pills tappable and correctly positioned.


- **Fixed ship too transparent**: Removed the `cleanSpriteAlpha()` runtime function entirely. It was double-processing PNGs that had already been fixed with Pillow, making the ship look like a ghost. Ship now renders at full opacity using the raw PNG assets.
- Deployed to https://doof-in-space.vercel.app and pushed to GitHub.

## Key Technical Notes Update
- **Ship sprite cleanup** is NO LONGER performed at runtime. The PNGs (`pug-ship.png`, `pug-ship-shooting.png`) were pre-processed with Pillow and are used as-is.

## Open Items
- [ ] The `doofs-adventures` repo has an older version of this game without difficulty modes — consider syncing
- [x] Mobile touch controls implemented and verified (Session 2026-06-23). Note: Konami/level-select screen is not touch-accessible (keyboard only) — acceptable for now.

### Session — part 3 — Mobile hardening + arcade UI redesign
- **Boss double-text fixed**: the on-screen level-transition banner is suppressed on boss levels via `if (levelTransitionActive && !shouldTriggerBossFight(level))` and only ever renders `LEVEL N+1`. The boss-intro overlay is now the SOLE source of "A CHALLENGER APPROACHES".
- **Removed duplicate HTML menu buttons** (btnStart/btnEasy/btnHard/btnCharDoof/btnCharDevyn) and their CSS. Menus are now **canvas hit-tested**: `drawCharacterSelectScreen`/`drawDifficultySelectOverlay` populate `characterCardRects`/`difficultyCardRects` each frame; `setupMenuPointer()` (single active pointerId, 26px drag threshold) + `handleMenuTap(x,y)` drive selection for BOTH mouse click and touch tap. This eliminated the desktop/mobile button overlap.
- **Zoom prevention**: viewport `maximum-scale=1,user-scalable=no,viewport-fit=cover`; CSS `touch-action:none` + `overscroll-behavior:none`; JS `preventDefault` on `gesturestart/gesturechange/dblclick`, double-tap (≤320ms `touchend`) and pinch (`touchmove` >1 touch).
- **Landscape gate**: `#rotateNotice` overlay is **JS-driven** (toggle `.show` via `isPortraitBlocked()`), no CSS media-query fallback (avoids JS/CSS mismatch). `isPortraitBlocked()` = `touchEnabled && phoneLike (min viewport edge <700px) && portrait` — so desktop touch monitors / Surface / tablets are NOT force-blocked.
- **Auto-pause/resume on rotation**: `handleOrientationGate()` sets `paused=true` + `pausedByPortrait=true` when rotated to portrait mid-game, and **auto-resumes** on return to landscape only if `pausedByPortrait` (a manual pause is preserved across rotation).
- **Touch reliability**: gameplay `hold()` buttons use `setPointerCapture` per button and release on `lostpointercapture` (not `pointerleave`) → tolerates finger drift and supports multitouch (rotate + fire).
- **Arcade aesthetic** for character-select and difficulty-select: marquee gradient header, neon corner brackets on the selected card, inner + full-screen scanlines, "1P" tag, nameplate bars, "CREDIT 00"/"TAP TO SELECT" flavor corners, blinking start prompt. Doof=amber, Devyn=cyan; EASY=green/ROOKIE, HARD=pink/VETERAN. Helpers: `drawArcadeScanlines()`, `drawNeonBracket()`.
- **Squad-reviewed** (code-review + rubber-duck): adopted phone-like gate heuristic, pausedByPortrait auto-resume, single-pointer menu tracking, and JS-only overlay. Verified end-to-end with agent-browser at 1280×720, 844×390 (landscape) and 390×844 (portrait gate).

### Session — part 4 — Character-select polish + boss-level crash fix
- **Boss-level crash fixed (frequent)**: in `update()` (~line 2863) the boss-body collision check dereferenced `ship.x` immediately after the asteroid-collision loop could have called `damagePlayerShip()` → `destroyPlayerShip()` (sets `ship = null`) on a fatal hit. On boss levels the large boss + boss-spawned asteroids make fatal hits common, so `ship` was frequently null at that line → `TypeError: Cannot read properties of null (reading 'x')` halted the RAF loop (apparent "crash"). Fixed by adding a `ship &&` guard to the boss-collision condition. Reproduced and verified fixed live (forced repeated ship deaths during a boss fight → 0 errors).
- **Portrait grounding (content-aware)**: added cached `getContentBox(img)` (offscreen-canvas alpha bbox, 0..1 fractions). `drawCharacterSelectScreen` now anchors each portrait by its *visible pixels* — content bottom sits exactly on the nameplate top, horizontally centered on content — so Doof (head-and-shoulders bust w/ transparent bottom margin) and Devyn (full-body) both look grounded instead of floating/cut-off. Falls back to full-frame box if `getImageData` is tainted/throws.
- **Character-select keyboard**: ArrowLeft → Doof (index 0, left card), ArrowRight → Devyn (index 1, right card); plays pickup sound + nulls `titleFrameCache` only when the selection actually changes; `e.preventDefault()` scoped to the arrow branch.
- **Touch UI only on real touch devices**: `setupTouchControls` isTouch is now `matchMedia('(any-pointer: coarse)') && !matchMedia('(any-pointer: fine)')` (fallback to ontouchstart/maxTouchPoints when matchMedia absent). Desktops and touch-screen laptops (which have a fine pointer) keep mouse/keyboard controls, hide the on-screen buttons, and skip the landscape gate. Verified desktop → touchUI `display:none`.
- **Squad-reviewed** (code-review ×2): portrait fit math finite/guarded, getImageData try/catch fallback, cache invalidation matches mouse path, null-deref guard complete with no other unguarded `ship.` derefs in the same update pass.

### Session — part 5 — Mobile "zoomed in" fix (virtual render resolution)
- **Problem**: On small phone screens the ship/asteroids looked huge ("too zoomed in") because the game world was sized in raw screen pixels (`canvas.width = window.innerWidth`). Fixed-size objects (ship radius 69, asteroids 40-60px, ship image 175px) occupy a large fraction of a short viewport.
- **Fix (resize() only)**: render at a larger **virtual resolution** on small screens. `resize()` now computes `scale = clamp(REFERENCE_MIN_EDGE / min(innerW, innerH), 1, MAX_RENDER_SCALE)` with `REFERENCE_MIN_EDGE = 720`, `MAX_RENDER_SCALE = 2.4`. Backing store `canvas.width/height = physical * scale` (the logical coordinate space all game code already uses), while `canvas.style.width/height` is pinned to the physical viewport so the browser downscales it — the zoom-out effect. Desktop (short edge ≥ 720) keeps scale = 1, i.e. unchanged 1:1.
- **Pointer mapping**: `setupMenuPointer()`'s `pos()` now scales client coords into the virtual space (`* canvas.width / rect.width`) since backing store ≠ CSS size on phones. This is the only coordinate mapper; touch-hold buttons drive `keys[]` directly and needed no change.
- **Verified with agent-browser** (`--no-proxy` required locally): at a 566px-tall display the canvas backing store is 720 tall (scale 1.27); character-select, difficulty-select and gameplay all render correctly and crisply, ship/asteroids now occupy a desktop-like fraction of the field. Note: agent-browser's `--window-size` flag bypasses `--no-proxy` (proxy errors), so test at default window size + read dims via `eval`.

