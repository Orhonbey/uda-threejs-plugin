---
title: Game Loop & Delta Time
since: "r1"
stable: "r1"
threejs_docs:
  - https://threejs.org/docs/#api/en/core/Clock
api_refs:
  - class: Clock
    url: https://threejs.org/docs/#api/en/core/Clock
last_verified: "2026-03-12"
---

# Game Loop & Delta Time

## Overview

Browser games use `requestAnimationFrame` (rAF) for the main loop. Unlike Unity's `Update()` which is called by the engine, in Three.js you manage the loop yourself. Delta time is critical for frame-rate-independent movement.

## Key Concepts

### requestAnimationFrame
- Called by the browser ~60fps (or monitor refresh rate)
- Automatically pauses when tab is hidden (saves CPU/battery)
- Returns a timestamp in milliseconds

### Delta Time
- Time elapsed since last frame, in seconds
- All movement/animation must multiply by `dt` for consistency
- Cap dt to prevent large jumps after tab switches: `Math.min(dt, 0.1)`

### THREE.Clock
Built-in helper that tracks elapsed time and delta:
- `clock.getDelta()` — seconds since last call
- `clock.getElapsedTime()` — total seconds since clock start

## Code Examples

### Standard Game Loop

```javascript
const clock = new THREE.Clock();

function gameLoop() {
  requestAnimationFrame(gameLoop);
  const dt = Math.min(clock.getDelta(), 0.1); // cap at 100ms

  updatePlayer(dt);
  updateBots(dt);
  updateTracers(dt);

  renderer.render(scene, camera);
}

gameLoop();
```

### Manual Timestamp Approach

```javascript
let lastTime = performance.now();

function gameLoop(now) {
  requestAnimationFrame(gameLoop);
  const dt = Math.min((now - lastTime) / 1000, 0.1);
  lastTime = now;

  update(dt);
  renderer.render(scene, camera);
}

requestAnimationFrame(gameLoop);
```

### Game State Machine

```javascript
const state = { mode: 'menu' }; // 'menu' | 'playing' | 'dead'

function gameLoop() {
  requestAnimationFrame(gameLoop);
  const dt = Math.min(clock.getDelta(), 0.1);

  if (state.mode === 'playing') {
    updatePlayer(dt);
    updateBots(dt);
  }

  renderer.render(scene, camera);
  updateHUD();
}
```

## Best Practices

- **Always cap delta time** — prevents physics explosions after tab switch
- **Single render call** — only one `renderer.render()` per frame
- **State-driven updates** — skip game logic when paused/menu/dead
- **Separate update and render** — keep logic out of render functions
- **Use named constants** — `MOVE_SPEED * dt`, not `6 * dt`

## Common Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Not using delta time | Movement varies with framerate | Multiply all rates by `dt` |
| Not capping delta | Huge jump after tab switch | `Math.min(dt, 0.1)` |
| Multiple render calls | Wasted GPU work | One `renderer.render()` per frame |
| Logic in rAF callback body | Hard to test, hard to pause | Extract to `update(dt)` function |
| Using `setInterval` for game loop | Doesn't sync with display, wastes battery | Use `requestAnimationFrame` |

## Timer Pattern

```javascript
// Countdown timer with delta time
player.shootCooldown = Math.max(0, player.shootCooldown - dt);
if (player.shootCooldown <= 0) {
  // Can shoot again
}

// Decay animation
player.weaponKick = Math.max(0, player.weaponKick - dt * 5);
```
