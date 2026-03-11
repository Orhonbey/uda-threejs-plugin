---
title: Three.js Project Structure
since: "r1"
stable: "r150"
threejs_docs:
  - https://threejs.org/docs/#manual/en/introduction/Installation
api_refs: []
last_verified: "2026-03-12"
---

# Three.js Project Structure

## Overview

Three.js browser games can run with zero build tools — just ES modules and a local HTTP server. This simplicity is a strength: no webpack, no bundler config, no compile step. For larger projects, Vite or similar tools can be added later.

## Key Concepts

### Zero-Build Setup
Three.js supports direct ES module imports via import maps or CDN. A game can ship as plain `.js` files served by any HTTP server.

### Module Organization
Split game code by responsibility, not by file type:
- **game.js** — main entry, game loop, state management
- **world.js** — level building, collision geometry
- **characters.js** — entity models, animation, health bars
- **weapons.js** — weapon view, recoil, muzzle flash
- **maps.js** — map data, grid layouts, spawn points

### Import Maps
HTML `<script type="importmap">` resolves bare specifiers like `'three'` to a CDN or local path without a bundler.

## Code Examples

### Minimal HTML Entry

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Game</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <canvas id="game"></canvas>
  <div id="hud"><!-- HUD elements --></div>

  <script type="importmap">
  {
    "imports": {
      "three": "https://cdn.jsdelivr.net/npm/three@0.162.0/build/three.module.js"
    }
  }
  </script>
  <script type="module" src="game.js"></script>
</body>
</html>
```

### Recommended Directory Layout

```
project/
├── index.html          # Entry point with import map
├── styles.css          # HUD and overlay styles
├── game.js             # Main loop, state, input
├── world.js            # Level geometry, collision
├── characters.js       # Entity models, animation
├── weapons.js          # First-person weapon view
├── maps.js             # Map data definitions
├── package.json        # Optional — for dev server
└── node_modules/       # Optional — if using npm
```

### Running Locally

```bash
# No npm needed — any HTTP server works
python3 -m http.server 4173

# Or with npm
npx serve .

# Or with package.json script
npm run serve
```

### Module Export Pattern

```javascript
// world.js — export pure functions, no side effects at module level
import * as THREE from 'three';
import { MAPS, WALL, getCell } from './maps.js';

export function buildWorld(scene, mapKey) { /* ... */ }
export function isWallAt(mapData, x, z) { /* ... */ }
export function moveWithCollision(entity, dx, dz, mapData) { /* ... */ }
```

## Best Practices

- **One module per system** — world, characters, weapons, maps separate
- **Export functions, not state** — game state lives in game.js, modules are stateless
- **Import map for Three.js** — avoids bundler for simple projects
- **Pinned version** — `three@0.162.0` not `three@latest` for reproducibility
- **Local dev server** — ES modules require HTTP, not `file://`

## Common Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Opening index.html as file:// | ES modules blocked by CORS | Use HTTP server |
| Single 2000-line file | Unmaintainable | Split by system/responsibility |
| Global variables between modules | Hidden coupling | Pass dependencies as function args |
| Using `require()` | Not supported in browser ES modules | Use `import`/`export` |
| Unpinned CDN version | Breaking changes on update | Pin exact Three.js version |

## Scaling Up

When the project grows beyond 5-6 modules:
- Add **Vite** for hot reload and bundling: `npm create vite@latest`
- Move Three.js to npm dependency: `npm install three`
- Vite auto-handles import maps, HMR, and production builds
- File structure stays the same — just add `vite.config.js`
