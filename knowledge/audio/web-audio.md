---
title: Web Audio API
since: "r1"
stable: "r150"
threejs_docs:
  - https://threejs.org/docs/#api/en/audio/AudioListener
  - https://threejs.org/docs/#api/en/audio/Audio
api_refs:
  - class: AudioListener
    url: https://threejs.org/docs/#api/en/audio/AudioListener
  - class: Audio
    url: https://threejs.org/docs/#api/en/audio/Audio
  - class: PositionalAudio
    url: https://threejs.org/docs/#api/en/audio/PositionalAudio
last_verified: "2026-03-12"
---

# Web Audio API

## Overview

Browser games use the Web Audio API for sound. The critical constraint: browsers block audio until the user interacts with the page (click, keypress). Use a singleton AudioContext pattern and create it on first user interaction.

## Key Concepts

### AudioContext Singleton
- Browsers require user gesture to start audio
- Create one AudioContext, reuse everywhere
- Lazy-init: create on first click/keypress, never before

### Three.js Audio Integration
Three.js wraps Web Audio with `AudioListener` (attached to camera) and `Audio`/`PositionalAudio` (attached to objects):
- **AudioListener** → camera (the "ears")
- **Audio** → non-positional (music, UI sounds)
- **PositionalAudio** → 3D spatialized (gunshots, footsteps)

### Programmatic Sound Generation
For simple effects (shoot, hit, explosion), generate sounds with OscillatorNode — no audio files needed:
- Create oscillator with frequency and type
- Connect through GainNode for volume envelope
- Short duration (50-200ms) for impact sounds

## Code Examples

### AudioContext Singleton

```javascript
let audioCtx = null;

function getAudioContext() {
  if (!audioCtx) {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  }
  if (audioCtx.state === 'suspended') {
    audioCtx.resume();
  }
  return audioCtx;
}

// Call on first user interaction
document.addEventListener('click', () => getAudioContext(), { once: true });
```

### Procedural Shoot Sound

```javascript
function playShootSound() {
  const ctx = getAudioContext();
  const osc = ctx.createOscillator();
  const gain = ctx.createGain();

  osc.type = 'sawtooth';
  osc.frequency.setValueAtTime(150, ctx.currentTime);
  osc.frequency.exponentialRampToValueAtTime(50, ctx.currentTime + 0.1);

  gain.gain.setValueAtTime(0.3, ctx.currentTime);
  gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 0.15);

  osc.connect(gain);
  gain.connect(ctx.destination);
  osc.start();
  osc.stop(ctx.currentTime + 0.15);
}
```

### Three.js Positional Audio

```javascript
const listener = new THREE.AudioListener();
camera.add(listener);

// Per-entity positional sound
const sound = new THREE.PositionalAudio(listener);
sound.setRefDistance(5);
sound.setVolume(0.5);
entityGroup.add(sound);
```

## Best Practices

- **Singleton AudioContext** — never create more than one
- **Lazy initialization** — create on user interaction, not on page load
- **Resume suspended context** — browsers suspend after inactivity
- **Short oscillator sounds** — cheaper than loading audio files for simple effects
- **`{ once: true }`** on init listener — don't re-create context on every click

## Common Mistakes

| Mistake | Problem | Solution |
|---------|---------|----------|
| Creating AudioContext on load | Blocked by browser autoplay policy | Create on user gesture |
| Multiple AudioContext instances | Resource waste, may hit browser limit | Singleton pattern |
| Not calling `resume()` | Silence after tab switch | Check and resume on each play |
| Loading large audio files | Slow initial load, bandwidth | Use procedural audio for simple effects |
| Not stopping oscillators | Memory leak, audio glitches | Always call `osc.stop()` with timeout |
