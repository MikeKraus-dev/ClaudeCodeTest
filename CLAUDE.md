# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the Games

No build step — open any HTML file directly in a browser:

```
start tictactoe.html
start shooter.html
```

There are no dependencies, no package manager, and no server required.

## Git Workflow

**Commit and push after every meaningful unit of work** — do not batch up changes across multiple tasks. This keeps GitHub in sync at all times so work is never lost and any state can be reverted.

When to commit:
- After creating or completing a new feature
- After any bug fix
- After updating or creating any file (including this one)

```bash
git add <file>
git commit -m "short present-tense summary of what changed and why"
git push
```

Remote: https://github.com/MikeKraus-dev/ClaudeCodeTest

Good commit messages are specific: `"Add high score persistence to Pixel Siege"` not `"update game"`. Always include the `Co-Authored-By` trailer for Claude-assisted commits.

## Project Structure

Each game is a single self-contained HTML file: all CSS and JS live inline with no external references. New games should follow the same pattern.

## shooter.html Architecture

**Canvas resolution:** 320×240 logical pixels rendered at 960×720 via CSS (`image-rendering: pixelated`). All coordinates and mouse input must be scaled by `W / rect.width`.

**Game loop:** `requestAnimationFrame` with `dt = Math.min(elapsed/1000, 0.05)` — the cap prevents physics explosions when the tab is backgrounded.

**State machine:** `gstate` is one of `'MENU'`, `'PLAYING'`, `'GAME_OVER'`. `setState(s)` handles transitions; `initGame()` resets all mutable state.

**Sprite system:** Sprites are 2D arrays of palette indices (0 = transparent). `drawSprite(spr, x, y, pal, flipX)` renders them pixel-by-pixel via `fillRect`. The player body (`P_TOP`, 8×8) is static; only `P_LEGS[frame]` (rows 8–9) animates across 4 walk frames. Enemy sprites are 2-frame.

**Enemy definitions (`EDEFS`):** Each entry holds base stats, sprite frames, palette, movement pattern (`'direct'` or `'orbit'`), and spawn weight. To add an enemy type, add an entry here and its sprite arrays above it.

**Wave scaling (`waveCfg(n)`):** Returns total enemy count (`5 + n*3`), spawn interval (floors at 0.4s), speed multiplier, and per-type weights. Tanks appear at wave 4+, Shooters at wave 3+.

**Collision:** Circle vs circle (`circle2`). All three cases — player bullets→enemies, enemy bullets→player, enemy bodies→player — are resolved in `resolveCollisions()` using backward-index loops so `splice` is safe.

**Draw order (PLAYING):** background → particles → enemies → player → bullets → HUD. The HUD (including damage flash overlay) always draws last.

**Key abbreviations used throughout:**
- `spd` = speed, `sCD` = shoot cooldown, `sRate` = shoot rate
- `iTimer` = invincibility timer, `flashT` = hit-flash timer
- `fTimer`/`fDur` = animation frame timer/duration
- `sz` = collision radius, `pts` = score points, `pat` = movement pattern
- `wCfg` = current wave config, `sw` = spawn weight

## tictactoe.html Architecture

DOM-based (no canvas). Board state is a flat `Array(9)`. The computer player uses full minimax — it is unbeatable. Switching modes (vs Computer ↔ vs Human) via `btnMode` resets scores and calls `init()`.
