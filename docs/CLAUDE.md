# 文档目录

在创作或编辑本目录中的文件时，请遵循这些标准。

## 架构决策记录（`docs/architecture/`）

使用 ADR 模板：`.claude/docs/templates/architecture-decision-record.md`

**必填部分：** 标题、状态、背景、决策、后果、
ADR 依赖关系、引擎兼容性、已解决的 GDD 需求

**状态生命周期：** `Proposed` → `Accepted` → `Superseded`
- 永远不要跳过 `Accepted` — 引用 `Proposed` ADR 的故事会被自动阻塞
- 使用 `/architecture-decision` 通过引导流程创建 ADR

**TR 注册表：** `docs/architecture/tr-registry.yaml`
- 稳定的需求 ID（例如 `TR-MOV-001`），将 GDD 需求链接到故事
- 永远不要对现有 ID 重新编号 — 只追加新的
- 由 `/architecture-review` 阶段 8 更新

**控制清单：** `docs/architecture/control-manifest.md`
- 平面化程序员规则表：每层的 Required / Forbidden / Guardrails
- 标题中有日期戳的 `Manifest Version:`
- 故事嵌入此版本；`/story-done` 检查是否过期

**验证：** 完成一组 ADR 后运行 `/architecture-review`。

## 引擎参考（`docs/engine-reference/`）

版本固定的引擎 API 快照。**在使用任何引擎 API 前务必先查阅此处** — 
LLM 的训练数据早于固定引擎版本。

当前引擎：见 `docs/engine-reference/godot/VERSION.md`
