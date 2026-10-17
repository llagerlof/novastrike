# AGENTS.md

Guide for agents working on NOVA STRIKE, a vertical pixel-art shoot 'em up written in plain JavaScript (ES modules), canvas 2D and WebAudio. There are no dependencies, no build step, no asset files and no package manager. All sprites, sounds and music are generated from data in the source.

`README.md` is written for players. Keep it in sync when gameplay, controls, scoring or how-to-run changes.

## Running

- ES modules don't load over `file://`, so the game must be served over HTTP (e.g. `python3 -m http.server` from the repo root). `index.html` shows an explanatory message when opened as a file.
- `index.html` is only a page shell: CSS for scaling and scanlines, the canvas, the file:// warning, and the module entry `src/main.js`.

## Core model

- **Resolution**: a fixed logical canvas of 180×320, scaled up with CSS using `image-rendering: pixelated`. Integer scaling is preferred unless it would waste more than about 15% of the screen. All game coordinates are in logical pixels.
- **Fixed timestep**: the game updates at exactly 60 Hz from an accumulator in `main.js`, and renders once per animation frame. **All speeds, timers and cooldowns are per frame, not per second.** Durations are frame counts (60 = one second).
- **Shared state**: a single mutable object `G` in `core/state.js` holds the whole world: state machine fields, entity arrays, score and lives, screen effects, and the current difficulty. The player ship `P` is exported from the same module as an ES live binding. It may only be replaced through `resetPlayer()`, because importers can't reassign it.
- **Entities are plain objects** in arrays on `G` (`enemies`, `bullets`, `ebullets`, `pups`, `parts`, `rings`, `pops`, `beams`). They're removed by setting a `dead` flag and filtering the array after its update pass. There are no classes and no entity framework.
- **Bosses live in `G.enemies`** like any other enemy (flagged `isBoss`, with an `st` sub-state), and also in `G.boss`. Collision and drawing code treat them uniformly, with boss-specific branches.

## Game flow

Two nested state machines.

**Top level** (`G.state`, owned by `game.js`): `title` → `play` ⇄ `paused` → `gameover` → `title`. After idling on the title screen, the game starts a demo: `play` with `G.demo` set and an AI pilot. In the `gameover` state the world keeps animating in "fx only" mode (no player, no stage progression).

**Stage** (`G.phase`, owned by `world/director.js`): `intro` → `waves` → `warning` → `boss` → `bossDeath` → `clear` → next stage's `intro`. Stages are endless; the boss cycles every 3 stages, and each loop adds an "MK" suffix and scales HP and bullet density.

**Waves**: each stage builds a queue of wave definitions picked by weight from those unlocked at that stage, with two UFO waves inserted. A wave produces a list of timed spawn events. The next wave starts once the current one has spawned everything and at most one of its enemies remains, or after a timeout. Every enemy records which wave spawned it.

**Difficulty**: a curve keyed on stage number (enemy HP, bullet speed, fire rate, spawn density), stored in `G.diff` when a stage starts. Gameplay code reads these multipliers instead of checking the stage number, except where an enemy unlocks extra behaviour at a specific stage.

## Module map

Dependency direction is roughly `main → game → world/render → gfx/audio/core`. Circular imports exist between `world/` modules (e.g. enemies ↔ bosses ↔ player ↔ game). They're safe only because nothing in those modules is called while modules are still loading. **Keep module top-level code free of cross-module calls.**

### Entry and flow
- `src/main.js`: canvas resizing, auto-pause on blur or a hidden tab, the fixed-timestep loop, and the `window.NOVA` debug object.
- `src/game.js`: top-level state transitions (start game, game over, back to title, pause) and the per-frame update order. It's the only place the world is reset.
- `src/input.js`: merges keyboard, gamepad and pointer input into one `inp` object of held actions plus edge-triggered "pressed this frame" flags. Pointer control is relative dragging: the ship follows the finger's movement, not its position. A second finger, or a tap on the bomb icons, drops a bomb. Also exposes an "any activity" flag, used to exit demo mode.

### core/
- `config.js`: screen size, `TAU`, URL params. Safe to import outside a browser.
- `state.js`: `G`, `P`, the difficulty curve, stage background colours.
- `util.js`: random helpers, number padding, and a localStorage wrapper that never throws.

### gfx/
- `canvas.js`: the canvas, the 2D context, and a pixel-exact ring drawer (canvas arcs are anti-aliased, so they aren't used).
- `palette.js`: the single-character palette keys used by sprite strings, explosion colours, and text gradients.
- `sprites.js`: every sprite as an array of strings, one character per pixel, with `.` for transparent. Each sprite is pre-rendered to an offscreen canvas along with an all-white copy used for hit flashes. Boss sprites store only their left half and are mirrored. Enemy bullet sprites are templates recoloured into five colour sets and four sizes. A row-width mismatch logs a console warning.
- `font.js`: a hand-made 3×5 bitmap font, drawn with fillRect. Colours can be per-row gradients. Supports alignment, scale and drop shadow. Only uppercase letters, digits and a small set of punctuation exist; anything else is silently skipped.

### world/ (simulation)
- `director.js`: wave definitions, stage building, stage start (which also picks the stage's music), and the stage phase machine.
- `enemies.js`: the enemy stat table, spawning, one behaviour function per type, damage and death (score, chain, drops, asteroid splitting), nearest-target lookup, and the enemy update and cleanup pass (which also stops the UFO's hum loop).
- `bosses.js`: boss definitions (sprite, HP, hitbox, gun muzzle offsets, minion type, and attack pattern lists per phase), the attack pattern library, the boss sub-state machine (enter → fight → dying), phase changes at 66% and 33% HP, laser beams, and the boss-for-stage lookup used by the UI.
- `player.js`: movement (keyboard, pointer or demo AI), the shot spread table per power level, homing missiles from the option pods, bombs, getting hit (the shield absorbs one hit, otherwise death with a power-level penalty), and respawning.
- `shots.js`: enemy bullet emitters (single shot, fan, ring), the "may this enemy fire" rule, and the enemy bullet update pass (acceleration, sine-wave motion, hits, grazing).
- `powerups.js`: capsule kinds, the drop table, pickup magnet and collection, and their effects.
- `fx.js`: explosions, sparks, shockwave rings, floating score popups, adding score (including extra-life thresholds), and turning bullets into points.
- `background.js`: the three-layer parallax starfield, procedurally generated Bayer-dithered planets (some with rings), and the warp-speed effect between stages.

### render/ (drawing only, never mutates the simulation)
- `index.js`: composes a frame: background, world, HUD, overlays, then screen shake and flash.
- `world.js`: playfield drawing. Enemy bullets are drawn last so they stay readable.
- `hud.js`: score line, chain meter, boss name and health bar (with phase markers), lives, bombs, power and missile pips.
- `screens.js`: title screen, stage intro, boss warning, stage-clear tally, pause and game over.

### audio/
- `index.js`: the `AU` facade. **Gameplay code should only talk to audio through this.** It covers engine start-up, the sfx library, play/stop music, music variants, pause muffling, and the saved mute and music-off settings.
- `engine.js`: the WebAudio graph and synthesis primitives. Graph: music voices go to a music bus, then fade, then a pause low-pass filter, then a ducking gain, then a shared compressor. SFX voices go to an sfx bus and then the same compressor. Any voice can also feed reverb (a generated impulse response) and a tempo-synced filtered echo. Provides NES-style pulse waves (12.5%, 25%, 50% duty) built from Fourier series, pitched blips and filtered noise bursts with stereo pan, a music-ducking helper, the instrument table (ADSR, filter envelope, vibrato, sub-oscillator, chorus), and the synthesized drum kit.
- `sfx.js`: the sound effect library, keyed by event name. Most effects take the emitter's x position for stereo panning. Rapid-fire effects are rate-limited. Continuous loops (currently the UFO hum) return a handle with update/stop, and all live loops are tracked so the game can silence them in one call when the world resets.
- `music.js`: compiles the text songs into per-step event lists when the module loads, then plays them with a lookahead scheduler on the audio clock (independent of the game loop). It supports intro then loop, play-once jingles, per-section tempo, and switching to an alternate loop at the next section boundary. It resyncs if the tab was stalled. The compiler logs `[music]` warnings for malformed data.
- `songs.js`: pure data. Shared pattern tables for bass, arpeggios and drums, and the songs: title, three stage themes (picked by stage number), boss (with an "intense" loop used in the final phase), and the stage-clear and game-over jingles. The notation is fully documented in the comment at the top of the file. Melodies are explicit note names on a 16-steps-per-bar grid; bass and arpeggios come from each bar's chord plus a named pattern.

## Per-frame update order (play state)

Input poll → mute/music toggles → frame counter → background → shake and flash decay → stage director → player → player bullets → enemies (including boss) → enemy bullets → boss beams → power-ups → particles and popups → chain timer. Rendering happens separately, after the frame's updates.

## Conventions

- Match the existing style: two-space indentation, semicolons, single quotes, small plain functions, terse comments that explain *why*. Data tables (enemy stats, boss definitions, waves, sprites, songs) are the intended way to add content. Prefer adding rows over adding special cases.
- Sound is triggered from gameplay code at the moment the event happens. When adding an effect, pass the emitter's x for panning, and rate-limit anything that can fire many times per second.
- Anything that should stop when the world resets (a looping sound, timers) must be reachable from the world-reset path in `game.js`.
- Persistent data lives in localStorage under the `nova_` prefix: the high score and the audio settings.
- Colours come from `palette.js` or the established hex values. Keep the limited palette.
- Gameplay changes that affect players (controls, scoring numbers, power-ups, enemies, bosses) must be reflected in `README.md`.

## Pitfalls already hit

- **Duplicate object keys fail silently.** A jingle once had a "don't loop" flag and its loop list under the same key, and the second one won, so the jingle looped forever. The play-once flag is now named `once` for this reason.
- **Browsers block audio until a user gesture.** The audio engine is created lazily on the first key, tap or gamepad press. Music requested earlier is remembered and starts then.
- **Headless browsers don't run animation frames reliably**, so wall-clock screenshots are useless for testing gameplay. Drive the game with the frame-step debug hook instead.
- **The music runs on the audio clock**, so stepping game frames quickly doesn't advance the music. Music state checks need real time to pass.
- A sprite row with the wrong width only produces a console warning and a skewed sprite. Watch the console when editing art.

## Testing and debugging

There's no automated test suite. Verify changes by running the game, preferably with headless Chromium over the DevTools protocol.

- **URL params**: `stage=N` starts at a given stage, `boss` skips waves straight to the boss, `autoplay` runs the demo AI immediately, `god` makes the player invincible. Combined, these exercise any stage or boss unattended.
- **`window.NOVA`**: exposes `G` and `P`; a frame stepper that advances N updates synchronously and then renders (the reliable way to test in headless browsers); the current music position (song, loop list, section, queued variant); a function to start audio programmatically; and an offline audio renderer. The renderer plays a song, or every sound effect, into an OfflineAudioContext and returns peak, RMS, NaN count and per-second loudness. Use it on a freshly loaded page, before audio has started, to catch silence, clipping or broken voices without listening.
- Pure-data modules (`songs.js`, and `music.js`'s compiler) can be imported from Node to validate song data, since config access is guarded for non-browser environments.
- After any change, load the page and check the console for exceptions and `[music]` or sprite warnings.
