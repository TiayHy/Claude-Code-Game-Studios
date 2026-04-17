# Claude Code Game Studios -- 游戏工作室 Agent 架构

通过 48 个协调的 Claude Code 子 Agent 管理独立游戏开发。
每个 Agent 拥有特定领域，执行关注点分离和质量控制。

## 技术栈

- **引擎**：[选择：Godot 4 / Unity / Unreal Engine 5]
- **语言**：[选择：GDScript / C# / C++ / Blueprint]
- **版本控制**：Git，trunk-based 开发模式
- **构建系统**：[选择引擎后指定]
- **资源流水线**：[选择引擎后指定]

> **注意**：Godot、Unity 和 Unreal 都有引擎专家 Agent 和专门的子专家。使用与你的引擎匹配的套件。

## 项目结构

@.claude/docs/directory-structure.md

## 引擎版本参考

@docs/engine-reference/godot/VERSION.md

## 技术偏好

@.claude/docs/technical-preferences.md

## 协调规则

@.claude/docs/coordination-rules.md

## 协作协议

**用户驱动的协作，而非自主执行。**
每个任务都遵循：**提问 -> 选项 -> 决定 -> 草稿 -> 批准**

- Agent 在使用 Write/Edit 工具前必须问"我可以把这个写到 [filepath] 吗？"
- Agent 在请求批准前必须展示草稿或摘要
- 多文件变更需要明确的完整变更集批准
- 未经用户指示不进行提交

详见 `docs/COLLABORATIVE-DESIGN-PRINCIPLE.md` 获取完整协议和示例。

> **首次会话？** 如果项目没有配置引擎且没有游戏概念，运行 `/start` 开始引导式入职流程。

## 编码标准

@.claude/docs/coding-standards.md

## 上下文管理

@.claude/docs/context-management.md
