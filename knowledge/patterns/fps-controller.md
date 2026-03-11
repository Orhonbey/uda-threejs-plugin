---
title: FPS Controller
since: "r1"
stable: "r150"
threejs_docs:
  - https://threejs.org/docs/#api/en/cameras/PerspectiveCamera
api_refs:
  - class: PerspectiveCamera
    url: https://threejs.org/docs/#api/en/cameras/PerspectiveCamera
  - class: Euler
    url: https://threejs.org/docs/#api/en/math/Euler
last_verified: "2026-03-12"
---

# FPS Controller

## Overview

A first-person shooter controller needs: Pointer Lock for mouse capture, WASD movement relative to camera yaw, vertical mouse look with pitch clamping, and jump with gravity. Three.js has no built-in FPS controller — you build it from the camera and DOM events.

## Key Concepts

### Pointer Lock API
- `canvas.requestPointerLock()` — captures mouse cursor
- `document.pointerLockElement` — check if locked
- `mousemove` event gives `movementX`/`movementY` — raw delta, not position
- ESC releases the lock automatically

### Yaw/Pitch with YXZ Order
- Set `camera.rotation.order = 'YXZ'` before any rotation
- **Yaw** (Y-axis) — horizontal look, unrestricted
- **Pitch** (X-axis) — vertical look, clamp to prevent flipping: `[-π/2, π/2]`

### Movement Relative to Camera
- Forward vector: `(-sin(yaw), 0, -cos(yaw))`
- Right vector: `(cos(yaw), 0, -sin(yaw))`
- Normalize diagonal movement to prevent speed boost

### Jump & Gravity
- Simple velocity-based: `vy += -GRAVITY * dt; y += vy * dt`
- Ground check: `if (y <= PLAYER_HEIGHT) { y = PLAYER_HEIGHT; vy = 0; onGround = true }`

## Code Examples

### Complete FPS Controller

```javascript
const MOVE_SPEED = 6;
const MOUSE_SENSITIVITY = 0.002;
const GRAVITY = 20;
const JUMP_FORCE = 8;
const PLAYER_HEIGHT = 1.5;

// Input state
const activeInputs = new Set();

function updateInputState(event, pressed) {
  const key = event.code.toLowerCase();
  const aliases = new Set([key]);

  if (key === 'keyw' || key === 'arrowup') aliases.add('forward');
  if (key === 'keys' || key === 'arrowdown') aliases.add('backward');
  if (key === 'keya') aliases.add('left');
  if (key === 'keyd') aliases.add('right');
  if (key === 'space') aliases.add('jump');

  for (const a of aliases) {
    if (pressed) activeInputs.add(a); else activeInputs.delete(a);
  }
}

document.addEventListener('keydown', e => updateInputState(e, true));
document.addEventListener('keyup', e => updateInputState(e, false));

// Pointer Lock
canvas.addEventListener('click', () => canvas.requestPointerLock());

document.addEventListener('mousemove', (e) => {
  if (document.pointerLockElement !== canvas) return;
  player.yaw -= e.movementX * MOUSE_SENSITIVITY;
  player.pitch -= e.movementY * MOUSE_SENSITIVITY;
  player.pitch = Math.max(-Math.PI / 2, Math.min(Math.PI / 2, player.pitch));
});

// Update loop
function updatePlayer(dt) {
  const p = player;

  // Movement
  const moveF = (activeInputs.has('forward') ? 1 : 0)
              - (activeInputs.has('backward') ? 1 : 0);
  const moveR = (activeInputs.has('right') ? 1 : 0)
              - (activeInputs.has('left') ? 1 : 0);

  if (moveF || moveR) {
    const fx = -Math.sin(p.yaw), fz = -Math.cos(p.yaw);
    const rx = Math.cos(p.yaw),  rz = -Math.sin(p.yaw);
    let dx = fx * moveF + rx * moveR;
    let dz = fz * moveF + rz * moveR;
    const len = Math.hypot(dx, dz);
    if (len > 0) {
      dx = (dx / len) * MOVE_SPEED * dt;
      dz = (dz / len) * MOVE_SPEED * dt;
      moveWithCollision(p, dx, dz, mapData);
    }
  }

  // Jump & gravity
  if (activeInputs.has('jump') && p.onGround) {
    p.vy = JUMP_FORCE;
    p.onGround = false;
  }
  p.vy -= GRAVITY * dt;
  p.y += p.vy * dt;
  if (p.y <= PLAYER_HEIGHT) {
    p.y = PLAYER_HEIGHT;
    p.vy = 0;
    p.onGround = true;
  }

  // Sync camera
  camera.position.set(p.x, p.y, p.z);
  camera.rotation.set(p.pitch, p.yaw, 0);
}
```

## Best Practices

- **Normalize diagonal movement** — `Math.hypot` then divide, prevents √2 speed boost
- **Clamp pitch** to `[-π/2, π/2]` — prevents camera flipping upside down
- **Use Set for input** — O(1) lookup, handles multiple keys pressed simultaneously
- **Clear inputs on pointer lock loss** — prevents stuck keys
- **Sensitivity ~0.002** — good default, make configurable

## Common Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Not setting `rotation.order = 'YXZ'` | Gimbal lock, wrong rotation | Set before any rotation |
| No diagonal normalization | 41% faster diagonal movement | Normalize direction vector |
| Not clamping pitch | Camera flips upside down | Clamp to ±π/2 |
| Using `event.key` for movement | Varies by keyboard layout | Use `event.code` (physical key) |
| Not clearing inputs on blur | Keys stuck after alt-tab | Clear Set on `blur` / pointer lock exit |
