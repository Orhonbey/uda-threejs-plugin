---
title: Sprites & Visual Effects
since: "r1"
stable: "r150"
threejs_docs:
  - https://threejs.org/docs/#api/en/objects/Sprite
  - https://threejs.org/docs/#api/en/materials/SpriteMaterial
  - https://threejs.org/docs/#api/en/constants/BlendingModes
api_refs:
  - class: Sprite
    url: https://threejs.org/docs/#api/en/objects/Sprite
  - class: SpriteMaterial
    url: https://threejs.org/docs/#api/en/materials/SpriteMaterial
  - class: CylinderGeometry
    url: https://threejs.org/docs/#api/en/geometries/CylinderGeometry
last_verified: "2026-03-12"
---

# Sprites & Visual Effects

## Overview

Sprites are always-facing-camera billboards, perfect for effects like muzzle flashes, particles, and health bars. Combined with AdditiveBlending and thin CylinderGeometry for tracers, they create convincing combat feedback without expensive particle systems.

## Key Concepts

### Sprites
- Always face the camera (billboard behavior)
- Rendered as a textured quad
- Cheaper than mesh-based effects
- Scale with `.scale.set(w, h, 1)` — Z is ignored

### AdditiveBlending
- Adds sprite color to background (brightens)
- Perfect for fire, sparks, muzzle flash, lasers
- Set `depthWrite: false` to prevent occluding objects behind

### Tracer Lines
- Use thin CylinderGeometry (radius 0.02) as bullet tracers
- Rotate to align with direction using `lookAt()` and `rotateX(Math.PI/2)`
- MeshBasicMaterial (unlit) so they glow regardless of lighting
- Short lifetime (~0.15s) with opacity fade

## Code Examples

### Muzzle Flash Sprite

```javascript
const flashSprite = new THREE.Sprite(
  new THREE.SpriteMaterial({
    color: 0xffaa33,
    transparent: true,
    opacity: 0.9,
    blending: THREE.AdditiveBlending,
    depthWrite: false,
  })
);
flashSprite.scale.set(0.15, 0.15, 1);
flashSprite.position.set(0, 0, -0.45); // barrel tip
flashSprite.visible = false;
weaponGroup.add(flashSprite);

// In update:
if (kick > 0.7) {
  flashSprite.visible = true;
  flashSprite.material.opacity = (kick - 0.7) * 3;
  const s = 0.1 + kick * 0.08;
  flashSprite.scale.set(s, s, 1);
} else {
  flashSprite.visible = false;
}
```

### Bullet Tracer

```javascript
// Shared geometry and material (reuse for all tracers)
const tracerGeo = new THREE.CylinderGeometry(0.02, 0.02, 1, 6);
tracerGeo.rotateX(Math.PI / 2); // align with Z-axis
const tracerMat = new THREE.MeshBasicMaterial({
  color: 0xffcc44,
  transparent: true,
  opacity: 0.9,
});

function createTracer(start, end, lifetime = 0.15) {
  const tracer = new THREE.Mesh(tracerGeo, tracerMat.clone());
  const mid = new THREE.Vector3().lerpVectors(start, end, 0.5);
  const length = start.distanceTo(end);

  tracer.position.copy(mid);
  tracer.scale.set(1, 1, length);
  tracer.lookAt(end);

  scene.add(tracer);
  return { mesh: tracer, life: lifetime };
}

// In update loop:
for (let i = tracers.length - 1; i >= 0; i--) {
  tracers[i].life -= dt;
  tracers[i].mesh.material.opacity = tracers[i].life / 0.15;
  if (tracers[i].life <= 0) {
    scene.remove(tracers[i].mesh);
    tracers[i].mesh.material.dispose();
    tracers.splice(i, 1);
  }
}
```

### Billboard Health Bar

```javascript
function createHealthBar() {
  const group = new THREE.Group();

  const bg = new THREE.Mesh(
    new THREE.BoxGeometry(0.6, 0.06, 0.02),
    new THREE.MeshBasicMaterial({ color: 0x000000, transparent: true, opacity: 0.7 })
  );
  group.add(bg);

  const fill = new THREE.Mesh(
    new THREE.BoxGeometry(0.58, 0.04, 0.02),
    new THREE.MeshBasicMaterial({ color: 0x00ff00 })
  );
  fill.position.z = 0.005;
  group.add(fill);

  group.userData.fill = fill;
  return group;
}

// Billboard: face camera every frame
healthBar.lookAt(camera.position);
```

## Best Practices

- **AdditiveBlending + depthWrite:false** for all glow effects
- **Toggle `visible`**, don't add/remove from scene — cheaper
- **Share geometry** across all tracers — clone material only if opacity varies
- **Reverse-iterate** when removing from arrays — prevents index skipping
- **Short lifetimes** (0.1–0.3s) for tracers — longer looks wrong

## Common Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| No `depthWrite: false` on additive sprites | Dark halos around edges | Add `depthWrite: false` |
| Not disposing tracer materials | Memory leak over time | `material.dispose()` on removal |
| Forward-iterating splice | Skips elements | Iterate backwards with `i--` |
| Tracer scale.y instead of scale.z | Wrong axis stretching | Depends on geometry rotation |
| Creating new geometry per tracer | GC pressure, slow | Share one `CylinderGeometry` |
