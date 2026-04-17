---
name: start
description: "首次上手引导——先了解你目前的状态，然后引导你到正确的工作流。不做任何假设。"
argument-hint: "[no arguments]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, AskUserQuestion
---

# 引导式上手

此技能写入一个文件：`production/review-mode.txt`（在阶段 3b 中设置的审查模式配置）。

此技能是新用户的入口。它**不**假设你有游戏想法、引擎偏好或任何先前经验。它先询问，然后引导你到正确的工作流。

---

## 阶段 1：检测项目状态

在询问任何问题之前，静默收集上下文，以便调整引导方式。不要未经提示就展示这些结果——它们服务于你的建议，而非对话开场白。

检查：
- **引擎已配置？** 读取 `.claude/docs/technical-preferences.md`。如果 Engine 字段包含 `[TO BE CONFIGURED]`，则引擎未配置。
- **游戏概念存在？** 检查 `design/gdd/game-concept.md` 是否存在。
- **源代码存在？** 在 `src/` 中 glob 源文件（`*.gd`、`*.cs`、`*.cpp`、`*.h`、`*.rs`、`*.py`、`*.js`、`*.ts`）。
- **原型存在？** 检查 `prototypes/` 下的子目录。
- **设计文档存在？** 统计 `design/gdd/` 中的 markdown 文件数量。
- **生产制品存在？** 检查 `production/sprints/` 或 `production/milestones/` 中是否有文件。

在内部存储这些发现，用于验证用户的自我评估并调整建议。

---

## 阶段 2：询问用户处于哪个阶段

这是用户看到的第一个问题。使用这些精确选项的 `AskUserQuestion`，以便用户可以点击而非输入：

- **Prompt**："欢迎使用 Claude Code Game Studios！在建议任何内容之前，我想了解你目前的游戏想法处于什么阶段。你现在在哪里？"
- **Options**：
  - `A) 还没有想法` — 我完全没有游戏概念。我想要探索并弄清楚做什么。
  - `B) 有一个模糊的想法` — 我心里有一个模糊的主题、感觉或题材（比如"太空相关的东西"或"一个休闲农场游戏"），但还没有具体化。
  - `C) 概念清晰` — 我知道核心想法——题材、基本机制、也许还有一句 pitch——但还没有将其正式化为文档。
  - `D) 已有工作` — 我已经有设计文档、原型、代码或大量规划。我想整理或继续这个工作。

等待用户的选择。在他们回复之前不要继续。

---

## 阶段 3：根据答案分流

#### 如果选 A：还没有想法

用户需要先进行创意探索。

1. 承认从零开始完全可以
2. 简要说明 `/brainstorm` 的作用（使用专业框架的引导式构思——MDA、玩家心理学、动词优先设计）。提到它有两种模式：`/brainstorm open` 用于完全开放的探索，或者 `/brainstorm [hint]` 如果他们有哪怕模糊的主题（比如"太空"、"休闲"、"恐怖"）。
3. 建议下一步运行 `/brainstorm open`，但如果他们想到什么也欢迎使用 hint
4. 展示推荐路径：
   **概念阶段：**
   - `/brainstorm open` — 发现你的游戏概念
   - `/setup-engine` — 配置引擎（brainstorm 会推荐一个）
   - `/art-bible` — 定义视觉身份（使用 brainstorm 产生的视觉身份锚点）
   - `/map-systems` — 将概念分解为系统
   - `/design-system` — 为每个 MVP 系统撰写 GDD
   - `/review-all-gdds` — 跨系统一致性检查
   - `/gate-check` — 在架构工作前验证就绪状态
   **架构阶段：**
   - `/create-architecture` — 产出主架构蓝图和所需 ADR 列表
   - `/architecture-decision (×N)` — 记录关键的技术决策，遵循所需 ADR 列表
   - `/create-control-manifest` — 将决策编译为可执行规则表
   - `/architecture-review` — 验证架构覆盖率
   **前期制作阶段：**
   - `/ux-design` — 为关键界面（主菜单、HUD、核心交互）撰写 UX 规格
   - `/prototype` — 构建一个可丢弃的原型以验证核心机制
   - `/playtest-report (×1+)` — 记录每个垂直切片 playtest 会话
   - `/create-epics` — 将系统映射到史诗
   - `/create-stories` — 将史诗分解为可实现的故事
   - `/sprint-plan` — 规划第一个冲刺
   **生产阶段：** → 用 `/dev-story` 拾取故事

#### 如果选 B：有一个模糊的想法

1. 让他们分享那个模糊的想法——几个词就够了
2. 将这个想法验证为起点（不要评判或重定向）
3. 建议运行 `/brainstorm [他们的 hint]` 来发展它
4. 展示推荐路径：
   **概念阶段：**
   - `/brainstorm [hint]` — 将想法发展成完整概念
   - `/setup-engine` — 配置引擎
   - `/art-bible` — 定义视觉身份（使用 brainstorm 产生的视觉身份锚点）
   - `/map-systems` — 将概念分解为系统
   - `/design-system` — 为每个 MVP 系统撰写 GDD
   - `/review-all-gdds` — 跨系统一致性检查
   - `/gate-check` — 在架构工作前验证就绪状态
   **架构阶段：**
   - `/create-architecture` — 产出主架构蓝图和所需 ADR 列表
   - `/architecture-decision (×N)` — 记录关键的技术决策，遵循所需 ADR 列表
   - `/create-control-manifest` — 将决策编译为可执行规则表
   - `/architecture-review` — 验证架构覆盖率
   **前期制作阶段：**
   - `/ux-design` — 为关键界面（主菜单、HUD、核心交互）撰写 UX 规格
   - `/prototype` — 构建一个可丢弃的原型以验证核心机制
   - `/playtest-report (×1+)` — 记录每个垂直切片 playtest 会话
   - `/create-epics` — 将系统映射到史诗
   - `/create-stories` — 将史诗分解为可实现的故事
   - `/sprint-plan` — 规划第一个冲刺
   **生产阶段：** → 用 `/dev-story` 拾取故事

#### 如果选 C：概念清晰

1. 让他们用一句话描述概念——题材和核心机制。用纯文本，不使用 AskUserQuestion（这是开放回答）。
2. 承认这个概念，然后用 `AskUserQuestion` 提供两条路径：
   - **Prompt**："你想怎么继续？"
   - **Options**：
     - `先将其正式化` — 运行 `/brainstorm [concept]` 将其结构化为正式的游戏概念文档
     - `直接开始` — 现在去 `/setup-engine`，之后手动写 GDD
3. 展示推荐路径：
   **概念阶段：**
   - `/brainstorm` 或 `/setup-engine` — （从步骤 2 他们的选择）
   - `/art-bible` — 定义视觉身份（如果运行了 brainstorm 则在之后，或者在概念文档存在之后）
   - `/design-review` — 验证概念文档
   - `/map-systems` — 将概念分解为独立系统
   - `/design-system` — 为每个 MVP 系统撰写 GDD
   - `/review-all-gdds` — 跨系统一致性检查
   - `/gate-check` — 在架构工作前验证就绪状态
   **架构阶段：**
   - `/create-architecture` — 产出主架构蓝图和所需 ADR 列表
   - `/architecture-decision (×N)` — 记录关键的技术决策，遵循所需 ADR 列表
   - `/create-control-manifest` — 将决策编译为可执行规则表
   - `/architecture-review` — 验证架构覆盖率
   **前期制作阶段：**
   - `/ux-design` — 为关键界面（主菜单、HUD、核心交互）撰写 UX 规格
   - `/prototype` — 构建一个可丢弃的原型以验证核心机制
   - `/playtest-report (×1+)` — 记录每个垂直切片 playtest 会话
   - `/create-epics` — 将系统映射到史诗
   - `/create-stories` — 将史诗分解为可实现的故事
   - `/sprint-plan` — 规划第一个冲刺
   **生产阶段：** → 用 `/dev-story` 拾取故事

#### 如果选 D：已有工作

1. 分享你在阶段 1 发现的情况：
   - "我看到你有 [X 个源文件 / Y 个设计文档 / Z 个原型]……"
   - "你的引擎 [已配置为 X / 尚未配置]……"

2. **D1 子情况 — 早期阶段**（引擎未配置或仅有游戏概念）：
   - 如果引擎未配置，建议先运行 `/setup-engine`
   - 然后运行 `/project-stage-detect` 进行缺口盘点

   **D2 子情况 — 已有 GDD、ADR 或故事：**
   - 解释："有文件不代表模板的技能能够使用它们。GDD 可能缺少必需的部分。`/adopt` 专门检查这个。"
   - 建议：
     1. `/project-stage-detect` — 了解当前阶段和完全缺失的部分
     2. `/adopt` — 检查现有制品是否格式正确

3. 为 D2 展示推荐路径：
   - `/project-stage-detect` — 阶段检测 + 存在性缺口
   - `/adopt` — 格式合规审计 + 迁移计划
   - `/setup-engine` — 如果引擎未配置
   - `/design-system retrofit [path]` — 填补缺失的 GDD 部分
   - `/architecture-decision retrofit [path]` — 添加缺失的 ADR 部分
   - `/architecture-review` — 引导 TR 要求注册表
   - `/gate-check` — 验证下一阶段的就绪状态

---

## 阶段 3b：设置审查模式

检查 `production/review-mode.txt` 是否已存在。

**如果已存在**：读取并显示当前模式——"审查模式设置为 `[当前]`。"——然后进入阶段 4。不要再次询问。

**如果不存在**：使用 `AskUserQuestion`：

- **Prompt**："一个设置选项：在工作流中，你需要多少设计审查？"
- **Options**：
  - `Full` — 总监专家在每个关键工作流步骤进行审查。适合团队、学习工作流、或想要每个决策都得到深入反馈。
  - `Lean（推荐）` — 仅在阶段门禁转换时（/gate-check）有总监参与。跳过每个技能的审查。对独立开发者和小型团队来说是平衡的方法。
  - `Solo` — 完全无总监审查。最高速度。适合 game jam、原型、或觉得审查是负担的情况。

在用户选择后立即将选择写入 `production/review-mode.txt`——不需要单独的"可以写入吗？"因为写入是选择的直接结果：
- `Full` → 写入 `full`
- `Lean（推荐）` → 写入 `lean`
- `Solo` → 写入 `solo`

如果 `production/` 目录不存在则创建。

---

## 阶段 4：继续前确认

在展示推荐路径后，使用 `AskUserQuestion` 询问用户他们想首先采取哪一步。永远不要自动运行下一个技能。

- **Prompt**："你想从 [推荐的第一步] 开始吗？"
- **Options**：
  - `好，我们从 [推荐的第一步] 开始`
  - `我想先做点别的`

---

## 阶段 5：交接

当用户确认他们的下一步时，用一行简短的文字回复："输入 `[skill command]` 开始。"不要其他内容。不要重新解释技能或添加鼓励。`/start` 技能的职责到此为止。

裁决：**COMPLETE**——用户已定向并交接给下一步。

---

## 边缘情况

- **用户选 D 但项目为空**：温和重定向——"看起来项目是一个全新的模板，没有任何制品。路径 A 或 B 更合适吗？"
- **用户选 A 但项目有代码**：提及发现的情况——"我注意到 `src/` 中已经有代码了。你是想选 D（已有工作）吗？"
- **用户是回归的（引擎已配置，概念存在）**：完全跳过上手——"看起来你已经设置好了！你的引擎是 [X]，游戏概念在 `design/gdd/game-concept.md`。审查模式：`[从 production/review-mode.txt 读取，或缺失时为 'lean（默认）']`。想从上次停下的地方继续吗？试试 `/sprint-plan` 或直接告诉我你想做什么。"
- **用户不符合任何选项**：让他们用自己的话描述情况，然后调整。

---

## 协作协议

1. **先问** — 永远不要假设用户的状态或意图
2. **提供选项** — 给出清晰路径，不是命令
3. **用户决定** — 他们选择方向
4. **不要自动执行** — 推荐下一个技能，不要不问就运行
5. **适应** — 如果用户的情况不适合模板，倾听并调整
