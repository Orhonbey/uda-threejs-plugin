---
title: InstancedMesh for Voxel Worlds
since: "r109"
stable: "r120"
threejs_docs:
  - https://threejs.org/docs/#api/en/objects/InstancedMesh
api_refs:
  - class: InstancedMesh
    url: https://threejs.org/docs/#api/en/objects/InstancedMesh
  - class: Object3D
    url: https://threejs.org/docs/#api/en/core/Object3D
  - class: Matrix4
    url: https://threejs.org/docs/#api/en/math/Matrix4
last_verified: "2026-03-12"
---

# InstancedMesh for Voxel Worlds

## Overview

InstancedMesh renders thousands of identical geometries in a single draw call. Instead of creating one Mesh per block (which creates one draw call each), InstancedMesh batches them all. This is the key to performant voxel worlds in Three.js.

## Key Concepts

### Why InstancedMesh?
- 1000 individual Mesh objects = 1000 draw calls = slow
- 1 InstancedMesh with 1000 instances = 1 draw call = fast
- All instances share the same geometry and material
- Each instance has its own transform (position/rotation/scale) via a Matrix4

### The Dummy Pattern
Use a temporary `Object3D` to build transform matrices:
1. Set `dummy.position`, `.rotation`, `.scale`
2. Call `dummy.updateMatrix()`
3. Copy `dummy.matrix` to the instance

### Two-Pass Construction
For voxel worlds built from a grid:
1. **Count pass** — iterate grid, count total wall blocks
2. **Build pass** — create InstancedMesh with exact count, set transforms

## Code Examples

### Voxel World from Grid

```javascript
function buildWorld(scene, mapKey) {
  const { width, height, wallHeight, grid } = MAPS[mapKey];

  // Pass 1: count instances
  let totalCount = 0;
  for (let z = 0; z < height; z++) {
    for (let x = 0; x < width; x++) {
      if (grid[z][x] === WALL) {
        totalCount += wallHeight;
      }
    }
  }

  // Create instanced mesh
  const boxGeo = new THREE.BoxGeometry(1, 1, 1);
  const blockMat = new THREE.MeshLambertMaterial({ map: createBlockTexture() });
  const instancedMesh = new THREE.InstancedMesh(boxGeo, blockMat, totalCount);

  // Pass 2: set transforms
  const dummy = new THREE.Object3D();
  let idx = 0;
  for (let z = 0; z < height; z++) {
    for (let x = 0; x < width; x++) {
      if (grid[z][x] === WALL) {
        for (let y = 0; y < wallHeight; y++) {
          dummy.position.set(x + 0.5, y + 0.5, z + 0.5);
          dummy.updateMatrix();
          instancedMesh.setMatrixAt(idx, dummy.matrix);
          idx++;
        }
      }
    }
  }

  instancedMesh.instanceMatrix.needsUpdate = true;
  scene.add(instancedMesh);
}
```

### Floor with PlaneGeometry

```javascript
const floorGeo = new THREE.PlaneGeometry(width, height);
const floorMat = new THREE.MeshLambertMaterial({ map: grassTexture });
const floor = new THREE.Mesh(floorGeo, floorMat);
floor.rotation.x = -Math.PI / 2; // lay flat
floor.position.set(width / 2, 0, height / 2); // center on grid
scene.add(floor);
```

## Best Practices

- **Count before creating** — InstancedMesh count is fixed at creation
- **Always call `instanceMatrix.needsUpdate = true`** after setting matrices
- **One material per InstancedMesh** — different looks need separate instances
- **Center blocks at (x + 0.5, y + 0.5, z + 0.5)** — aligns with integer grid
- **Reuse dummy Object3D** — create once, reuse for all instances

## Common Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Creating Mesh per block | Thousands of draw calls | Use InstancedMesh |
| Wrong instance count | Missing blocks or wasted memory | Count first, then create |
| Forgetting `needsUpdate` | Instances don't appear | Set `instanceMatrix.needsUpdate = true` |
| Not calling `updateMatrix()` | All instances at origin | Call `dummy.updateMatrix()` after positioning |
| Blocks not on grid | Visual gaps between blocks | Use `x + 0.5` centering |

## Performance Notes

- 10,000 instances = ~1 draw call (excellent)
- 100,000 instances = still 1 draw call but heavy on vertex processing
- For very large worlds, consider chunking (split into multiple InstancedMesh by region)
- InstancedMesh supports per-instance colors via `setColorAt()` for variety
