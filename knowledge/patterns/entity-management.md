---
title: Entity & Bot Management
since: "r1"
stable: "r150"
threejs_docs:
  - https://threejs.org/docs/#api/en/objects/Group
api_refs:
  - class: Group
    url: https://threejs.org/docs/#api/en/objects/Group
  - class: BoxGeometry
    url: https://threejs.org/docs/#api/en/geometries/BoxGeometry
last_verified: "2026-03-12"
---

# Entity & Bot Management

## Overview

Game entities (bots, NPCs) combine a data object (position, health, state) with a Three.js Group (visual model). The pattern: store game state in plain objects, sync 3D transforms each frame, and use Group.userData for cross-referencing.

## Key Concepts

### Entity = Data + Visual
- **Data object**: `{ x, z, health, alive, shootCooldown, ... }` — plain JS, no Three.js
- **Visual group**: `THREE.Group` with character meshes, health bar, effects
- Sync: `group.position.set(data.x, 0, data.z)` each frame

### Blocky Character Models
Build Minecraft-style characters from BoxGeometry primitives:
- Head, body, arms (pivoted at shoulder), legs (pivoted at hip)
- Pivot groups enable limb rotation for walk animation
- Color palettes via MeshLambertMaterial for variety

### Walk Animation
- Increment a `walkCycle` counter based on movement distance
- `sin(walkCycle)` drives arm/leg swing on X rotation
- No keyframes or animation mixer needed

### Spawn/Respawn Pattern
- On death: hide mesh, start respawn timer
- On respawn: reset health, pick spawn point, show mesh
- Dispose geometry/material only when permanently removing entity

## Code Examples

### Bot Data Structure

```javascript
function spawnBot(index) {
  const spawn = mapData.spawns[index % mapData.spawns.length];
  const palette = SKIN_PALETTES[index % SKIN_PALETTES.length];
  const group = createCharacter(palette);
  group.position.set(spawn.x, 0, spawn.z);
  scene.add(group);

  const healthBar = createHealthBar();
  group.add(healthBar);

  return {
    id: index,
    x: spawn.x,
    z: spawn.z,
    health: BOT_MAX_HEALTH,
    alive: true,
    respawnTimer: 0,
    shootCooldown: 0.8 + index * 0.15,
    strafeDir: index % 2 === 0 ? 1 : -1,
    walkCycle: index * 0.8,
    characterGroup: group,
    healthBar,
    shootFlash: 0,
  };
}
```

### Blocky Character Factory

```javascript
function createCharacter(palette) {
  const group = new THREE.Group();
  const skinMat = new THREE.MeshLambertMaterial({ color: palette.skin });
  const shirtMat = new THREE.MeshLambertMaterial({ color: palette.shirt });

  // Head
  const head = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.5, 0.5), skinMat);
  head.position.y = 1.55;
  group.add(head);

  // Body
  const body = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.75, 0.3), shirtMat);
  body.position.y = 1.125;
  group.add(body);

  // Arms — pivot at shoulder for animation
  const leftArm = new THREE.Group();
  leftArm.position.set(-0.375, 1.5, 0);
  const leftArmMesh = new THREE.Mesh(
    new THREE.BoxGeometry(0.25, 0.7, 0.25), skinMat
  );
  leftArmMesh.position.y = -0.35;
  leftArm.add(leftArmMesh);
  group.add(leftArm);

  // Store refs for animation
  group.userData.parts = { head, body, leftArm /* ... */ };
  return group;
}
```

### Walk Animation

```javascript
function animateCharacter(group, walkCycle, dt, shootKick = 0) {
  const parts = group.userData.parts;
  if (!parts) return;

  parts.leftArm.rotation.x = Math.sin(walkCycle) * 0.6;
  parts.rightArm.rotation.x = -Math.sin(walkCycle) * 0.6 - shootKick * 0.8;
  parts.leftLeg.rotation.x = -Math.sin(walkCycle) * 0.6;
  parts.rightLeg.rotation.x = Math.sin(walkCycle) * 0.6;
}
```

### Proper Cleanup

```javascript
function removeBotMesh(bot) {
  if (bot.characterGroup) {
    scene.remove(bot.characterGroup);
    bot.characterGroup.traverse(child => {
      if (child.geometry) child.geometry.dispose();
      if (child.material) {
        if (Array.isArray(child.material))
          child.material.forEach(m => m.dispose());
        else child.material.dispose();
      }
    });
    bot.characterGroup = null;
  }
}
```

## Best Practices

- **Separate data from visuals** — game logic operates on plain objects, rendering syncs
- **Pivot groups for limbs** — rotate the group, not the mesh, for natural swing
- **userData for cross-references** — `group.userData.parts`, `group.userData.botId`
- **Traverse + dispose** on removal — prevents GPU memory leaks
- **Stagger initial values** — `shootCooldown: 0.8 + index * 0.15` prevents sync

## Common Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Game state inside Three.js objects | Hard to serialize, debug | Keep state in plain objects |
| Not disposing on removal | GPU memory grows | `traverse()` + `dispose()` |
| Rotating mesh directly | Rotation origin wrong | Use pivot Group with offset mesh |
| All bots fire simultaneously | Unrealistic | Stagger cooldowns per bot |
| Not checking `alive` flag | Updating dead entities | Guard all updates with alive check |
