---
title: Materials & Lighting
since: "r1"
stable: "r150"
threejs_docs:
  - https://threejs.org/docs/#api/en/materials/MeshLambertMaterial
  - https://threejs.org/docs/#api/en/materials/MeshBasicMaterial
  - https://threejs.org/docs/#api/en/materials/MeshStandardMaterial
  - https://threejs.org/docs/#api/en/lights/PointLight
api_refs:
  - class: MeshBasicMaterial
    url: https://threejs.org/docs/#api/en/materials/MeshBasicMaterial
  - class: MeshLambertMaterial
    url: https://threejs.org/docs/#api/en/materials/MeshLambertMaterial
  - class: MeshStandardMaterial
    url: https://threejs.org/docs/#api/en/materials/MeshStandardMaterial
  - class: PointLight
    url: https://threejs.org/docs/#api/en/lights/PointLight
  - class: CanvasTexture
    url: https://threejs.org/docs/#api/en/textures/CanvasTexture
last_verified: "2026-03-12"
---

# Materials & Lighting

## Overview

Three.js provides a material hierarchy from cheap (MeshBasicMaterial) to physically accurate (MeshStandardMaterial). For voxel/blocky games, MeshLambertMaterial gives the best balance of visual quality and performance.

## Key Concepts

### Material Types (Performance Order)

| Material | Lighting | Cost | Use Case |
|----------|----------|------|----------|
| MeshBasicMaterial | None (unlit) | Lowest | Tracers, HUD, effects |
| MeshLambertMaterial | Diffuse only | Low | Voxel blocks, characters |
| MeshStandardMaterial | PBR (metalness/roughness) | Medium | Realistic surfaces |
| MeshPhysicalMaterial | Full PBR + clearcoat | High | Glass, car paint |

### Procedural Textures
Generate textures at runtime using Canvas2D — no external assets needed:

- Create a `<canvas>` element
- Draw pixel-by-pixel with `fillRect`
- Wrap in `THREE.CanvasTexture`
- Set `magFilter = THREE.NearestFilter` for pixel-art look

### Dynamic Lights
- **PointLight** — emits from a point in all directions (muzzle flash, fire)
- Toggle `intensity` to 0 to "turn off" without removing from scene
- Set `distance` to limit falloff range

## Code Examples

### Procedural Block Texture

```javascript
function createBlockTexture() {
  const size = 16;
  const canvas = document.createElement('canvas');
  canvas.width = size;
  canvas.height = size;
  const ctx = canvas.getContext('2d');

  // Per-pixel noise
  const base = 130;
  for (let y = 0; y < size; y++) {
    for (let x = 0; x < size; x++) {
      const noise = (Math.random() - 0.5) * 30;
      const v = Math.max(0, Math.min(255, (base + noise) | 0));
      ctx.fillStyle = `rgb(${v},${v},${v})`;
      ctx.fillRect(x, y, 1, 1);
    }
  }

  // Border lines
  ctx.fillStyle = 'rgb(90,90,90)';
  ctx.fillRect(0, 0, size, 1);
  ctx.fillRect(0, size - 1, size, 1);
  ctx.fillRect(0, 0, 1, size);
  ctx.fillRect(size - 1, 0, 1, size);

  const texture = new THREE.CanvasTexture(canvas);
  texture.magFilter = THREE.NearestFilter;
  return texture;
}
```

### Tiling Ground Texture

```javascript
function createGrassTexture() {
  const size = 64;
  const canvas = document.createElement('canvas');
  canvas.width = size;
  canvas.height = size;
  const ctx = canvas.getContext('2d');

  for (let y = 0; y < size; y++) {
    for (let x = 0; x < size; x++) {
      const g = 100 + Math.random() * 50;
      const r = 60 + Math.random() * 40;
      const b = 30 + Math.random() * 20;
      ctx.fillStyle = `rgb(${r | 0},${g | 0},${b | 0})`;
      ctx.fillRect(x, y, 1, 1);
    }
  }

  const texture = new THREE.CanvasTexture(canvas);
  texture.wrapS = THREE.RepeatWrapping;
  texture.wrapT = THREE.RepeatWrapping;
  texture.repeat.set(16, 16);
  texture.magFilter = THREE.NearestFilter;
  return texture;
}
```

### Dynamic Muzzle Flash Light

```javascript
const muzzleFlash = new THREE.PointLight(0xff8c00, 0, 3);
muzzleFlash.position.set(0, 0, -0.45); // barrel tip
weaponGroup.add(muzzleFlash);

// In update loop:
if (shooting) {
  muzzleFlash.intensity = kick * 3;
} else {
  muzzleFlash.intensity = 0;
}
```

## Best Practices

- **Use MeshLambertMaterial** for voxel games — MeshStandardMaterial is overkill
- **Use MeshBasicMaterial** for unlit objects — tracers, effects, UI elements
- **NearestFilter for pixel art** — prevents blurry upscaling
- **RepeatWrapping for ground** — tiles the texture across large surfaces
- **Toggle intensity, don't add/remove lights** — cheaper than scene graph changes

## Common Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| MeshStandardMaterial everywhere | Slow on mobile/low-end | Use Lambert for simple objects |
| Not setting NearestFilter | Blurry pixel-art textures | `texture.magFilter = THREE.NearestFilter` |
| Forgetting `needsUpdate` | Texture changes not visible | Set `texture.needsUpdate = true` after modifying |
| Too many PointLights | Performance drops sharply | Max 3-4 active per scene |
| Not disposing materials | GPU memory leak | Call `material.dispose()` on removal |
