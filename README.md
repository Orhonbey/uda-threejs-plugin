# uda-threejs-plugin

Three.js game development knowledge, workflows, and agents for [UDA](https://github.com/Orhonbey/Uda).

## Install

```bash
uda plugin add https://github.com/Orhonbey/uda-threejs-plugin.git
```

## What's Included

### Knowledge Base (10 files)

| Category | Topics |
|----------|--------|
| **Core** | Scene/Camera/Renderer setup, Game loop & delta time, Project structure |
| **Rendering** | Materials & lighting, InstancedMesh for voxel worlds, Sprites & visual effects |
| **Patterns** | FPS controller (PointerLock + WASD), Collision detection & raycasting, Entity/bot management |
| **Audio** | Web Audio API, AudioContext singleton |

### Workflows (4)

| Workflow | Description |
|----------|-------------|
| `create-scene` | Scaffold a new Three.js scene with camera, renderer, and lighting |
| `create-fps-controller` | Set up first-person movement with PointerLock and WASD |
| `create-entity` | Create a new game entity with blocky character model |
| `setup-voxel-world` | Build a voxel world from a 2D map array |

### Agents (3)

| Agent | Role |
|-------|------|
| `threejs-architect` | System design and architecture decisions |
| `threejs-debugger` | Browser console errors, WebGL issues, performance |
| `threejs-reviewer` | Code review for Three.js best practices |

## Requirements

- UDA CLI v0.6.0+
- Three.js r150+

## License

MIT
