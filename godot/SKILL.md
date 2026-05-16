---
name: godot
description: >-
  Godot Engine 4 specialist: architecture patterns, scene/node design, signals,
  resources, autoloads, project settings, export presets, and engine-level decisions.
  Use when working on Godot 4 game projects for: (1) Designing scene/node architecture,
  (2) Deciding between GDScript vs C# vs GDExtension, (3) Setting up autoloads or
  singletons, (4) Configuring project settings or export presets, (5) Reviewing Godot
  code for engine best practices, (6) Any general Godot 4 question.
  Includes bundled engine reference docs for version 4.6 with breaking changes,
  deprecated APIs, current best practices, and module references.
---

# Godot Engine Specialist

This skill provides Godot 4.6 engine expertise: architecture, scene/node patterns,
signals, resources, autoloads, project configuration, and engine best practices.
Bundled reference docs cover breaking changes, deprecated APIs, and post-cutoff
features the model may not know.

## Version Awareness

**CRITICAL**: Training data has a May 2025 cutoff. The bundled references cover
Godot 4.4 through 4.6. Before suggesting any Godot API, read these files:

1. `references/VERSION.md` — current engine version
2. `references/deprecated-apis.md` — avoid deprecated APIs
3. `references/breaking-changes.md` — version-specific changes
4. `references/current-best-practices.md` — new/updated patterns
5. `references/modules/<subsystem>.md` — subsystem-specific reference

Key post-cutoff changes: Jolt Physics default (4.6), D3D12 default on Windows (4.6),
glow before tonemapping (4.6), variadic arguments and @abstract in GDScript (4.5),
stencil buffer and SMAA (4.5), Accessibility via AccessKit (4.5).

When in doubt, prefer APIs documented in the reference files over training data.

## Language Decision Guide

| Language | Best For | Avoid For |
|----------|----------|-----------|
| GDScript | Game logic, rapid iteration, scenes, UI, prototyping, 2D | Heavy computation, data processing >1000/frame |
| C# | Complex systems, AI, data processing, unit-tested code | Simple behaviors, scenes needing fast iteration |
| GDExtension | Hot paths >1000/frame, native libs, SIMD, multithreading | Game logic, prototypes, simple behaviors |

Prefer signals over direct cross-language method calls at language boundaries.

## Scene and Node Architecture

- Prefer composition over inheritance — attach behavior via child nodes
- Each scene should be self-contained and reusable
- Use `@onready` for node references, never hardcoded distant paths
- Scenes should have a single root node with a clear responsibility
- Use `PackedScene` for instantiation
- Keep the scene tree shallow — deep nesting causes performance and readability issues
- Maximum inheritance depth: 3 levels after `Node` base class

## Naming Conventions

**GDScript**:
- Files: `snake_case.gd` matching `class_name`
- Classes: `PascalCase` (`class_name PlayerCharacter`)
- Functions/variables: `snake_case` (`func calculate_damage()`, `var current_health: float`)
- Constants: `SCREAMING_SNAKE_CASE` (`const MAX_SPEED: float = 500.0`)
- Signals: `snake_case`, past tense (`signal health_changed`, `signal died`)
- Private members: prefix with `_` (`var _internal_state: int`)
- Scenes: `snake_case.tscn` matching root class

**C#**:
- Files: `PascalCase.cs` matching class name
- Classes: `PascalCase`
- Public members: `PascalCase`
- Private fields: `_camelCase`
- Methods: `PascalCase`
- Constants: `PascalCase`
- Signal delegates: `PascalCase + EventHandler`
- Scenes: `PascalCase.tscn`

## Resource Management

- Use `Resource` subclasses for data-driven content (items, abilities, stats)
- Save shared data as `.tres` files, not hardcoded in scripts
- Use `load()` for small resources, `ResourceLoader.load_threaded_request()` for large assets
- Custom resources must implement `_init()` with default values
- Use `resource.duplicate()` for per-instance data; use `duplicate_deep()` for nested trees (4.5+)

## Signals

- Use signals for decoupled communication
- `signal.emit(args)` not `emit_signal("name", args)`
- Connect via `signal.connect(callable)`, never string-based `connect()`
- Check `is_connected()` before connecting or use `CONNECT_ONE_SHOT`
- Always include types in signal declarations
- Prefer `EventBus` (autoload) for global events, direct signals for parent-child

## Autoloads

- Use sparingly — only for truly global systems:
  - `EventBus` — global signal hub
  - `GameManager` — game state (pause, scene transitions)
  - `SaveManager` — save/load system
  - `AudioManager` — music and SFX management
- Autoloads must NOT hold references to scene-specific nodes
- Never use autoloads as a dumping ground for utility functions

## Performance

- Minimize `_process()` and `_physics_process()` — disable with `set_process(false)` when idle
- Use `Tween` for animations instead of manual interpolation in `_process()`
- Object pooling for frequently instantiated scenes (projectiles, particles, enemies)
- Use `VisibleOnScreenNotifier2D/3D` to disable off-screen processing
- Use `MultiMeshInstance` for large numbers of identical meshes
- Profile with Godot's built-in profiler and `Performance` singleton

## Common Pitfalls

- Using `get_node()` with long relative paths instead of signals or groups
- Processing every frame when event-driven would suffice
- Not freeing nodes (`queue_free()`) — memory leaks with orphan nodes
- Connecting signals in `_process()` (connects every frame, massive leak)
- Using `@tool` scripts without proper editor safety checks
- Ignoring `tree_exited` signal for cleanup
- Not using typed arrays: `var enemies: Array[Enemy] = []`

## Tooling

**ripgrep for GDScript**: There is no `gdscript` type in ripgrep. `*.gd` files are
registered under the `gap` type. Always use `--glob "*.gd"` or `glob: "*.gd"`
in the Grep tool — `--type gdscript` produces a hard error.
