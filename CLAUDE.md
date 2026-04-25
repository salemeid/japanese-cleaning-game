# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A self-contained 2D retro top-down browser game — no build step, no dependencies. The entire project is a single file: `game.html`.

Open it directly in any browser (`file://` or a local server). There is no dev server, no bundler, and no package manager.

## Git Workflow

Every change must be committed with a clean message and pushed to GitHub:

```bash
git add game.html
git commit -m "type: short summary"
git push
```

Commit types: `feat` (new feature), `fix` (bug fix), `refactor` (restructure without behavior change). Remote: `https://github.com/salemeid/japanese-cleaning-game`.

## Architecture: game.html

All game code lives in one `<script>` block. The structure is:

| Section | Purpose |
|---|---|
| Constants | Canvas size, wall thickness, bin positions, game limits |
| State | `lady`, `kid`, `garbageList`, `sparks`, `phase`, `score`, `frame` |
| `px(x,y,w,h,c)` | Core drawing primitive — all visuals are colored rectangles |
| `drawRoom()` | Tatami tiles, wood walls, sliding door decoration |
| `drawBins()` | Four corner waste bins with glow when `phase === 'full'` |
| `drawGarbage()` | 5 garbage types drawn with `px()` calls, centered on `g.x/g.y` |
| `drawLadySprite(clothLen)` | Canonical right-facing lady sprite (origin = center) |
| `drawLady()` | Translates/rotates canvas then calls `drawLadySprite` |
| `drawBabySprite(dropping)` | Canonical right-facing baby sprite (origin = center) |
| `drawKid()` | Translates/rotates canvas then calls `drawBabySprite` |
| `drawHUD()` | Top bar: garbage counter + progress bar, score, full warning |
| `updateLady()` | WASD/arrow key input, clamped movement, direction tracking |
| `updateKid()` | Random-walk AI, wall bounce, timed garbage drops |
| `checkCollisions()` | AABB lady↔garbage collection; lady↔bin emptying |
| `loop()` | `requestAnimationFrame` loop: update → clear → draw |

**Direction rendering pattern** — both characters use the same approach: draw a single canonical right-facing sprite at local origin (0,0), then use `ctx.save/translate/rotate or scale/restore` to orient it:
- Left: `ctx.scale(-1, 1)`
- Down: `ctx.rotate(Math.PI / 2)`
- Up: `ctx.rotate(-Math.PI / 2)`

**Game state machine**: `phase` is either `'playing'` or `'full'`. It flips to `'full'` when `lady.count >= 50`; back to `'playing'` when the lady enters a corner bin.
