---
title: Scene, Camera & Renderer Setup
since: "r50"
stable: "r150"
threejs_docs:
  - https://threejs.org/docs/#api/en/scenes/Scene
  - https://threejs.org/docs/#api/en/cameras/PerspectiveCamera
  - https://threejs.org/docs/#api/en/renderers/WebGLRenderer
api_refs:
  - class: Scene
    url: https://threejs.org/docs/#api/en/scenes/Scene
  - class: PerspectiveCamera
    url: https://threejs.org/docs/#api/en/cameras/PerspectiveCamera
  - class: WebGLRenderer
    url: https://threejs.org/docs/#api/en/renderers/WebGLRenderer
  - class: AmbientLight
    url: https://threejs.org/docs/#api/en/lights/AmbientLight
  - class: DirectionalLight
    url: https://threejs.org/docs/#api/en/lights/DirectionalLight
last_verified: "2026-03-12"
---

# Scene, Camera & Renderer Setup

## Overview

Every Three.js application starts with three core objects: a Scene (container for all 3D objects), a Camera (the viewpoint), and a WebGLRenderer (draws the scene to a canvas). Setting these up correctly is the foundation for all rendering.

## Key Concepts

### Scene
The scene is the root container. All meshes, lights, and groups must be added to the scene (or a child of the scene) to be rendered.

- `scene.background` — set a solid color or skybox
- `scene.fog` — distance-based fog for atmosphere
- `scene.add(object)` / `scene.remove(object)` — manage scene graph

### PerspectiveCamera
Simulates human vision with a field-of-view angle.

- **fov**: field of view in degrees (70–90 typical for games)
- **aspect**: `window.innerWidth / window.innerHeight`
- **near/far**: clipping planes (0.1–100 for small scenes)
- `camera.rotation.order = 'YXZ'` — required for FPS-style yaw/pitch rotation

### WebGLRenderer
Draws the scene using WebGL.

- `antialias: true` — smoother edges (slight perf cost)
- `setPixelRatio(Math.min(devicePixelRatio, 2))` — cap at 2x for performance
- `setSize(width, height)` — match canvas to window

### Lighting
Most scenes need at least two lights:
- **AmbientLight** — flat fill, no shadows (intensity 0.4–0.7)
- **DirectionalLight** — sun-like, casts parallel rays (intensity 0.6–1.0)

## Code Examples

### Minimal Scene Setup

```javascript
import * as THREE from 'three';

const canvas = document.getElementById('game');

const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87CEEB); // sky blue

const camera = new THREE.PerspectiveCamera(
  90,
  window.innerWidth / window.innerHeight,
  0.1,
  100
);
camera.rotation.order = 'YXZ'; // FPS yaw/pitch

const renderer = new THREE.WebGLRenderer({ canvas, antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

// Lighting
scene.add(new THREE.AmbientLight(0xffffff, 0.6));
const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
dirLight.position.set(10, 20, 10);
scene.add(dirLight);

// Add camera to scene (required if camera has children like weapon view)
scene.add(camera);
```

### Adding Fog

```javascript
scene.fog = new THREE.Fog(0x87ceeb, 20, 50);
// color matches background, starts at 20 units, fully opaque at 50
```

### Responsive Resize

```javascript
window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
```

## Best Practices

- **Match fog color to background** — prevents visible seam at fog boundary
- **Cap pixel ratio at 2** — 3x+ pixel ratios tank performance with minimal visual gain
- **Set rotation order before rotating** — changing order after setting rotation gives unexpected results
- **Add camera to scene** when it has children (weapon view, HUD elements)
- **Use `dispose()`** when removing objects — prevents GPU memory leaks

## Common Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Not setting camera aspect on resize | Stretched/squished view | Update aspect + `updateProjectionMatrix()` |
| Forgetting `scene.add(camera)` | Camera children invisible | Always add camera to scene if it has children |
| Near plane too small (0.001) | Z-fighting artifacts | Use 0.1 for most games |
| Far plane too large (100000) | Depth buffer precision loss | Keep as small as needed |
| Not capping pixel ratio | Very slow on high-DPI mobile | `Math.min(devicePixelRatio, 2)` |
