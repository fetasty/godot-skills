---
name: godot-csharp
description: >-
  Godot 4 C# code quality: .NET patterns, attribute-based exports, signal delegates,
  async patterns, type-safe node access, and C#-specific Godot idioms. Use when
  writing or reviewing .cs files in Godot 4 projects for: (1) Enforcing partial class
  and nullable reference types, (2) Designing [Signal] delegate architecture,
  (3) Implementing C# design patterns with Godot integration, (4) Managing .csproj
  and NuGet dependencies, (5) Reviewing C# for Godot-specific anti-patterns.
---

# Godot C# Specialist

C# code quality, patterns, and performance for Godot 4.6 projects.

## The `partial class` Requirement (Mandatory)

ALL node scripts MUST be declared as `partial class`:

```csharp
// YES — partial class, matches node type
public partial class PlayerController : CharacterBody3D { }

// NO — source generator fails silently
public class PlayerController : CharacterBody3D { }
```

Godot 4's source generator requires `partial` to inject `_GodotClass` infrastructure.
Missing this causes silent failures that are very hard to debug.

## Static Typing and Null Safety

- Enable nullable reference types: `<Nullable>enable</Nullable>` in `.csproj`
- Use `?` for nullable references; never assume a reference is non-null:

```csharp
private HealthComponent? _healthComponent;   // nullable — may not be assigned yet
private Node3D _cameraRig = null!;            // non-nullable — guaranteed in _Ready()
```

- Prefer explicit types; `var` is acceptable when type is obvious from RHS

## Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Classes | `PascalCase` | `PlayerController`, `WeaponData` |
| Public members | `PascalCase` | `MoveSpeed`, `JumpVelocity` |
| Private fields | `_camelCase` | `_currentHealth`, `_isGrounded` |
| Methods | `PascalCase` | `TakeDamage()`, `GetCurrentHealth()` |
| Constants | `PascalCase` | `MaxHealth`, `DefaultMoveSpeed` |
| Signal delegates | `PascalCaseEventHandler` | `HealthChangedEventHandler` |
| Signal callbacks | `On` prefix | `OnHealthChanged()` |
| Files | `PascalCase.cs` | `PlayerController.cs` |

## Export and Inspector Attributes

```csharp
using Godot;

[Export] public float MoveSpeed { get; set; } = 5.0f;
[Export] public bool IsEnabled { get; set; } = true;

// Enums
public enum DamageType { Physical, Magical, TrueDamage }
[Export] public DamageType DamageType { get; set; }

// Node references — use unique names (%) in .tscn
[Export] public HealthComponent? HealthComponent { get; set; }

// Resources
[Export] public WeaponData? StartingWeapon { get; set; }
```

## Signal Architecture

```csharp
// Declare delegate outside class OR use delegate keyword
[Signal]
public delegate void HealthChangedEventHandler(float newHealth, float oldHealth);

[Signal]
public delegate void DiedEventHandler(Node source);

// Emit
EmitSignal(SignalName.HealthChanged, _currentHealth, _previousHealth);
EmitSignal(SignalName.Died, killer);

// Connect
HealthChanged += OnHealthChanged;

// In _ExitTree():
HealthChanged -= OnHealthChanged;
```

- Signal delegate names MUST end with `EventHandler` — source generator requires this
- Always use `SignalName.X` (not string literals) for type safety
- Unsubscribe in `_ExitTree()` to prevent memory leaks

## Async Patterns

```csharp
// Timers — use SceneTree, not Task.Delay()
await ToSignal(GetTree().CreateTimer(0.5f), SceneTreeTimer.SignalName.Timeout);

// Await signal
await ToSignal(animationPlayer, AnimationMixer.SignalName.AnimationFinished);

// Tween
var tween = CreateTween();
tween.TweenProperty(sprite, "modulate:a", 0.0f, 0.3f);
await ToSignal(tween, Tween.SignalName.Finished);
```

**Never use `Task.Delay()`** — it breaks frame synchronization with the engine.

## Design Patterns

### Composition Over Inheritance

```csharp
private HealthComponent _healthComponent = null!;
private HitboxComponent _hitboxComponent = null!;

public override void _Ready()
{
    _healthComponent = GetNode<HealthComponent>("%HealthComponent");
    _hitboxComponent = GetNode<HitboxComponent>("%HitboxComponent");
    _healthComponent.Died += OnDied;
    _hitboxComponent.HitReceived += OnHitReceived;
}
```

Maximum inheritance depth: 3 levels after `GodotObject`.

### Resource Pattern

```csharp
[GlobalClass]
public partial class WeaponData : Resource
{
    [Export] public float Damage { get; set; } = 10.0f;
    [Export] public float AttackSpeed { get; set; } = 1.0f;
    [Export] public WeaponType WeaponType { get; set; }
}
```

### Autoload Access

```csharp
// In GameManager.cs
public partial class GameManager : Node
{
    public static GameManager Instance { get; private set; } = null!;

    public override void _Ready()
    {
        Instance = this;
    }
}

// Usage elsewhere
GameManager.Instance.PauseGame();
```

## Performance

- Disable `_Process` and `_PhysicsProcess` when not needed:
  ```csharp
  SetProcess(false);
  SetPhysicsProcess(false);
  ```
- `_Process(double delta)` uses `double` — cast to `float` for engine math: `(float)delta`
- Cache `GetNode<T>()` in `_Ready()` — never call inside `_Process`
- Use `StringName` for frequently compared strings: `new StringName("group_name")`
- Avoid LINQ in hot paths (`_Process`, collision callbacks) — allocates garbage
- Prefer `List<T>` over `Godot.Collections.Array<T>` for C#-internal collections
- Use object pooling for frequently spawned objects
- Profile with Godot's profiler AND dotnet counters for GC pressure

## GDScript/C# Boundary

- Keep in C#: complex game systems, data processing, AI, anything unit-tested
- Keep in GDScript: scenes needing fast iteration, level scripts, simple behaviors
- At the boundary: prefer signals over direct cross-language method calls
- Avoid `GodotObject.Call()` (string-based) — define typed interfaces
- Threshold for C# → GDExtension: if a method runs >1000 times per frame AND profiling shows bottleneck

## Common C# Anti-Patterns

- Missing `partial` on node classes — source generator fails silently
- Using `Task.Delay()` instead of `GetTree().CreateTimer()` — breaks frame sync
- Calling `GetNode()` without generics — drops type safety
- Forgetting to disconnect signals in `_ExitTree()` — memory leaks
- Using `Godot.Collections.*` for internal C# data — unnecessary marshalling overhead
- Static fields holding node references — breaks scene reload/multiple instances
- Calling lifecycle methods directly — never call `_Ready()`, `_Process()` yourself
- Capturing `this` in long-lived lambda signal handlers — prevents GC
- Signal delegate names without `EventHandler` suffix — source generator fails
