# Procedural Clouds — Claude Code Three.js Skill

A Claude Code skill for generating beautiful procedural clouds in Three.js, emphasizing **artistic quality** across all major cloud types with **WebGPU raymarching and WebGL fallbacks**.

Part of the **Teaching Three.js** skill series.

## What This Skill Does

When installed, this skill teaches Claude Code how to generate procedural cloudscapes including:

- **Volumetric raymarching** with Beer-Lambert absorption, Henyey-Greenstein phase functions, and light marching for self-shadowing
- **Mesh cluster clouds** — instanced soft-particle spheres with cloud-type-specific profiles
- **Billboard clouds** — camera-facing sprites with procedural noise textures for mobile/background use
- **All 10 cloud genera** — cumulus, stratus, cirrus, cumulonimbus, stratocumulus, altocumulus, altostratus, nimbostratus, cirrostratus, cirrocumulus
- **Physically-inspired lighting** — silver linings, subsurface scattering, two-lobe phase functions, Beer-powder bright edges
- **Time-of-day coloring** — dawn, golden hour, sunset, twilight, and night palettes
- **God ray post-processing** — radial blur for volumetric light shafts
- **WGSL 3D noise compute** — GPU-generated tiling Perlin + Worley noise textures
- **Animated drift** — wind-driven movement, formation, and dissipation

## Installation

### Claude Code (CLI)

```bash
claude install-skill path/to/procedural-clouds
```

Or copy the `procedural-clouds/` folder into your Claude Code skills directory.

### Manual Usage

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill — three rendering paths, lighting model, presets, scene assembly |
| `references/cloud-shaders.md` | Complete GLSL/WGSL shaders for volumetric, mesh, billboard, god rays |
| `references/cloud-types.md` | All 10 cloud genera with density parameters, lighting, and artistic direction |

## Quick Start Prompts

> "Create a sky with beautiful cumulus clouds and golden hour lighting"

> "Build volumetric raymarched clouds with silver linings and god rays"

> "Generate a dramatic cumulonimbus storm cloud with dark base and bright anvil top"

> "Create a sunset scene with stratocumulus clouds lit from below in gold and purple"

> "Build a peaceful cirrus sky with delicate ice crystal wisps at high altitude"

> "Create a layered sky with cirrus above altocumulus above scattered cumulus"

## Requirements

- **Three.js r170+** for WebGPU support and TSL node materials
- WebGPU browser (Chrome 121+, Edge 121+) for volumetric raymarching and compute noise
- WebGL2 fallback (mesh/billboard paths) works in all modern browsers

## Skill Architecture

```
procedural-clouds/
├── SKILL.md                          # Core skill (read first)
├── README.md                         # This file
└── references/
    ├── cloud-shaders.md              # GLSL, WGSL, and TSL shader code
    └── cloud-types.md                # 10 genera profiles with artistic notes
```

## Key Concepts

### Three Rendering Paths

| Path | Quality | Cost | Best For |
|------|---------|------|----------|
| Volumetric Raymarching | Highest | High | Desktop hero scenes, WebGPU |
| Mesh Cluster | Good | Medium | General use, WebGL2 |
| Billboard Sprites | Acceptable | Low | Mobile, distant backgrounds |

### Lighting Is Everything

Cloud beauty comes from light interaction, not shape alone. The skill implements:

- **Henyey-Greenstein phase** — two-lobe forward + back scattering
- **Beer-Lambert + powder** — bright thin edges, dark thick centers
- **Silver linings** — dramatic rim glow when sun is behind clouds
- **Self-shadowing** — inner light march creates natural dark bases
- **Time-of-day palettes** — from peach dawn to purple twilight

### Cloud Genera at a Glance

| Cloud | Altitude | Key Visual |
|-------|----------|------------|
| Cumulus | Low | Cotton ball, flat base |
| Stratus | Low | Grey blanket |
| Stratocumulus | Low | Lumpy blanket with gaps |
| Cumulonimbus | All | Towering storm anvil |
| Altocumulus | Mid | Mackerel sky pattern |
| Altostratus | Mid | Thin veil, watery sun |
| Nimbostratus | Mid | Dark rain sheet |
| Cirrus | High | Wispy ice streaks |
| Cirrostratus | High | Milky haze with halos |
| Cirrocumulus | High | Tiny fish-scale ripples |

## Series: Teaching Three.js

This skill is part of a series designed for learning and teaching Three.js with Claude Code. Each skill is independent and can be used standalone or combined with others.

**Companion skills**: [procedural-landscapes](../procedural-landscapes/) for terrain, [procedural-grass](../procedural-grass/) for ground cover.

## License

MIT — use freely in your projects.
