# slopsmith-plugin-minigames

The minigame framework for [Slopsmith](https://github.com/byrongamatos/slopsmith).

This plugin provides:

- A **Minigames hub** screen that discovers every installed minigame plugin and lists them as tiles with leaderboards.
- A **shared profile** (XP, level, unlocks, totals) that aggregates runs across every minigame.
- A JS **SDK** exposed at `window.slopsmithMinigames` that minigame plugins use to access scoring, HUD primitives, run persistence, and a scheduler — so individual minigames do not need their own DSP or backend.

## Writing a minigame

A minigame is a standard Slopsmith plugin that:

1. Adds a `minigame` block to its `plugin.json`:

   ```json
   {
     "id": "my_game",
     "name": "My Game",
     "version": "0.1.0",
     "script": "game.js",
     "minigame": {
       "title": "My Game",
       "tagline": "Short pitch",
       "type": "chart-free",
       "scoring": "pitch-continuous",
       "thumbnail": "thumb.png"
     }
   }
   ```

2. On script load, registers itself with the SDK using the safe late-binding
   pattern (minigame plugins may load before the SDK; the pending queue and
   the `slopsmith-minigames-ready` event handle both orderings):

   ```js
   const spec = {
     id: 'my_game',
     start: ({ container, modifiers, sdk }) => { /* mount game into container */ },
     stop:  () => { /* tear down */ },
   };

   if (window.slopsmithMinigames) {
     window.slopsmithMinigames.register(spec);
   } else {
     (window.__slopsmithMinigamesPending = window.__slopsmithMinigamesPending || []).push(spec);
   }
   ```

3. Calls `window.slopsmithMinigames.end({ score, durationMs, modifiers, meta })` when the run ends.

See [`slopsmith-plugin-flappy-bend`](https://github.com/byrongamatos/slopsmith-plugin-flappy-bend) for a working example.

## SDK reference

`window.slopsmithMinigames` exposes:

- `register(spec)` — declare a minigame
- `start(gameId, opts)` / `end(result)` — lifecycle
- `scoring.createContinuous(opts)` — emits per-frame pitch (YIN, ~60 Hz)
- `scoring.createDiscrete(opts)` — wraps `createNoteDetector`, emits hit/miss
- `scoring.createChord(opts)` — full chord scorer (delegates to note_detect)
- `ui.mountHUD(html)` / `ui.runSummary(result)` / `ui.modifierPicker(defs)`
- `submitRun(...)` / `getLeaderboard(...)` / `getProfile()`
- `scheduler.every(ms, cb)` / `in(ms, cb)` / `cancel(id)`

## Dependencies

- `slopsmith-plugin-notedetect` >= 1.10.0 — required for discrete/chord scoring modes (continuous mode is self-contained).
