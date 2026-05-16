# Codex Godot Skills
>:globe_with_meridians: 中文 | [English](./README.md)


一个面向 Godot 4.6 的 AI 编程技能库，为 AI 编程助手（如 Codex CLI、Claude Code 等）提供深度 Godot 引擎知识，帮助 AI 生成更准确、更符合引擎最佳实践的代码。

## 简介

本技能库将 Godot 4.6 引擎的架构模式、编码规范、版本变更、性能优化等专业知识打包为结构化的 **技能（Skill）** 文件。当 AI 助手加载这些技能后，能够：

- 遵循 Godot 4.6 的最新 API（避免使用废弃接口）
- 生成符合各语言规范的代码（GDScript / C# / C++ / Rust）
- 了解 4.4 → 4.6 版本之间的破坏性变更
- 在场景架构、信号设计、资源管理等方面做出正确决策
- 识别并避免常见的反模式与性能陷阱

> **为什么需要这个？** 大多数 AI 模型的训练数据截止于 2025 年 5 月，只覆盖到 Godot ~4.3。从 4.4 到 4.6 引入了大量新特性（Jolt 物理默认、D3D12、GDScript 变参、@abstract 等），本技能库作为「知识补丁」帮助 AI 跟上最新版本。

## 技能总览

| 技能 | 适用场景 |
|------|---------|
| `godot` | 引擎级架构、场景/节点设计、信号、资源管理、Autoload、项目配置、导出预设、版本知识 |
| `godot-gdscript` | `.gd` 文件编写/审查、静态类型、协程、状态机、设计模式、GDScript 优化 |
| `godot-csharp` | `.cs` 文件编写/审查、partial class、Signal 委托、async 模式、.csproj 与 NuGet 管理 |
| `godot-gdextension` | C++ / Rust 原生扩展、godot-cpp、godot-rust、自定义节点、跨平台构建、性能热点优化 |
| `godot-shader` | `.gdshader` 文件、可视化着色器、粒子效果、材质设置、后处理、渲染管线选择 |

## 目录结构

```
codex-godot-skills/
├── README.md
├── godot/                          # 核心 Godot 引擎技能
│   ├── SKILL.md                    # 架构、信号、资源、命名规范等
│   └── references/                 # 截至 4.6 版本的参考资料
│       ├── VERSION.md              # 版本对照与知识盲区警示
│       ├── breaking-changes.md     # 4.4 → 4.6 破坏性变更
│       ├── deprecated-apis.md      # 已废弃 API 及替代方案
│       ├── current-best-practices.md # 新版推荐实践
│       └── modules/                # 按子系统分类的模块参考
│           ├── animation.md
│           ├── audio.md
│           ├── input.md
│           ├── navigation.md
│           ├── networking.md
│           ├── physics.md
│           ├── rendering.md
│           └── ui.md
├── godot-gdscript/                 # GDScript 代码质量技能
│   └── SKILL.md
├── godot-csharp/                   # C# 代码质量技能
│   └── SKILL.md
├── godot-gdextension/              # GDExtension 原生扩展技能
│   └── SKILL.md
└── godot-shader/                   # 着色器与渲染技能
    └── SKILL.md
```

## 安装

将技能目录复制到你的 AI 助手技能路径下即可。

### Codex CLI

```bash
# 复制到 Codex 全局技能目录
~/.codex/skills/
```

### Claude Code

```bash
# 复制到 Claude Code 技能目录
~/.claude/skills/
```

安装后，AI 助手会自动识别并在处理 Godot 相关任务时调用对应技能。

## 使用示例

安装技能后，你可以在自然语言中直接提及相关主题，AI 会自动加载对应技能：

- **"帮我写一个玩家角色控制器的 GDScript"** → 触发 `godot` + `godot-gdscript`
- **"用 C# 实现背包系统的数据结构"** → 触发 `godot` + `godot-csharp`
- **"给角色受击添加一个屏幕空间扭曲着色器"** → 触发 `godot-shader`
- **"这个寻路算法太慢了，用 GDExtension 重写"** → 触发 `godot-gdextension`

你也可以显式指定技能名称：`@godot-gdscript 帮我审查这段代码`。

## 各技能核心要点

### godot（引擎核心）

- 目标版本 **Godot 4.6**，提供 4.4~4.6 的破坏性变更、废弃 API、新增最佳实践
- 场景架构：组合优于继承，继承深度 ≤ 3
- 信号：`signal.emit()` / `signal.connect(callable)` 现代化语法
- Autoload：仅用于真正的全局系统
- 语言选择指南：GDScript → 游戏逻辑 / C# → 复杂系统 / GDExtension → 性能热点

### godot-gdscript

- **强制静态类型**：所有变量、参数、返回值必须有类型注解
- 协程：`await` 替代 `yield()`
- 设计模式：状态机、资源模式、组合模式
- 4.5+ 新特性：变参 `Variant...`、`@abstract` 抽象类

### godot-csharp

- **必须 `partial class`**，否则源码生成器静默失败
- Nullable 引用类型启用
- 信号委托名必须以 `EventHandler` 结尾
- 异步：`ToSignal(GetTree().CreateTimer(...))`，**禁止 `Task.Delay()`**

### godot-gdextension

- C++（godot-cpp）与 Rust（godot-rust）双语言支持
- ABI 版本不兼容警示：小版本升级必须重新编译
- 边界模式：GDScript 负责逻辑，原生负责计算
- 后台线程不得访问场景树

### godot-shader

- 三档渲染器选择：Forward+ / Mobile / Compatibility
- 着色器类型：spatial / canvas_item / particles / fog / sky
- 后处理变更（4.6）：Glow 在色调映射前处理，D3D12 默认
- 渲染预算参考（60 FPS 目标）

## 许可证

本项目遵循 MIT 许可证。

## 贡献

欢迎提交 Issue 和 Pull Request 来改进技能内容。请确保参考 [Godot 官方文档](https://docs.godotengine.org/en/stable/) 验证 API 准确性。
