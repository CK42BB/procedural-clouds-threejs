# Cloud Shaders Reference

Complete shader implementations for all three cloud rendering paths: volumetric
raymarching, mesh cluster, and billboard — plus god ray post-process and WGSL
compute noise.

## Shared: Fullscreen Vertex Shader

Used by volumetric and god ray passes.

```glsl
// fullscreen.vert
varying vec2 vUv;
void main() {
  vUv = uv;
  gl_Position = vec4(position.xy, 0.0, 1.0);
}
```

## Volumetric Raymarching Fragment Shader

Full-quality cloud rendering in a single fragment shader pass.

```glsl
// volumetric_clouds.frag
precision highp float;

uniform vec3  cameraPos;
uniform mat4  invProjection;
uniform mat4  invView;
uniform vec3  sunDir;
uniform vec3  sunColor;
uniform vec3  ambientSky;
uniform float time;
uniform float cloudBase;
uniform float cloudTop;
uniform float coverage;
uniform float detailStrength;
uniform vec2  windDirection;
uniform float windSpeed;
uniform float absorptionCoeff;

varying vec2 vUv;

#define MAX_STEPS 80
#define LIGHT_STEPS 6
#define PI 3.14159265

// ─── Noise ──────────────────────────────────────────
vec3 hash3(vec3 p) {
  p = vec3(dot(p, vec3(127.1,311.7,74.7)),
           dot(p, vec3(269.5,183.3,246.1)),
           dot(p, vec3(113.5,271.9,124.6)));
  return fract(sin(p) * 43758.5453);
}

float noise3D(vec3 p) {
  vec3 i = floor(p);
  vec3 f = fract(p);
  f = f * f * (3.0 - 2.0 * f);

  return mix(mix(mix(dot(hash3(i+vec3(0,0,0)), f-vec3(0,0,0)),
                     dot(hash3(i+vec3(1,0,0)), f-vec3(1,0,0)), f.x),
                 mix(dot(hash3(i+vec3(0,1,0)), f-vec3(0,1,0)),
                     dot(hash3(i+vec3(1,1,0)), f-vec3(1,1,0)), f.x), f.y),
             mix(mix(dot(hash3(i+vec3(0,0,1)), f-vec3(0,0,1)),
                     dot(hash3(i+vec3(1,0,1)), f-vec3(1,0,1)), f.x),
                 mix(dot(hash3(i+vec3(0,1,1)), f-vec3(0,1,1)),
                     dot(hash3(i+vec3(1,1,1)), f-vec3(1,1,1)), f.x), f.y), f.z);
}

float fbm(vec3 p, int octaves) {
  float sum = 0.0, amp = 1.0, freq = 1.0, maxA = 0.0;
  for (int i = 0; i < 6; i++) {
    if (i >= octaves) break;
    sum += noise3D(p * freq) * amp;
    maxA += amp;
    amp *= 0.5;
    freq *= 2.0;
  }
  return sum / maxA;
}

// Worley noise for cumulus-like cellular shapes
float worley(vec3 p) {
  vec3 i = floor(p);
  vec3 f = fract(p);
  float minDist = 1.0;

  for (int x = -1; x <= 1; x++)
  for (int y = -1; y <= 1; y++)
  for (int z = -1; z <= 1; z++) {
    vec3 neighbor = vec3(float(x), float(y), float(z));
    vec3 point = hash3(i + neighbor);
    vec3 diff = neighbor + point - f;
    minDist = min(minDist, dot(diff, diff));
  }
  return sqrt(minDist);
}

// ─── Remap utility ──────────────────────────────────
float remap(float v, float lo, float hi, float newLo, float newHi) {
  return newLo + (clamp(v, lo, hi) - lo) / (hi - lo) * (newHi - newLo);
}

// ─── Cloud density ──────────────────────────────────
float cloudDensity(vec3 p) {
  vec3 wind = vec3(windDirection.x, 0.0, windDirection.y) * windSpeed * time * 0.001;

  // Altitude envelope
  float altNorm = (p.y - cloudBase) / (cloudTop - cloudBase);
  float altEnv = smoothstep(0.0, 0.15, altNorm) * smoothstep(1.0, 0.7, altNorm);

  // Large-scale shape
  vec3 shapePos = (p + wind) * 0.0003;
  float shape = fbm(shapePos, 3);
  // Worley for cumulus-like billows
  float cellShape = 1.0 - worley(shapePos * 4.0);
  shape = shape * 0.6 + cellShape * 0.4;

  // Coverage threshold
  shape = remap(shape, 1.0 - coverage, 1.0, 0.0, 1.0);

  // Detail erosion
  vec3 detailPos = (p + wind * 2.0) * 0.003;
  float detail = fbm(detailPos, 5) * detailStrength;

  float density = max(shape - detail, 0.0) * altEnv;

  // Round bottom for cumulus (less density below midpoint)
  float bottomRound = smoothstep(0.0, 0.3, altNorm);
  density *= bottomRound;

  return density;
}

// ─── Phase function ─────────────────────────────────
float henyeyGreenstein(float cosTheta, float g) {
  float g2 = g * g;
  return (1.0 - g2) / (4.0 * PI * pow(1.0 + g2 - 2.0 * g * cosTheta, 1.5));
}

float cloudPhase(float cosTheta) {
  return henyeyGreenstein(cosTheta, 0.6) * 0.7
       + henyeyGreenstein(cosTheta, -0.3) * 0.3;
}

// ─── Light march ────────────────────────────────────
float lightMarch(vec3 p) {
  float stepL = (cloudTop - p.y) / float(LIGHT_STEPS);
  vec3 lightStep = sunDir * stepL;
  float accum = 0.0;

  for (int i = 0; i < LIGHT_STEPS; i++) {
    p += lightStep;
    accum += max(cloudDensity(p), 0.0) * stepL * 0.001;
  }

  // Beer-powder for bright thin edges
  float beer = exp(-accum * absorptionCoeff);
  float powder = 1.0 - exp(-accum * absorptionCoeff * 2.0);
  return mix(beer, beer * powder, 0.5);
}

// ─── Ray-slab intersection ──────────────────────────
vec2 intersectSlab(vec3 ro, vec3 rd, float yMin, float yMax) {
  float tMin = (yMin - ro.y) / rd.y;
  float tMax = (yMax - ro.y) / rd.y;
  if (tMin > tMax) { float tmp = tMin; tMin = tMax; tMax = tmp; }
  return vec2(max(tMin, 0.0), tMax);
}

// ─── Main ───────────────────────────────────────────
void main() {
  // Reconstruct ray direction from screen UV
  vec4 clipPos = vec4(vUv * 2.0 - 1.0, 1.0, 1.0);
  vec4 viewPos = invProjection * clipPos;
  viewPos.xyz /= viewPos.w;
  vec3 rd = normalize((invView * vec4(viewPos.xyz, 0.0)).xyz);
  vec3 ro = cameraPos;

  // Intersect cloud layer
  vec2 slabT = intersectSlab(ro, rd, cloudBase, cloudTop);
  if (slabT.x >= slabT.y || slabT.y < 0.0) {
    gl_FragColor = vec4(0.0);
    return;
  }

  float cosTheta = dot(rd, sunDir);
  float phase = cloudPhase(cosTheta);

  // Jitter start to reduce banding
  float jitter = fract(sin(dot(vUv, vec2(12.9898, 78.233))) * 43758.5453);
  float stepSize = (slabT.y - slabT.x) / float(MAX_STEPS);
  float t = slabT.x + jitter * stepSize;

  vec4 result = vec4(0.0);

  for (int i = 0; i < MAX_STEPS; i++) {
    if (result.a > 0.98 || t > slabT.y) break;

    vec3 p = ro + rd * t;
    float density = cloudDensity(p);

    if (density > 0.001) {
      float lightEnergy = lightMarch(p);

      // Cloud color
      vec3 cloudCol = sunColor * lightEnergy * phase + ambientSky * 0.2;

      // Silver lining at edges
      float edgeDensity = cloudDensity(p + sunDir * 50.0);
      float silver = pow(max(1.0 - edgeDensity, 0.0), 2.0);
      silver *= pow(max(-cosTheta, 0.0), 2.0);
      cloudCol += sunColor * silver * 0.4;

      // Self-shadow darkening toward base
      float altNorm = (p.y - cloudBase) / (cloudTop - cloudBase);
      cloudCol *= mix(0.4, 1.0, altNorm);

      float alpha = 1.0 - exp(-density * stepSize * absorptionCoeff * 80.0);
      result.rgb += cloudCol * alpha * (1.0 - result.a);
      result.a += alpha * (1.0 - result.a);
    }

    t += stepSize;
  }

  gl_FragColor = result;
}
```

## Mesh Cloud Vertex Shader

Per-instance transform with soft fade at edges.

```glsl
// mesh_cloud.vert
varying vec3 vWorldPos;
varying vec3 vNormal;
varying float vEdgeFade;

void main() {
  vNormal = normalize(normalMatrix * normal);
  vec4 worldPos = modelMatrix * instanceMatrix * vec4(position, 1.0);
  vWorldPos = worldPos.xyz;

  // Edge fade: distance from center of sphere → soft edges
  vEdgeFade = 1.0 - length(position) * 0.5;
  vEdgeFade = smoothstep(0.0, 0.5, vEdgeFade);

  gl_Position = projectionMatrix * viewMatrix * worldPos;
}
```

## Mesh Cloud Fragment Shader

Soft-particle shading with scattering approximation.

```glsl
// mesh_cloud.frag
precision highp float;

uniform vec3  sunDir;
uniform vec3  sunColor;
uniform vec3  ambientColor;
uniform vec3  baseColor;
uniform float opacity;
uniform float time;

varying vec3  vWorldPos;
varying vec3  vNormal;
varying float vEdgeFade;

void main() {
  vec3 N = normalize(vNormal);
  vec3 L = normalize(sunDir);

  // Wrap diffuse for softness
  float diff = dot(N, L) * 0.5 + 0.5;

  // Subsurface approximation — light shining through from behind
  float sss = pow(max(dot(-N, L), 0.0), 2.0) * 0.4;

  // Top-lit bias: clouds are brighter on top
  float topBias = smoothstep(-0.2, 0.5, N.y) * 0.3;

  vec3 color = baseColor * (sunColor * diff + ambientColor * 0.4 + sunColor * sss);
  color += sunColor * topBias;

  // Darker base
  float baseDarken = smoothstep(0.3, -0.3, N.y) * 0.3;
  color *= 1.0 - baseDarken;

  // Soft edge fade
  float alpha = opacity * vEdgeFade;

  gl_FragColor = vec4(color, alpha);
}
```

## God Ray Post-Process Fragment Shader

Radial blur from sun screen position for volumetric light shafts.

```glsl
// god_rays.frag
precision highp float;

uniform sampler2D tInput;
uniform vec2  sunScreenPos;
uniform float exposure;
uniform float decay;
uniform float density;
uniform float weight;
uniform int   samples;

varying vec2 vUv;

void main() {
  vec2 texCoord = vUv;
  vec2 deltaUV = (texCoord - sunScreenPos) * density / float(samples);

  vec3 result = vec3(0.0);
  float illuminationDecay = 1.0;

  for (int i = 0; i < 100; i++) { // Max 100, controlled by `samples` uniform
    if (i >= samples) break;
    texCoord -= deltaUV;
    vec3 samp = texture2D(tInput, texCoord).rgb;
    samp *= illuminationDecay * weight;
    result += samp;
    illuminationDecay *= decay;
  }

  gl_FragColor = vec4(texture2D(tInput, vUv).rgb + result * exposure, 1.0);
}
```

## WGSL 3D Noise Compute Texture

Generate a tiling 3D noise texture on the GPU for use in volumetric cloud sampling.
Avoids expensive per-fragment noise computation.

```wgsl
// noise3d_compute.wgsl
@group(0) @binding(0) var output: texture_storage_3d<rgba8unorm, write>;

struct Params {
  resolution: u32,
  frequency: f32,
  octaves: u32,
  lacunarity: f32,
  gain: f32,
  seed: f32,
}
@group(0) @binding(1) var<uniform> params: Params;

fn hash3(p: vec3<f32>) -> vec3<f32> {
  var q = vec3<f32>(
    dot(p, vec3<f32>(127.1, 311.7, 74.7)),
    dot(p, vec3<f32>(269.5, 183.3, 246.1)),
    dot(p, vec3<f32>(113.5, 271.9, 124.6))
  );
  return fract(sin(q) * 43758.5453);
}

fn gradientNoise3D(p: vec3<f32>) -> f32 {
  let i = floor(p);
  let f = fract(p);
  let u = f * f * (3.0 - 2.0 * f);

  // 8-corner gradient interpolation
  let n000 = dot(hash3(i) * 2.0 - 1.0, f);
  let n100 = dot(hash3(i + vec3<f32>(1,0,0)) * 2.0 - 1.0, f - vec3<f32>(1,0,0));
  let n010 = dot(hash3(i + vec3<f32>(0,1,0)) * 2.0 - 1.0, f - vec3<f32>(0,1,0));
  let n110 = dot(hash3(i + vec3<f32>(1,1,0)) * 2.0 - 1.0, f - vec3<f32>(1,1,0));
  let n001 = dot(hash3(i + vec3<f32>(0,0,1)) * 2.0 - 1.0, f - vec3<f32>(0,0,1));
  let n101 = dot(hash3(i + vec3<f32>(1,0,1)) * 2.0 - 1.0, f - vec3<f32>(1,0,1));
  let n011 = dot(hash3(i + vec3<f32>(0,1,1)) * 2.0 - 1.0, f - vec3<f32>(0,1,1));
  let n111 = dot(hash3(i + vec3<f32>(1,1,1)) * 2.0 - 1.0, f - vec3<f32>(1,1,1));

  return mix(
    mix(mix(n000, n100, u.x), mix(n010, n110, u.x), u.y),
    mix(mix(n001, n101, u.x), mix(n011, n111, u.x), u.y),
    u.z
  );
}

fn worley3D(p: vec3<f32>) -> f32 {
  let i = floor(p);
  let f = fract(p);
  var minDist = 1.0;

  for (var x = -1; x <= 1; x++) {
    for (var y = -1; y <= 1; y++) {
      for (var z = -1; z <= 1; z++) {
        let neighbor = vec3<f32>(f32(x), f32(y), f32(z));
        let point = hash3(i + neighbor);
        let diff = neighbor + point - f;
        minDist = min(minDist, dot(diff, diff));
      }
    }
  }
  return sqrt(minDist);
}

fn fbm3D(p: vec3<f32>, octaves: u32) -> f32 {
  var sum = 0.0;
  var amp = 1.0;
  var freq = 1.0;
  var maxA = 0.0;

  for (var i = 0u; i < octaves; i++) {
    sum += gradientNoise3D(p * freq) * amp;
    maxA += amp;
    amp *= params.gain;
    freq *= params.lacunarity;
  }
  return sum / maxA;
}

@compute @workgroup_size(8, 8, 8)
fn main(@builtin(global_invocation_id) gid: vec3<u32>) {
  let res = params.resolution;
  if (gid.x >= res || gid.y >= res || gid.z >= res) { return; }

  let p = vec3<f32>(gid) / f32(res) * params.frequency + params.seed;

  // Channel R: Perlin FBM (large-scale shape)
  let perlin = fbm3D(p, params.octaves) * 0.5 + 0.5;

  // Channel G: Worley (cellular billows)
  let cell = 1.0 - worley3D(p * 4.0);

  // Channel B: High-frequency detail
  let detail = fbm3D(p * 4.0, min(params.octaves, 4u)) * 0.5 + 0.5;

  // Channel A: Worley detail (erosion)
  let erosion = worley3D(p * 8.0);

  textureStore(output, gid, vec4<f32>(perlin, cell, detail, erosion));
}
```

### Using the 3D Noise Texture in Volumetric Shader

Replace per-fragment noise evaluation with texture lookups for massive speedup:

```glsl
uniform sampler3D noiseTexture;

float cloudDensityFast(vec3 p) {
  vec3 wind = vec3(windDirection.x, 0.0, windDirection.y) * windSpeed * time * 0.001;
  vec3 samplePos = (p + wind) * 0.0003;

  vec4 noise = texture(noiseTexture, samplePos);
  float shape = noise.r * 0.6 + noise.g * 0.4;  // Perlin + Worley blend
  shape = remap(shape, 1.0 - coverage, 1.0, 0.0, 1.0);

  float detail = noise.b * detailStrength;
  float altNorm = (p.y - cloudBase) / (cloudTop - cloudBase);
  float altEnv = smoothstep(0.0, 0.15, altNorm) * smoothstep(1.0, 0.7, altNorm);

  return max(shape - detail, 0.0) * altEnv;
}
```

## TSL Cloud Material (WebGPU)

Node-based cloud material for the mesh cluster path using Three.js Shading Language.

```javascript
import { attribute, cameraPosition, color, dot, float as tslFloat,
         max as tslMax, mix, normalize as tslNorm, positionWorld,
         smoothstep, uniform, vec3, MeshStandardNodeMaterial } from 'three/tsl';

function createCloudNodeMaterial() {
  const material = new MeshStandardNodeMaterial();
  material.transparent = true;
  material.depthWrite = false;
  material.side = THREE.DoubleSide;

  const sunDir = uniform(vec3(0.3, 0.8, 0.5));
  const N = tslNorm(attribute('normal'));
  const L = tslNorm(sunDir);

  // Wrap diffuse
  const diff = dot(N, L).mul(0.5).add(0.5);

  // SSS approximation
  const sss = tslMax(dot(N.negate(), L), tslFloat(0)).pow(2).mul(0.4);

  // Top bias
  const topBias = smoothstep(tslFloat(-0.2), tslFloat(0.5), N.y).mul(0.3);

  const cloudCol = color(0xffffff).mul(diff.add(sss).add(topBias));

  material.colorNode = cloudCol;
  material.opacityNode = tslFloat(0.55);
  material.roughnessNode = tslFloat(1.0);
  material.metalnessNode = tslFloat(0.0);

  return material;
}
```

## Performance Notes

- **3D noise texture**: Compute once (128³ or 256³), sample forever. 128³ RGBA8 = 8MB VRAM.
- **Temporal reprojection**: Store previous frame's cloud result, reproject using motion vectors. Only march 25% of pixels per frame, blend with history. 4× speedup.
- **Blue noise jitter**: `texture(blueNoiseTex, screenUV).r * stepSize` eliminates banding far better than white noise hash.
- **Half-resolution**: Render volumetric pass at 50% resolution, bilateral upsample using depth edges. 4× cheaper.
- **Early exit**: `if (density < 0.001) t += stepSize * 2.0` — skip empty space faster.
