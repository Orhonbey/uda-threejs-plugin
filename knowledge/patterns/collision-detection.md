---
title: Collision Detection & Raycasting
since: "r1"
stable: "r150"
threejs_docs:
  - https://threejs.org/docs/#api/en/core/Raycaster
api_refs:
  - class: Raycaster
    url: https://threejs.org/docs/#api/en/core/Raycaster
  - class: Vector3
    url: https://threejs.org/docs/#api/en/math/Vector3
last_verified: "2026-03-12"
---

# Collision Detection & Raycasting

## Overview

For grid-based voxel games, collision detection uses the grid itself — no physics engine needed. Movement collision checks corner points against the grid, while shooting uses step-based raycasting along the line of sight.

## Key Concepts

### Grid-Based Wall Collision
Instead of complex physics, check if the entity's bounding corners overlap a wall cell:
- Check 4 corners at entity radius
- Process X and Z axes independently — enables wall sliding
- Integer floor converts world position to grid cell

### Wall Sliding
By testing X and Z movement separately:
1. Try moving on X axis alone → if no wall, apply X
2. Try moving on Z axis alone → if no wall, apply Z

This lets entities slide along walls instead of stopping dead.

### Line-of-Sight Raycasting
Step along a line at fine intervals checking each point against the grid:
- `steps = ceil(distance * 8)` — 8 checks per unit for accuracy
- Returns boolean (has LOS) or hit point (for tracer clipping)

## Code Examples

### Grid Wall Check

```javascript
function isWallAt(mapData, worldX, worldZ) {
  return getCell(mapData, Math.floor(worldX), Math.floor(worldZ)) === WALL;
}

function getCell(mapData, x, z) {
  if (x < 0 || z < 0 || x >= mapData.width || z >= mapData.height) return WALL;
  return mapData.grid[z][x];
}
```

### Sliding Collision

```javascript
function moveWithCollision(entity, dx, dz, mapData, radius = 0.3) {
  function touchesWall(x, z, r, md) {
    return (
      isWallAt(md, x - r, z - r) ||
      isWallAt(md, x + r, z - r) ||
      isWallAt(md, x - r, z + r) ||
      isWallAt(md, x + r, z + r)
    );
  }

  // X axis independently
  if (!touchesWall(entity.x + dx, entity.z, radius, mapData)) {
    entity.x += dx;
  }

  // Z axis independently
  if (!touchesWall(entity.x, entity.z + dz, radius, mapData)) {
    entity.z += dz;
  }
}
```

### Line-of-Sight Check

```javascript
function hasLineOfSight(x0, z0, x1, z1) {
  const dx = x1 - x0, dz = z1 - z0;
  const distance = Math.hypot(dx, dz);
  const steps = Math.max(1, Math.ceil(distance * 8));

  for (let i = 1; i < steps; i++) {
    const t = i / steps;
    if (isWallAt(mapData, x0 + dx * t, z0 + dz * t)) return false;
  }
  return true;
}
```

### Find Wall Hit Point (Tracer Clipping)

```javascript
function findWallHit(x0, z0, x1, z1) {
  const dx = x1 - x0, dz = z1 - z0;
  const distance = Math.hypot(dx, dz);
  const steps = Math.max(1, Math.ceil(distance * 8));

  for (let i = 1; i < steps; i++) {
    const t = i / steps;
    if (isWallAt(mapData, x0 + dx * t, z0 + dz * t)) {
      const hitT = Math.max(0, (i - 1) / steps);
      return { x: x0 + dx * hitT, z: z0 + dz * hitT, t: hitT };
    }
  }
  return null; // no wall hit
}
```

## Best Practices

- **4-corner check** covers rectangular entities without full AABB
- **Independent axis testing** gives free wall sliding
- **8 steps per unit** balances accuracy vs performance for LOS
- **Treat out-of-bounds as wall** — prevents entities escaping the map
- **Use `Math.floor`** for world-to-grid conversion, not `Math.round`

## Common Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Single-point collision | Entity clips into walls | Check all 4 corners at radius |
| Combined X+Z test | Entity stops at wall corners | Test axes independently |
| Too few raycast steps | Bullets pass through thin walls | `ceil(distance * 8)` minimum |
| `Math.round` for grid | Off-by-one at cell boundaries | Use `Math.floor` |
| Not handling out-of-bounds | Entity falls out of world | Return WALL for OOB coordinates |
