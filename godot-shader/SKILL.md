---
name: godot-shader
description: >-
  Godot 4 shaders, materials, visual effects, and rendering customization: Godot
  shading language (.gdshader), visual shader graphs, particle shaders, material
  setup, post-processing, and GPU performance optimization. Use when working on
  Godot 4 rendering for: (1) Writing or optimizing .gdshader files, (2) Designing
  visual shader graphs, (3) Creating particle effects or GPU-driven VFX,
  (4) Configuring render pipelines (Forward+/Mobile/Compatibility),
  (5) Optimizing draw calls and shader complexity, (6) Setting up post-processing.
---

# Godot Shader Specialist

Godot 4 shaders, materials, visual effects, rendering customization, and GPU
performance. Covers the Godot shading language, visual shader graphs, particle
shaders, materials, post-processing, and render pipeline selection.

## Renderer Selection

### Forward+ (Default for Desktop)
- Use for: PC, console, high-end mobile
- Features: clustered lighting, volumetric fog, SDFGI, SSAO, SSR, glow
- Supports unlimited real-time lights via clustered rendering
- Best visual quality, highest GPU cost

### Mobile Renderer
- Use for: mobile devices, low-end hardware
- Features: limited lights per object (8 omni + 8 spot), no volumetrics
- Lower precision, fewer post-process options

### Compatibility Renderer
- Use for: web exports, very old hardware (OpenGL 3.3/WebGL 2)
- No compute shaders — plan visual design around this if targeting web
- Most limited feature set

## Shader File Standards

- One shader per file — file name matches material purpose
- Naming: `[type]_[category]_[name].gdshader`
  - `spatial_env_water.gdshader` (3D environment water)
  - `canvas_ui_healthbar.gdshader` (2D UI health bar)
  - `particles_combat_sparks.gdshader` (particle effect)
- Use `#include` (Godot 4.3+) or `#define` for shared functions

## Shader Types

| Type | Use |
|------|-----|
| `shader_type spatial` | 3D mesh rendering |
| `shader_type canvas_item` | 2D sprites, UI elements |
| `shader_type particles` | GPU particle behavior |
| `shader_type fog` | Volumetric fog effects |
| `shader_type sky` | Procedural sky rendering |

## Code Standards

```glsl
shader_type spatial;

// Uniforms with hints for artist-exposed parameters
uniform vec4 albedo_color : source_color = vec4(1.0);
uniform float roughness : hint_range(0.0, 1.0) = 0.5;
uniform sampler2D albedo_texture : source_color, filter_linear_mipmap, repeat_enable;
uniform sampler2D normal_map : hint_normal, filter_linear_mipmap, repeat_enable;

// Group uniforms for inspector organization — Godot 4.3+
group_uniforms Albedo;
uniform vec4 albedo_color : source_color = vec4(1.0);
uniform sampler2D albedo_texture : source_color, filter_linear_mipmap, repeat_enable;
group_uniforms Normal;
uniform sampler2D normal_map : hint_normal, filter_linear_mipmap, repeat_enable;
```

- All uniforms must have descriptive names and appropriate hints
- Comment non-obvious calculations, especially math-heavy sections
- No magic numbers — use named constants or documented uniform values

## Material Setup

### StandardMaterial3D
- Use when a `spatial` shader is unnecessary
- Features: albedo, metallic/roughness, emission, normal, AO, detail maps
- Performance: significantly cheaper than custom shaders
- Good for 80%+ of project materials

### ShaderMaterial
- Use when `StandardMaterial3D` doesn't have a needed feature
- Attach a `.gdshader` file or use `ShaderMaterial` for code-defined shaders

### Material Overlay
- Adds an extra render pass on a `MeshInstance3D`
- Use sparingly — each overlay adds a draw call per mesh
- Good for: damage flash, selection highlight, temporary effects

## Post-4.3 Rendering Changes

From the `godot` skill references — key changes the model may not know:

- **4.6**: D3D12 default on Windows (was Vulkan), glow processes BEFORE tonemapping,
  SSR overhauled, AgX tonemapper has new white point/contrast controls
- **4.5**: Shader Baker (pre-compile to eliminate startup hitching), SMAA 1x,
  stencil buffer support, bent normal maps, specular occlusion
- **4.4**: Shader texture types changed from `Texture2D` to `Texture`

## Particle Systems

### GPU Particles (preferred)
- Use `GPUParticles3D`/`GPUParticles2D` for large counts (100+)
- Write `shader_type particles` for custom behavior
- Particle shader controls: spawn, velocity, color over lifetime, size over lifetime
- Use `TRANSFORM` for position, `VELOCITY` for movement, `COLOR`/`CUSTOM` for data

### CPU Particles
- Use `CPUParticles3D`/`CPUParticles2D` for small counts (<50) or Compatibility renderer
- Simpler setup — use inspector properties

### Particle Performance
- Set `lifetime` to minimum needed
- Use `visibility_aabb` to cull off-screen particles
- LOD: reduce particle count at distance
- Target: all particle systems combined < 2ms GPU time

## Post-Processing

### WorldEnvironment
- `WorldEnvironment` node with `Environment` resource for scene-wide effects
- Configure: glow, tone mapping, SSAO, SSR, fog, adjustments
- Multiple environments for different areas (indoor vs outdoor)

### Compositor Effects (4.3+)
- Custom full-screen effects via `CompositorEffect` scripts
- Access screen texture, depth, normals for custom passes
- Use sparingly — each effect adds a full-screen pass

### Screen-Space Shaders
```glsl
uniform sampler2D screen_texture : hint_screen_texture;
uniform sampler2D depth_texture : hint_depth_texture;
```
- Apply via `ColorRect` or `TextureRect` covering the viewport
- Use for: heat distortion, underwater, damage vignette, blur effects

## Performance Optimization

### Draw Call Management
- `MultiMeshInstance3D` for repeated objects — batches draw calls
- `MeshInstance3D.material_overlay` sparingly — extra draw call per mesh
- Merge static geometry where possible
- Profile draw calls with Profiler and `Performance.get_monitor()`

### Shader Complexity
- Minimize texture samples in fragment shaders — expensive on mobile
- Use `hint_default_white`/`hint_default_black` for optional textures
- Avoid dynamic branching in fragment shaders — use `mix()` and `step()` instead
- Pre-compute expensive operations in vertex shader when possible
- LOD materials: simplified shaders for distant objects

### Render Budgets (60 FPS target)

| Subsystem | Budget |
|-----------|--------|
| Geometry rendering | 4-6ms |
| Lighting | 2-3ms |
| Shadows | 2-3ms |
| Particles/VFX | 1-2ms |
| Post-processing | 1-2ms |
| UI | < 1ms |

## Common Shader Anti-Patterns

- Texture reads in a loop — exponential cost
- Full precision (`highp`) everywhere on mobile — use `mediump`/`lowp` where possible
- Dynamic branching on per-pixel data — unpredictable on GPUs
- Not using mipmaps on textures sampled at varying distances
- Overdraw from transparent objects without depth pre-pass
- Post-processing effects sampling screen texture multiple times — blur should be two-pass
- Not setting `render_priority` on transparent materials — incorrect sort order
