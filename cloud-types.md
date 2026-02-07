# Cloud Types Reference

Detailed profiles for all 10 major cloud genera with shader parameters, density field
tuning, lighting characteristics, and artistic direction for beautiful rendering.

## Altitude Layers

```
12km ┌─────────────────────────────────────┐
     │  HIGH: Cirrus, Cirrostratus,        │  Ice crystals
     │        Cirrocumulus                  │  Thin, translucent
 6km ├─────────────────────────────────────┤
     │  MID: Altocumulus, Altostratus,     │  Mixed ice/water
     │       Nimbostratus                   │  Moderate density
 2km ├─────────────────────────────────────┤
     │  LOW: Cumulus, Stratus,             │  Water droplets
     │       Stratocumulus                  │  Dense, defined edges
     │       Cumulonimbus (extends to high)│
     └─────────────────────────────────────┘
```

## Low Clouds

### Cumulus

**Character**: The classic "cotton ball" cloud. Flat grey base, bright cauliflower
billows on top. Fair weather, friendly, photogenic.

```javascript
const CUMULUS = {
  altitude: { base: 1000, top: 2500 },
  density: {
    coverage: 0.35,
    shapeOctaves: 3,
    detailOctaves: 5,
    detailStrength: 0.35,
    worleyBlend: 0.5,        // High — cellular billows are key
    baseRoundness: 0.3,      // Flat base, round top
    edgeSharpness: 0.6,      // Defined, puffy edges
  },
  lighting: {
    absorptionCoeff: 0.045,
    phaseForward: 0.6,       // Strong forward scattering
    phaseBack: -0.2,
    silverLining: 0.4,       // Very photogenic silver edges
    baseDarkening: 0.35,     // Grey shadow at base
    sssStrength: 0.3,
  },
  wind: { speed: 10, turbulence: 0.2 },
  artistic: `
    The "hero" cloud. Top surfaces are brilliant white in direct sun.
    Shadows on the base create a satisfying grey gradient. Edges should
    be distinctly puffy — not smooth, not ragged. Think "friendly and solid."
    At sunset, tops turn gold/pink while bases go deep purple.
  `,
  meshProfile: {
    particlesPerCloud: 30,
    scaleRange: [4, 12],
    verticalBias: 0.7,      // More volume above center
    flatBase: true,
  },
};
```

### Stratus

**Character**: Flat, featureless grey blanket. Uniform, can cover entire sky.
Moody, contemplative, diffuses all light.

```javascript
const STRATUS = {
  altitude: { base: 300, top: 1500 },
  density: {
    coverage: 0.85,
    shapeOctaves: 2,          // Low — minimal shape variation
    detailOctaves: 3,
    detailStrength: 0.15,     // Very smooth
    worleyBlend: 0.1,         // Barely any cellular structure
    baseRoundness: 0.0,       // Flat everywhere
    edgeSharpness: 0.2,       // Soft, blurred edges
  },
  lighting: {
    absorptionCoeff: 0.06,    // Denser
    phaseForward: 0.3,        // Less directional
    phaseBack: -0.1,
    silverLining: 0.1,        // Minimal — no dramatic edges
    baseDarkening: 0.15,      // Uniformly lit (diffused)
    sssStrength: 0.15,
  },
  wind: { speed: 8, turbulence: 0.05 },
  artistic: `
    Think "overcast day." No drama, no highlights — just uniform soft light.
    Color ranges from white-grey to dark grey depending on thickness.
    The beauty is in the subtlety: slight variations in thickness create
    barely-perceptible lighter and darker patches. At sunset, bottom
    can catch warm light while maintaining grey above.
  `,
  meshProfile: {
    particlesPerCloud: 40,
    scaleRange: [10, 20],
    verticalBias: 0.0,       // No vertical preference
    flatBase: false,
  },
};
```

### Stratocumulus

**Character**: Lumpy blanket with gaps showing blue sky. Most common cloud type
worldwide. Has personality — not as flat as stratus, not as puffy as cumulus.

```javascript
const STRATOCUMULUS = {
  altitude: { base: 600, top: 2000 },
  density: {
    coverage: 0.65,
    shapeOctaves: 3,
    detailOctaves: 4,
    detailStrength: 0.3,
    worleyBlend: 0.35,        // Moderate cellular pattern
    baseRoundness: 0.15,
    edgeSharpness: 0.4,
    tilePattern: true,        // Repeating lumpy pattern
  },
  lighting: {
    absorptionCoeff: 0.05,
    phaseForward: 0.4,
    phaseBack: -0.2,
    silverLining: 0.25,
    baseDarkening: 0.25,
    sssStrength: 0.25,
  },
  wind: { speed: 12, turbulence: 0.15 },
  artistic: `
    The "blanket with holes" cloud. Gaps between lumps let blue sky through.
    Each lump has a subtle bright top and darker underside. This is THE
    sunset cloud — when lit from below, each lump becomes individually
    painted with gold, pink, and purple. Stunning in golden hour.
  `,
  meshProfile: {
    particlesPerCloud: 20,
    scaleRange: [6, 14],
    verticalBias: 0.3,
    flatBase: true,
  },
};
```

### Cumulonimbus

**Character**: Towering monster cloud. Dark, threatening base; brilliant white
anvil top spreading at tropopause. The most dramatic cloud in nature.

```javascript
const CUMULONIMBUS = {
  altitude: { base: 500, top: 12000 },  // Extends through all layers
  density: {
    coverage: 0.3,            // Individual cells, not widespread
    shapeOctaves: 4,
    detailOctaves: 6,
    detailStrength: 0.4,
    worleyBlend: 0.6,         // Strong billowing structure
    baseRoundness: 0.5,       // Very flat dark base
    edgeSharpness: 0.8,       // Sharp, hard-edged billows
    anvilSpread: true,        // Spreads horizontally at top
  },
  lighting: {
    absorptionCoeff: 0.1,     // Very dense = very dark base
    phaseForward: 0.7,
    phaseBack: -0.3,
    silverLining: 0.6,        // Dramatic backlit edges
    baseDarkening: 0.6,       // Ominously dark base
    sssStrength: 0.2,
    internalLightning: true,  // Optional: illumination flashes
  },
  wind: { speed: 5, turbulence: 0.3 },
  artistic: `
    Maximum drama. The base should be dark grey to near-black. The tower
    is brilliant white cauliflower that catches sunlight on every billow.
    The anvil top is flat and spreads like a mushroom cap, with wispy
    edges. If backlit by the sun, the entire outline glows with an
    intense silver/gold rim. Lightning can illuminate the interior with
    brief warm flashes. This is the cloud you want for epic scenes.
  `,
  meshProfile: {
    particlesPerCloud: 50,
    scaleRange: [5, 25],
    verticalBias: 0.9,        // Strongly vertical
    flatBase: true,
  },
};
```

## Mid-Level Clouds

### Altocumulus

**Character**: "Mackerel sky" — repeating pattern of small cloudlets in rows.
Beautiful, ordered, photogenic. Often predicts weather change.

```javascript
const ALTOCUMULUS = {
  altitude: { base: 2500, top: 5000 },
  density: {
    coverage: 0.5,
    shapeOctaves: 2,
    detailOctaves: 3,
    detailStrength: 0.25,
    worleyBlend: 0.7,         // Key: cellular/tiled pattern
    baseRoundness: 0.1,
    edgeSharpness: 0.5,
    tileScale: 0.8,           // Smaller, repeating units
  },
  lighting: {
    absorptionCoeff: 0.035,
    phaseForward: 0.4,
    phaseBack: -0.2,
    silverLining: 0.3,
    baseDarkening: 0.2,
    sssStrength: 0.35,
  },
  artistic: `
    The pattern is the beauty. Regular, repeating rows of small cloudlets
    with blue sky between them. Think fish scales or a waffle pattern.
    Each individual element is small and well-defined. At sunset, the
    pattern creates a tapestry effect — each cloudlet individually colored.
    Use high Worley blend to achieve the cellular repetition.
  `,
};
```

### Altostratus

**Character**: Thin translucent veil. Sun visible as a bright diffuse spot
behind it ("watery sun"). Atmospheric, melancholic.

```javascript
const ALTOSTRATUS = {
  altitude: { base: 2000, top: 5000 },
  density: {
    coverage: 0.8,
    shapeOctaves: 2,
    detailOctaves: 2,
    detailStrength: 0.1,
    worleyBlend: 0.05,
    edgeSharpness: 0.15,
  },
  lighting: {
    absorptionCoeff: 0.02,    // Thin enough to see sun through
    phaseForward: 0.5,
    silverLining: 0.1,
    baseDarkening: 0.1,
    sssStrength: 0.5,         // High — light passes through
    sunDiscVisible: true,      // Bright spot where sun is
  },
  artistic: `
    A translucent curtain across the sky. The sun is visible as a bright
    but diffuse disc behind it. Colors shift subtly across the veil —
    slightly thicker patches are darker, thinner ones brighter. Can be
    beautiful in a quiet, understated way. Use high SSS and low absorption.
  `,
};
```

### Nimbostratus

**Character**: Thick, dark, rain-bearing blanket. Low visibility, no features.
The "bad weather" cloud.

```javascript
const NIMBOSTRATUS = {
  altitude: { base: 1000, top: 4000 },
  density: {
    coverage: 0.95,
    shapeOctaves: 2,
    detailOctaves: 3,
    detailStrength: 0.15,
    worleyBlend: 0.1,
    edgeSharpness: 0.1,
  },
  lighting: {
    absorptionCoeff: 0.09,    // Very dense
    phaseForward: 0.2,
    silverLining: 0.05,
    baseDarkening: 0.5,
    sssStrength: 0.1,
  },
  artistic: `
    Dark, oppressive, featureless. The sky becomes a uniform dark grey
    ceiling. No silver linings, no definition — just weight and mood.
    Beauty here comes from the atmosphere it creates: rain streaks below,
    faint lighter patches suggesting hidden depth. Pair with fog.
  `,
};
```

## High Clouds

### Cirrus

**Character**: Delicate wispy streaks of ice crystals. Hooks, mares' tails,
feathery strands. Ethereal, high-altitude beauty.

```javascript
const CIRRUS = {
  altitude: { base: 7000, top: 12000 },
  density: {
    coverage: 0.25,
    shapeOctaves: 2,
    detailOctaves: 4,
    detailStrength: 0.5,      // High detail erodes to wisps
    worleyBlend: 0.15,
    edgeSharpness: 0.7,       // Defined streaks
    directional: true,        // Aligned to wind
    curlStrength: 0.4,        // Hook/curl at ends
  },
  lighting: {
    absorptionCoeff: 0.008,   // Very thin
    phaseForward: 0.8,        // Ice crystals = strong forward scatter
    silverLining: 0.5,
    baseDarkening: 0.05,
    sssStrength: 0.7,         // Nearly transparent
  },
  artistic: `
    Pure elegance. Thin, curved strokes painted across a deep blue sky.
    Almost like calligraphy. Each strand should have a hook or curl at
    its end (ice crystals falling and curving in wind). Catch sunlight
    brilliantly — at sunset they're the first to turn gold/pink because
    of their altitude. Use domain warping to create the curving streaks.
  `,
};
```

### Cirrostratus

**Character**: Thin milky veil at high altitude. Creates halos around sun/moon.
Subtle, ethereal.

```javascript
const CIRROSTRATUS = {
  altitude: { base: 6000, top: 11000 },
  density: {
    coverage: 0.6,
    shapeOctaves: 1,
    detailOctaves: 2,
    detailStrength: 0.1,
    worleyBlend: 0.0,
    edgeSharpness: 0.1,
  },
  lighting: {
    absorptionCoeff: 0.005,   // Nearly invisible
    phaseForward: 0.85,       // Ice crystal halo
    silverLining: 0.2,
    baseDarkening: 0.0,
    sssStrength: 0.8,
    haloEffect: true,         // 22° halo around sun
  },
  artistic: `
    Barely there — a thin milky wash that slightly whitens the sky. The
    hallmark is the halo effect: a ring of light around the sun caused
    by ice crystal refraction. Implement as a subtle brightening ring
    at ~22° from the sun position. Beautiful in its restraint.
  `,
};
```

### Cirrocumulus

**Character**: Tiny rippled cloudlets at high altitude. Delicate fish-scale or
grain-of-rice pattern. Rare and beautiful.

```javascript
const CIRROCUMULUS = {
  altitude: { base: 6000, top: 10000 },
  density: {
    coverage: 0.35,
    shapeOctaves: 2,
    detailOctaves: 3,
    detailStrength: 0.3,
    worleyBlend: 0.8,         // Very cellular — small tiles
    edgeSharpness: 0.6,
    tileScale: 1.5,           // Very small repeating elements
  },
  lighting: {
    absorptionCoeff: 0.01,
    phaseForward: 0.7,
    silverLining: 0.3,
    baseDarkening: 0.1,
    sssStrength: 0.6,
  },
  artistic: `
    Like altocumulus but at high altitude and much smaller/thinner. The
    pattern is exquisitely delicate — tiny ripples or grains. Each element
    is so small it's almost pointillistic. At sunset, creates a fine-
    textured tapestry of color. Use very high-frequency Worley noise.
  `,
};
```

## Combining Cloud Types

Real skies often feature multiple cloud types at different altitudes simultaneously.

```javascript
function createRealisticSky(presets) {
  // Layer: high cirrus over mid altocumulus over low cumulus
  return [
    { type: CIRRUS,      weight: 0.3, altitudeOffset: 0 },
    { type: ALTOCUMULUS,  weight: 0.5, altitudeOffset: 0 },
    { type: CUMULUS,      weight: 0.8, altitudeOffset: 0 },
  ];
}

// In the density function, sample all active layers:
float multiLayerDensity(vec3 p) {
  float density = 0.0;
  for (each layer) {
    if (p.y >= layer.base && p.y <= layer.top) {
      density += cloudDensityForType(p, layer.type) * layer.weight;
    }
  }
  return density;
}
```

## Artistic Color Palettes by Time of Day

| Time | Cloud Lit Side | Cloud Shadow | Sky |
|------|---------------|-------------|-----|
| Dawn | `#FFB088` peach | `#6B4A6B` mauve | `#2A4A7A` deep blue |
| Morning | `#FFF0D0` warm white | `#9AAABB` blue-grey | `#6699CC` sky blue |
| Midday | `#FFFFFF` pure white | `#AABBCC` cool grey | `#4488BB` bright blue |
| Afternoon | `#FFF4D0` cream | `#99AABB` steel | `#5599CC` mid blue |
| Golden Hour | `#FFAA44` gold | `#774433` brown | `#3366AA` deepening blue |
| Sunset | `#FF6633` orange | `#442255` purple | `#1A3366` dark blue |
| Twilight | `#CC4466` rose | `#332244` violet | `#0A1A33` navy |
| Night | `#222233` dark blue | `#111122` near-black | `#050510` black |
