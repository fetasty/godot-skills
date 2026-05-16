---
name: godot-gdscript
description: >-
  Godot 4 GDScript code quality: static typing, signal architecture, coroutines,
  design patterns, performance optimization, and GDScript-specific idioms. Use when
  writing or reviewing .gd files in Godot 4 projects for: (1) Enforcing static typing
  standards, (2) Designing signal and node communication patterns, (3) Implementing
  state machines/command/observer patterns in GDScript, (4) Optimizing GDScript
  performance, (5) Reviewing GDScript code for anti-patterns.
---

# Godot GDScript Specialist

GDScript 2.0 code quality, patterns, and optimization for Godot 4.6 projects.

## Static Typing (Mandatory)

ALL variables, parameters, and return types must have explicit type annotations:

```gdscript
# YES — typed
var health: float = 100.0
var inventory: Array[Item] = []
func take_damage(amount: float, source: Node3D) -> void:
func get_items() -> Array[Item]:

# NO — untyped
var health = 100.0
func take_damage(amount, source):
```

Use `@onready` for typed node references:

```gdscript
@onready var health_bar: ProgressBar = %HealthBar
@onready var sprite: Sprite2D = $Visuals/Sprite2D
```

Enable `unsafe_*` warnings in project settings to catch untyped code.

## File Organization

- One `class_name` per file — file name matches class in `snake_case`
  - `player_character.gd` → `class_name PlayerCharacter`
- Scripts extend the node type they're attached to
- Use `@export_group` and `@export_subgroup` for inspector organization

## Signal Architecture

```gdscript
signal health_changed(new_health: float, old_health: float)
signal died(source: Node)

# Modern emit syntax (Godot 4)
health_changed.emit(_current_health, _previous_health)
died.emit(killer)

# Typed connections
health_changed.connect(_on_health_changed)
# One-shot
health_changed.connect(_on_damaged_once, CONNECT_ONE_SHOT)
```

- Prefer signals for decoupled communication between nodes
- Never connect signals in `_process()` — connects every frame
- Always disconnect in `_exit_tree()` if node may outlive the connection
- Use `EventBus` autoload for global events, direct signals for parent-child

## Coroutines and Async

```gdscript
# Await a signal
await get_tree().create_timer(0.5).timeout

# Await a tween
var tween := create_tween()
tween.tween_property(sprite, "modulate:a", 0.0, 0.3)
await tween.finished

# Await parallel operations
var timer := get_tree().create_timer(1.0)
var tween := create_tween().tween_property(self, "position", target, 1.0)
await timer.timeout  # or await tween.finished
```

Never use `yield()` — that's Godot 3 syntax. Use `await` exclusively.

## Design Patterns

### State Machine

```gdscript
enum State { IDLE, RUNNING, JUMPING, FALLING, ATTACKING }
var _current_state: State = State.IDLE

func _physics_process(delta: float) -> void:
    match _current_state:
        State.IDLE:
            _process_idle(delta)
        State.RUNNING:
            _process_running(delta)
```

For complex states, use node-based state machines (each state is a child Node).

### Resource Pattern

```gdscript
class_name WeaponData extends Resource
@export var damage: float = 10.0
@export var attack_speed: float = 1.0
@export var weapon_type: WeaponType
```

Resources are shared by default — use `resource.duplicate()` for per-instance data.
Use `duplicate_deep()` for nested resources (Godot 4.5+).

### Composition Over Inheritance

```gdscript
@onready var health_component: HealthComponent = %HealthComponent
@onready var hitbox_component: HitboxComponent = %HitboxComponent

func _ready() -> void:
    health_component.died.connect(_on_died)
    hitbox_component.hit_received.connect(_on_hit_received)
```

Maximum inheritance depth: 3 levels after `Node` base.

## Post-4.3 GDScript Features

From `references/current-best-practices.md` in the `godot` skill:

- **Variadic arguments** (4.5+): `func log(prefix: String, values: Variant...) -> void:`
- **Abstract classes** (4.5+): `@abstract class_name BaseEnemy` and `@abstract func get_pattern()`
- **Script backtracing**: Call stacks available even in Release builds (4.5+)

## Performance

- Disable `_process` and `_physics_process` when not needed:
  ```gdscript
  set_process(false)
  set_physics_process(false)
  ```
- Use `_physics_process` for movement/physics, `_process` for visuals/UI
- Cache node references in `@onready` — never `get_node()` in `_process`
- Use `StringName` for frequently compared strings: `&"animation_name"`
- Avoid `Array.find()` in hot paths — use `Dictionary` lookups
- Object pooling for frequently spawned objects (projectiles, particles)
- Typed arrays (`Array[Type]`) are faster than untyped arrays
- Threshold for GDScript → GDExtension: if a function runs >1000 times per frame

## Common GDScript Anti-Patterns

- Untyped variables and functions — disables compiler optimizations
- Using `$NodePath` in `_process` instead of caching with `@onready`
- Deep inheritance trees instead of composition
- Signals for synchronous communication — use direct method calls
- String comparisons instead of enums or `StringName`
- Dictionaries for structured data instead of typed Resources
- God-class Autoloads that manage everything
- Editor signal connections — invisible in code, hard to track
- Using `yield()` instead of `await`

## Tooling

**ripgrep for GDScript**: No `gdscript` type exists. Always use `--glob "*.gd"`
or `glob: "*.gd"`. Using `--type gdscript` produces a hard error.
