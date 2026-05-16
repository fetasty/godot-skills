---
name: godot-gdextension
description: >-
  Godot 4 GDExtension native code integration: godot-cpp (C++), godot-rust, custom
  node types, native performance optimization, cross-platform builds, and the
  GDScript/native boundary. Use when working on Godot 4 GDExtension for:
  (1) Implementing performance-critical systems in C++ or Rust,
  (2) Creating custom node types exposed to the Godot editor,
  (3) Integrating native libraries with Godot, (4) Setting up GDExtension build
  systems (SCons/CMake/Cargo), (5) Deciding what belongs in native code vs GDScript.
---

# Godot GDExtension Specialist

Native code integration for Godot 4.6 via GDExtension: C++ (godot-cpp), Rust
(godot-rust), custom nodes, performance optimization, and cross-platform builds.

## When to Use GDExtension

- Performance-critical computation: pathfinding, procedural generation, physics queries
- Large data processing: world generation, terrain systems, spatial indexing
- Integration with native libraries: networking, audio DSP, image processing
- Systems that run >1000 iterations per frame
- Custom server implementations: custom physics, custom rendering
- Anything benefiting from SIMD, multithreading, or zero-allocation patterns

## When NOT to Use GDExtension

- Simple game logic (state machines, UI, scene management) — use GDScript
- Prototypes or experimental features — use GDScript until proven necessary
- Anything that doesn't measurably benefit from native performance
- If GDScript runs it fast enough, keep it in GDScript

## The Boundary Pattern

- **GDScript owns**: game logic, scene management, UI, high-level coordination
- **Native owns**: heavy computation, data processing, performance-critical hot paths
- **Interface**: native exposes nodes, resources, and functions callable from GDScript
- **Data flow**: GDScript calls native methods with simple types → native computes → returns results

## godot-cpp (C++ Bindings)

### Project Structure

```
project/
├── gdextension/
│   ├── src/
│   │   ├── register_types.cpp    # Module registration
│   │   ├── register_types.h
│   │   └── [source files]
│   ├── godot-cpp/                 # Submodule
│   ├── SConstruct                 # Build file
│   └── [project].gdextension     # Extension descriptor
```

### Class Registration

```cpp
// register_types.h
#pragma once
#include <godot_cpp/core/class_db.hpp>

void initialize_example_module(godot::ModuleInitializationLevel p_level);
void uninitialize_example_module(godot::ModuleInitializationLevel p_level);
```

```cpp
// register_types.cpp
#include "register_types.h"
#include "my_custom_node.h"

#include <godot_cpp/core/class_db.hpp>
#include <godot_cpp/core/defs.hpp>
#include <godot_cpp/godot.hpp>

using namespace godot;

void initialize_example_module(ModuleInitializationLevel p_level) {
    if (p_level != MODULE_INITIALIZATION_LEVEL_SCENE) {
        return;
    }
    ClassDB::register_class<MyCustomNode>();
}

void uninitialize_example_module(ModuleInitializationLevel p_level) {
    if (p_level != MODULE_INITIALIZATION_LEVEL_SCENE) {
        return;
    }
}

extern "C" {
GDotExtensionBool GDE_EXPORT
gdextension_init(GDotExtensionInterface *p_interface, GDotExtensionClassLibraryPtr p_library, GDotExtensionInitialization *r_initialization) {
    godot::GDExtensionBinding::InitObject init_obj(p_interface, p_library, r_initialization);
    init_obj.register_initializer(initialize_example_module);
    init_obj.register_terminator(uninitialize_example_module);
    init_obj.set_minimum_library_initialization_level(MODULE_INITIALIZATION_LEVEL_SCENE);
    return init_obj.init();
}
}
```

### Custom Node Example

```cpp
// my_custom_node.h
#pragma once
#include <godot_cpp/classes/node3d.hpp>

namespace godot {

class MyCustomNode : public Node3D {
    GDCLASS(MyCustomNode, Node3D)

private:
    float speed = 10.0f;

protected:
    static void _bind_methods();

public:
    void set_speed(float p_speed);
    float get_speed() const;

    void _process(double delta) override;
};

} // namespace godot
```

```cpp
// my_custom_node.cpp
#include "my_custom_node.h"
#include <godot_cpp/core/class_db.hpp>

namespace godot {

void MyCustomNode::_bind_methods() {
    // Bind with getter/setter for editor integration
    ClassDB::bind_method(D_METHOD("set_speed", "speed"), &MyCustomNode::set_speed);
    ClassDB::bind_method(D_METHOD("get_speed"), &MyCustomNode::get_speed);
    ADD_PROPERTY(PropertyInfo(Variant::FLOAT, "speed"), "set_speed", "get_speed");

    // Bind a regular method
    ClassDB::bind_method(D_METHOD("do_calculation", "input_value"), &MyCustomNode::do_calculation);
}

void MyCustomNode::set_speed(float p_speed) {
    speed = p_speed;
}

float MyCustomNode::get_speed() const {
    return speed;
}

void MyCustomNode::_process(double delta) {
    // Cast delta to float for engine math
    Vector3 pos = get_position();
    pos.y += speed * (float)delta;
    set_position(pos);
}

} // namespace godot
```

## godot-rust (Rust Bindings)

```rust
use godot::prelude::*;

#[derive(GodotClass)]
#[class(base=Node3D)]
struct MyRustNode {
    #[export]
    speed: f64,

    base: Base<Node3D>,
}

#[godot_api]
impl INode3D for MyRustNode {
    fn init(base: Base<Node3D>) -> Self {
        Self { speed: 10.0, base }
    }

    fn process(&mut self, delta: f64) {
        let mut pos = self.base().get_position();
        pos.y += (self.speed * delta) as f32;
        self.base_mut().set_position(pos);
    }
}
```

### C++ vs Rust Decision

| Aspect | godot-cpp (C++) | godot-rust |
|--------|----------------|------------|
| Performance | Maximum | Near-maximum |
| Editor integration | Full | Full |
| Build system | SCons | Cargo |
| Safety | Manual memory management | Compile-time memory safety |
| Ecosystem | Larger, more examples | Smaller but growing |
| When to use | When C++ is required or team knows C++ | When safety/concurrency matter or team knows Rust |

## Build System

### godot-cpp (SCons)

```bash
scons platform=windows target=template_debug
scons platform=windows target=template_release
```

CI must build for all target platforms: windows, linux, macos.
Debug: symbols + runtime checks. Release: stripped + full optimization.

### godot-rust (Cargo)

```bash
cargo build
cargo build --release
```

```toml
# Cargo.toml
[profile.release]
opt-level = 3
lto = "thin"
```

### .gdextension File

```ini
[configuration]
entry_symbol = "gdext_rust_init"
compatibility_minimum = "4.2"

[libraries]
linux.debug.x86_64 = "res://rust/target/debug/lib[project].so"
linux.release.x86_64 = "res://rust/target/release/lib[project].so"
windows.debug.x86_64 = "res://rust/target/debug/[project].dll"
windows.release.x86_64 = "res://rust/target/release/[project].dll"
macos.debug = "res://rust/target/debug/lib[project].dylib"
macos.release = "res://rust/target/release/lib[project].dylib"
```

## ABI Compatibility Warning

GDExtension binaries are **NOT ABI-compatible across minor Godot versions**:
- A binary compiled for Godot 4.3 will NOT work with Godot 4.4 without recompilation
- Always recompile extensions when the project upgrades Godot
- Before recommending extension patterns, check the current version in the `godot` skill's `references/VERSION.md`
- Flag: "This extension will need recompilation if Godot version changes."

## Performance Patterns

### Data-Oriented Design
- Process data in contiguous arrays, not scattered objects
- Structure of Arrays (SoA) over Array of Structures (AoS) for batch processing
- Minimize Godot API calls in tight loops — batch data, process natively, return results
- Use SIMD intrinsics or auto-vectorizable loops for math-heavy code

### Threading
- Use native threading (std::thread, rayon) for background computation
- **NEVER** access Godot scene tree from background threads
- Pattern: schedule on background thread → collect results → apply in `_process()`
- Use `call_deferred()` for thread-safe Godot API calls

### Profiling
- Godot's built-in profiler for high-level timing
- Platform profilers (VTune, perf, Instruments) for native code details
- Custom profiling markers via Godot's profiler API
- Measure: time in native vs time in GDScript for the same operation

## Common GDExtension Anti-Patterns

- Moving ALL code to native — GDScript is fast enough for most logic
- Frequent Godot API calls in tight loops — each call has boundary overhead
- Not handling hot-reload — extension should survive editor reimport
- Platform-specific code without cross-platform abstractions
- Forgetting to register classes/methods — invisible to GDScript
- Using raw pointers for Godot objects instead of `Ref<T>` / `Gd<T>`
- Not building for all target platforms in CI
- Allocating in hot paths instead of pre-allocating buffers

## Coordination

- The `godot` skill contains engine reference docs (`references/`) — check them for version-specific API changes
- The `godot-gdscript` skill covers the GDScript/native boundary from the GDScript side
- The `godot-shader` skill covers compute shader vs native alternatives
