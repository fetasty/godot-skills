# Codex Godot Skills

> :globe_with_meridians: [中文版本](./README_zh.md) | English

A curated AI coding skill library for **Godot 4.6**, providing deep engine expertise to AI coding assistants (Codex CLI, Claude Code, etc.) to generate accurate, best-practice-compliant Godot code.

## Overview

This skill library packages Godot 4.6 engine knowledge — architecture patterns, coding conventions, version changes, performance optimization — into structured **Skill** files. When loaded by an AI assistant, these skills help it:

- Follow Godot 4.6''s latest APIs (avoiding deprecated interfaces)
- Generate language-idiomatic code (GDScript / C# / C++ / Rust)
- Understand breaking changes between versions 4.4 → 4.6
- Make informed decisions on scene architecture, signal design, and resource management
- Identify and avoid common anti-patterns and performance pitfalls

> **Why is this needed?** Most AI models have a training cutoff around May 2025, covering only up to Godot ~4.3. Versions 4.4 through 4.6 introduced significant new features (Jolt Physics default, D3D12, GDScript variadic args, `@abstract`, etc.). This library acts as a knowledge patch to bring AI up to speed with the latest release.

## Skill Overview

| Skill | Use Cases |
|-------|----------|
| `godot` | Engine-level architecture, scene/node design, signals, resources, autoloads, project settings, export presets, version awareness |
| `godot-gdscript` | `.gd` file authoring/review, static typing, coroutines, state machines, design patterns, GDScript optimization |
| `godot-csharp` | `.cs` file authoring/review, partial class, Signal delegates, async patterns, `.csproj` and NuGet management |
| `godot-gdextension` | C++ / Rust native extensions, godot-cpp, godot-rust, custom nodes, cross-platform builds, performance hot-path optimization |
| `godot-shader` | `.gdshader` files, visual shader graphs, particle effects, material setup, post-processing, render pipeline selection |

## Directory Structure

```
codex-godot-skills/
├── README.md
├── README_zh.md
├── LICENSE
├── godot/                          # Core Godot engine skill
│   ├── SKILL.md                    # Architecture, signals, resources, naming, etc.
│   └── references/                 # Reference docs up to version 4.6
│       ├── VERSION.md              # Version matrix and knowledge gap warnings
│       ├── breaking-changes.md     # 4.4 → 4.6 breaking changes
│       ├── deprecated-apis.md      # Deprecated APIs and replacements
│       ├── current-best-practices.md # New/updated best practices
│       └── modules/                # Subsystem-specific references
│           ├── animation.md
│           ├── audio.md
│           ├── input.md
│           ├── navigation.md
│           ├── networking.md
│           ├── physics.md
│           ├── rendering.md
│           └── ui.md
├── godot-gdscript/                 # GDScript code quality skill
│   └── SKILL.md
├── godot-csharp/                   # C# code quality skill
│   └── SKILL.md
├── godot-gdextension/              # GDExtension native extension skill
│   └── SKILL.md
└── godot-shader/                   # Shader & rendering skill
    └── SKILL.md
```

## Installation

Copy the skill directories to your AI assistant''s skills path.

### Codex CLI

```powershell
Copy-Item -Recurse godot, godot-gdscript, godot-csharp, godot-gdextension, godot-shader $env:USERPROFILE\.codex\skills\
```

### Claude Code

```powershell
Copy-Item -Recurse godot, godot-gdscript, godot-csharp, godot-gdextension, godot-shader $env:USERPROFILE\.claude\skills\
```

Once installed, the AI assistant automatically loads the relevant skill when handling Godot-related tasks.

## Usage Examples

After installing the skills, you can mention topics naturally — the AI will load the appropriate skill:

- **"Write a player character controller in GDScript"** → triggers `godot` + `godot-gdscript`
- **"Implement an inventory data structure in C#"** → triggers `godot` + `godot-csharp`
- **"Add a screen-space distortion shader for damage feedback"** → triggers `godot-shader`
- **"This pathfinding algorithm is too slow, rewrite it in GDExtension"** → triggers `godot-gdextension`

You can also explicitly name a skill: `@godot-gdscript review this code`.

## Key Highlights Per Skill

### godot (Engine Core)

- Targets **Godot 4.6**, with breaking changes, deprecated APIs, and updated best practices for 4.4–4.6
- Scene architecture: composition over inheritance, max inheritance depth ≤ 3
- Signals: `signal.emit()` / `signal.connect(callable)` modern syntax
- Autoloads: reserved for truly global systems only
- Language decision guide: GDScript → gameplay / C# → complex systems / GDExtension → hot paths

### godot-gdscript

- **Mandatory static typing**: all variables, parameters, and return values must be annotated
- Coroutines: `await` replaces `yield()`
- Design patterns: state machines, resource pattern, composition pattern
- 4.5+ features: variadic args `Variant...`, `@abstract` classes

### godot-csharp

- **Must use `partial class`**, otherwise the source generator fails silently
- Nullable reference types enabled
- Signal delegate names must end with `EventHandler`
- Async: `ToSignal(GetTree().CreateTimer(...))`, **never `Task.Delay()`**

### godot-gdextension

- Dual language support: C++ (godot-cpp) and Rust (godot-rust)
- ABI incompatibility warning: minor version upgrades require recompilation
- Boundary pattern: GDScript owns logic, native owns computation
- Never access the scene tree from background threads

### godot-shader

- Three renderer tiers: Forward+ / Mobile / Compatibility
- Shader types: spatial / canvas_item / particles / fog / sky
- Post-processing changes (4.6): glow processes before tonemapping, D3D12 default
- Render budget reference (60 FPS target)

## License

MIT License. See [LICENSE](./LICENSE) for details.

## Contributing

Issues and pull requests are welcome to improve the skill content. Please verify API accuracy against the [official Godot documentation](https://docs.godotengine.org/en/stable/).
