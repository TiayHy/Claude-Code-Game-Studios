---
name: design-system
description: "引导式、逐节 GDD 撰写，针对单个游戏系统。从现有文档收集上下文、协作完成每个必需部分、交叉引用依赖关系，并增量写入文件。"
argument-hint: "<system-name> [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Task, AskUserQuestion, TodoWrite
---

当此技能被调用时：

## 1. 解析参数并验证

解析审查模式（一次性，本轮所有 gate spawn 共用）：
1. 如果传入了 `--review [full|lean|solo]` → 使用该值
2. 否则读取 `production/review-mode.txt` → 使用该值
3. 否则 → 默认为 `lean`

参见 `.claude/docs/director-gates.md` 完整检查模式。

系统名称或 retrofit 路径是**必需的**。如果缺失：

1. 检查 `design/gdd/systems-index.md` 是否存在。
2. 如果存在：读取它，找到优先级最高且状态为"Not Started"或等效状态的系统，然后使用 `AskUserQuestion`：
   - Prompt："你设计顺序中的下一个系统是 **[system-name]**（[priority] | [layer]）。开始设计它？"
   - Options：`[A] 是 — 设计 [system-name]` / `[B] 选一个不同的系统` / `[C] 到此为止`
   - 如果 [A]：用该系统名称继续。如果 [B]：问想设计哪个系统（纯文本）。如果 [C]：退出。
3. 如果没有系统索引，失败并提示：
   > "用法：`/design-system <system-name>` — 例如 `/design-system movement`
   > 或填充现有 GDD 中的空白：`/design-system retrofit design/gdd/[system-name].md`
   > 未找到系统索引。先运行 `/map-systems` 来映射你的系统并获取设计顺序。"

**检测 retrofit 模式：**
如果参数以 `retrofit` 开头或参数是指向 `design/gdd/` 中现有 `.md` 文件的路径，进入 **retrofit 模式**：

1. 读取现有 GDD 文件。
2. 识别 8 个必需部分中哪些已存在（扫描章节标题）。
   必需部分：Overview、Player Fantasy、Detailed Design/Rules、Formulas、Edge Cases、Dependencies、Tuning Knobs、Acceptance Criteria。
3. 识别哪些部分只包含占位符文本（`[To be designed]` 或等效——空白的、单独一行的、或明显不完整的）。
4. 在做任何事情之前向用户呈现：
   ```
   ## Retrofit：[System Name]
   文件：design/gdd/[filename].md

   已写部分（不会修改）：
   ✓ [section name]
   ✓ [section name]

   缺失或不完整部分（将被撰写）：
   ✗ [section name] — 缺失
   ✗ [section name] — 仅占位符
   ```
5. 问："我可以填充这 [N] 个缺失部分吗？我不会修改任何现有内容。"
6. 如果是：正常进入**阶段 2（收集上下文）**，但在**阶段 3** 跳过创建骨架（文件已存在）并在**阶段 4** 跳过已完成的部分。只对缺失/不完整部分运行章节循环。
7. **永远不要覆盖现有部分内容。** 使用 Edit 工具只替换 `[To be designed]` 占位符或空的部分主体。

如果不是 retrofit 模式，将系统名称规范化为 kebab-case 作为文件名（例如 "combat system" 变为 `combat-system`）。

---

## 2. 收集上下文（读取阶段）

在询问用户任何问题之前，读取所有相关上下文。这是此技能相对于临时设计的主要优势——它是有备而来的。

### 2a：必需读取

- **游戏概念**：读取 `design/gdd/game-concept.md`——如果缺失则失败：
  > "未找到游戏概念。先运行 `/brainstorm`。"
- **系统索引**：读取 `design/gdd/systems-index.md`——如果缺失则失败：
  > "未找到系统索引。先运行 `/map-systems` 来映射你的系统。"
- **目标系统**：在索引中找到该系统。如果未列出，警告：
  > "[system-name] 不在系统索引中。你想添加它，还是作为索引外系统来设计？"
- **实体注册表**：如果 `design/registry/entities.yaml` 存在，读取它。提取此系统引用或相关的所有条目（grep `referenced_by.*[system-name]` 和 `source.*[system-name]`）。将这些作为**已知事实**保存在上下文中——其他 GDD 已建立且此 GDD 不得矛盾的值。
- **反思日志**：如果 `docs/consistency-failures.md` 存在，读取它。提取其 Domain 匹配此系统类别的条目。这些是反复出现的冲突模式——在阶段 2d 上下文摘要中的"过去失败模式"下呈现，使用户知道在此领域过去发生过什么错误。

### 2b：依赖读取

从系统索引中识别：
- **上游依赖**：此系统所依赖的系统。读取其 GDD（如果存在）（这些包含此系统必须遵守的决策）。
- **下游依赖方**：依赖此系统的系统。读取其 GDD（如果存在）（这些包含此系统必须满足的期望）。

对于每个存在的依赖 GDD，提取并保存在上下文中：
- 关键接口（系统之间流动的数据）
- 引用此系统输出的公式
- 假设此系统行为的边界情况
- 馈入此系统的调优旋钮

### 2c：可选读取

- **游戏支柱**：如果 `design/gdd/game-pillars.md` 存在，读取它
- **现有 GDD**：如果 `design/gdd/[system-name].md` 存在，读取它（继续而非从头开始）
- **相关 GDD**：Glob `design/gdd/*.md` 并读取任何主题相关的（例如，如果设计的系统与另一个在范围上有重叠，即使它不是正式依赖也读取相关 GDD）

### 2d：呈现上下文摘要

在开始设计工作之前，向用户呈现简要摘要：

> **正在设计：[System Name]**
> - 优先级：[来自索引] | 层级：[来自索引]
> - 依赖：[列表，注明哪些有 GDD 哪些未设计]
> - 被依赖：[列表，注明哪些有 GDD 哪些未设计]
> - 须遵守的现有决策：[来自依赖 GDD 的关键约束]
> - 支柱对齐：[此系统主要服务的支柱]
> - **已知的跨系统事实（来自注册表）：**
>   - [entity_name]：[attribute]=[value]，[attribute]=[value]（由 [source GDD] 拥有）
>   - [item_name]：[attribute]=[value]，[attribute]=[value]（由 [source GDD] 拥有）
>   - [formula_name]：变量=[列表]，输出=[最小–最大]（由 [source GDD] 拥有）
>   - [constant_name]：[value] [unit]（由 [source GDD] 拥有）
>   *（这些值是锁定的——如果此 GDD 需要不同的值，在写入之前先暴露冲突。不要静默使用不同的数字。）*
>
> 如果没有相关注册表条目：省略"Known cross-system facts"部分。

如果有上游依赖未设计，警告：
> "[dependency] 还没有 GDD。我们需要对其接口做假设。先设计它，或者我们可以定义预期的契约并标记为暂定。"

### 2e：技术可行性预检

在要求用户开始设计之前，加载引擎上下文并呈现将塑造设计的任何约束或知识差距。

**步骤 1——确定此系统的引擎领域：**
将系统类别（来自 systems-index.md）映射到引擎领域：

| 系统类别 | 引擎领域 |
|----------------|--------------|
| Combat, physics, collision | Physics |
| Rendering, visual effects, shaders | Rendering |
| UI, HUD, menus | UI |
| Audio, sound, music | Audio |
| AI, pathfinding, behavior trees | Navigation / Scripting |
| Animation, IK, rigs | Animation |
| Networking, multiplayer, sync | Networking |
| Input, controls, keybinding | Input |
| Save/load, persistence, data | Core |
| Dialogue, quests, narrative | Scripting |

**步骤 2——读取引擎上下文（如果有）：**
- 读取 `.claude/docs/technical-preferences.md` 以识别引擎和版本
- 如果引擎已配置，读取 `docs/engine-reference/[engine]/VERSION.md`
- 如果 `docs/engine-reference/[engine]/modules/[domain].md` 存在，读取它
- 读取 `docs/engine-reference/[engine]/breaking-changes.md` 中与领域相关的条目
- Glob `docs/architecture/adr-*.md` 并读取其领域匹配（检查引擎兼容性表中的"Domain"字段）的任何 ADR

**步骤 3——呈现可行性简报：**

如果引擎参考文档存在，在开始设计之前呈现：

```
## 技术可行性简报：[System Name]
引擎：[name + version]
领域：[domain]

### 已知的引擎能力（已为 [version] 验证）
- [与此系统相关的能力]
- [能力 2]

### 将塑造此设计的引擎约束
- [来自 engine-reference 或现有 ADR 的约束]

### 知识差距（承诺之前请验证）
- [此设计可能依赖的 post-cutoff 功能——标记为 HIGH/MEDIUM 风险]

### 约束此系统的现有 ADR
- ADR-XXXX：[决策摘要]——意味着 [对此 GDD 的影响]
  （或"尚无"）
```

如果没有引擎参考文档（引擎尚未配置），显示简短备注：
> "尚未配置引擎——跳过技术可行性检查。如果尚未运行，在进入架构之前运行 `/setup-engine`。"

**步骤 4——继续之前询问：**

使用 `AskUserQuestion`：
- "在我们开始之前有任何约束要添加，还是按所述进行？"
  - Options："按所述进行"、"先添加一个约束"、"我需要查看引擎文档——暂停这里"

---

使用 `AskUserQuestion`：
- "准备好开始设计 [system-name] 了吗？"
  - Options："是，开始吧"、"先给我看更多上下文"、"先设计一个依赖"

---

## 3. 创建文件骨架

用户确认后，**立即**创建带有空章节头的 GDD 文件。这确保增量写入有目标。

使用 `.claude/docs/templates/game-design-document.md` 中的模板结构：

```markdown
# [System Name]

> **状态**：设计中
> **作者**：[user + agents]
> **最后更新**：[今天的日期]
> **实现支柱**：[来自上下文]

## Overview

[To be designed]

## Player Fantasy

[To be designed]

## Detailed Design

### Core Rules

[To be designed]

### States and Transitions

[To be designed]

### Interactions with Other Systems

[To be designed]

## Formulas

[To be designed]

## Edge Cases

[To be designed]

## Dependencies

[To be designed]

## Tuning Knobs

[To be designed]

## Visual/Audio Requirements

[To be designed]

## UI Requirements

[To be designed]

## Acceptance Criteria

[To be designed]

## Open Questions

[To be designed]
```

问："我可以创建骨架文件到 `design/gdd/[system-name].md` 吗？"

写入后，更新 `production/session-state/active.md`：
- 使用 Glob 检查文件是否存在。
- 如果文件**不存在**：使用 **Write** 工具创建它。永远不要尝试编辑可能不存在的文件。
- 如果文件**已存在**：使用 **Edit** 工具更新相关字段。

文件内容：
- Task: Designing [system-name] GDD
- Current section: Starting (skeleton created)
- File: design/gdd/[system-name].md

---

## 4. 逐节设计

按顺序遍历每个部分。对于**每个部分**，遵循此循环：

### 部分循环

```
上下文  ->  问题  ->  选项  ->  决策  ->  草稿  ->  批准  ->  写入
```

1. **上下文**：说明此部分需要包含什么，并呈现任何约束它的来自依赖 GDD 的相关决策。

2. **问题**：提出针对此部分的澄清问题。使用 `AskUserQuestion` 处理约束性问题，开放性探索用对话文本。

3. **选项**：如果部分涉及设计选择（不只是文档），呈现 2-4 种方法及优缺点。在对话文本中解释推理，然后使用 `AskUserQuestion` 捕获决策。

4. **决策**：用户选择方法或提供自定义方向。

5. **草稿**：在对话文本中撰写部分内容供审查。标记任何关于未设计依赖的暂定假设。

6. **批准**：紧接在草稿之后——在同一回复中——使用 `AskUserQuestion`。**永远不要用纯文本。永远不要跳过此步骤。**
   - Prompt："批准 [Section Name] 部分？"
   - Options：`[A] 批准——写入文件` / `[B] 做修改——描述要修复的内容` / `[C] 重新开始`

   **草稿和批准部件必须出现在同一回复中。如果草稿出现而没有部件，用户会面临空白提示而没有前进路径——这是协议违规。**

7. **写入**：使用 Edit 工具用批准的内容替换占位符。
   **关键**：始终在 `old_string` 中包含章节标题以确保唯一性——不要单独匹配 `[To be designed]`，因为多个部分使用相同占位符且 Edit 工具需要唯一匹配。使用此模式：
   ```
   old_string: "## [Section Name]\n\n[To be designed]"
   new_string: "## [Section Name]\n\n[approved content]"
   ```
   确认写入。

8. **注册表冲突检查**（仅 C 和 D 部分——详细设计和公式）：
   写入后，扫描部分内容中出现在注册表中的实体名称、物品名称、公式名称和数字常量。对于每个匹配项：
   - 将刚刚写入的值与注册表条目进行比较。
   - 如果不同：**立即暴露冲突**，再开始下一部分之前。不要静默继续。
     > "注册表冲突：[name] 在 [source GDD] 中注册为 [registry_value]。
     > 此部分刚写了 [new_value]。哪个是正确的？"
   - 如果是新的（不在注册表中）：标记为注册表注册候选（将在阶段 5 处理）。

写入每个部分后，用完成的章节名称更新 `production/session-state/active.md`。使用 Glob 检查文件是否存在——如果不存在用 Write 创建，如果存在用 Edit 更新。

### 各部分指导

每个部分都有独特的设计考虑，可能受益于专家 agent：

---

### 部分 A：Overview

**目标**：一陌生人读完能理解的段落。

**构建 widget 前先推导推荐选项**：从系统索引中读取系统类别和层级（已在阶段 2 上下文中），然后为每个标签确定推荐选项：
- **Framing 标签**：Foundation/Infrastructure 层 → 推荐 `[A]`。面向玩家的类别（Combat, UI, Dialogue, Character, Animation, Visual Effects, Audio）→ 推荐 `[C] Both`。
- **ADR ref 标签**：Glob `docs/architecture/adr-*.md` 并在任意 ADR 的 GDD Requirements 部分中 grep 系统名称。如果找到匹配的 ADR → 推荐 `[A] Yes — cite the ADR`。如果没找到 → 推荐 `[B] No`。
- **Fantasy 标签**：Foundation/Infrastructure 层 → 推荐 `[B] No`。所有其他类别 → 推荐 `[A] Yes`。

在适当的选项文本后附加`（推荐）`。

** Framing 问题（起草前问）**：使用带多标签的 `AskUserQuestion`：
- 标签 "Framing"——"概述应该如何框架化这个系统？"
  选项：`[A] 作为数据/基础设施层（技术框架）` / `[B] 通过其面向玩家的效果（设计框架）` / `[C] 两者——描述数据层及其对玩家的影响`
- 标签 "ADR ref"——"概述应该引用此系统的现有 ADR 吗？"
  选项：`[A] 是——引用 ADR 获取实现细节` / `[B] 否——保持 GDD 在纯粹设计层面`
- 标签 "Fantasy"——"这个系统有值得陈述的玩家幻想吗？"
  选项：`[A] 是——玩家直接感受它` / `[B] 否——纯粹的基础设施，玩家感受它所启用的`

用用户的答案来塑造草稿。不要自己回答这些问题并自动起草。

**要问的问题**：
- 用一句话说这是什么系统？
- 玩家如何与它互动？（主动/被动/自动）
- 这个系统为什么存在——没有它游戏会失去什么？

**交叉引用**：检查描述是否与系统索引中的描述一致。标记差异。

**设计与实现边界**：Overview 问题必须保持在行为层面——系统*做什么*，而非*如何构建*。如果在 Overview 期间出现实现问题（例如"这应该使用 Autoload 单例还是信号总线？"），将它们标记为"→ 成为 ADR"并继续。实现模式属于 `/architecture-decision`，而非 GDD。GDD 描述行为；ADR 描述用于实现行为的技术方法。

---

### 部分 B：Player Fantasy

**目标**：情感目标——玩家应该*感受到*什么。

**构建 widget 前先推导推荐选项**：从阶段 2 上下文中读取系统类别和层级：
- 面向玩家的类别（Combat, UI, Dialogue, Character, Animation, Audio, Level/World）→ 推荐 `[A] Direct`
- Foundation/Infrastructure 层 → 推荐 `[B] Indirect`
- 混合类别（Camera/input, Economy, AI with visible player effects）→ 推荐 `[C] Both`

在适当的选项文本后附加`（推荐）`。

** Framing 问题（起草前问）**：使用 `AskUserQuestion`：
- Prompt："这是玩家直接参与的系统，还是他们间接体验的基础设施？"
- Options：`[A] 直接——玩家主动使用或感受这个系统` / `[B] 间接——玩家体验效果，不是系统本身` / `[C] 两者——有直接交互层和其下的基础设施`

用答案适当框架化 Player Fantasy 部分。不要假设答案。

**要问的问题**：
- 这个系统服务的情感或权力幻想是什么？
- 哪些参考游戏精确地把握了这种感受？是什么具体创造了它？
- 这是"你热爱参与的系统"还是"你没注意到的基础设施"？

**交叉引用**：必须与游戏支柱对齐。如果系统服务某个支柱，引用相关支柱文本。

**Agent 委托（强制）**：在给出框架答案之后、起草之前，通过 Task 生成 `creative-director`：
- 提供：系统名称、框架答案（direct/indirect/both）、游戏支柱、用户提到的任何参考游戏、游戏概念摘要
- 问："为这个系统塑造 Player Fantasy。它应该服务什么情感或权力幻想？我们应该锚定哪个玩家时刻？什么语气和语言适合游戏已建立的感觉？要具体——给我 2-3 个候选框架。"
- 收集 creative-director 的框架并与草稿一起呈现给用户。

**在先咨询 `creative-director` 之前不要起草部分 B。** 框架答案告诉我们它*是什么类型*的幻想；creative-director 塑造它*如何被描述*——语气、语言、要锚定的特定玩家时刻。

---

### 部分 C：Detailed Design（Core Rules, States, Interactions）

**目标**：程序员无需提问即可实现的无歧义规范。

这是通常最大的部分。分解为子部分：

1. **Core Rules**：基本机制。对于顺序过程使用编号规则，属性用项目符号。
2. **States and Transitions**：如果系统有状态，映射每个状态和每个有效转换。使用表格。
3. **Interactions with Other Systems**：对于每个依赖（上游和下游），指定什么数据流入、什么流出、谁拥有接口。

**要问的问题**：
- 典型使用这个系统的流程，走一遍
- 玩家面临哪些决策点？
- 玩家不能做什么？（约束和能力同样重要）

**Agent 委托（强制）**：在起草部分 C 之前，通过 Task 并行生成专家 agent：
- 在路由表中查找系统类别（本技能第 6 节）
- 生成列出的主要 agent 和支持 agent
- 向每个 agent 提供：系统名称、游戏概念摘要、支柱集、依赖 GDD 摘要、正在处理的具体部分
- 在起草前收集他们的发现
- 通过 `AskUserQuestion` 向用户呈现 agent 之间的任何分歧
- 仅在收到专家输入后起草

**在没有先咨询适当专家的情况下不要起草部分 C。** `systems-designer` 审查规则和机制会捕捉主会话无法发现的设计漏洞。

**交叉引用**：对于列出的每个交互，验证它是否与依赖 GDD 的指定一致。如果依赖定义了此系统期望的不同值或公式，标记冲突。

---

### 部分 D：Formulas

**目标**：每个数学公式，包含变量定义、范围说明和边界情况说明。

**完成引导——始终以这个确切结构开始每个公式：**

```
[formula_name] 公式定义如下：

`[formula_name] = [expression]`

**变量：**
| 变量 | 符号 | 类型 | 范围 | 描述 |
|----------|--------|------|-------|-------------|
| [name] | [sym] | float/int | [min–max] | [它代表什么] |

**输出范围：** 正常游戏下达 [min] 到 [max]；极端情况下 [行为]
**示例：** [带实际数字的工作示例]
```

不要写 `[Formula TBD]` 或用散文描述公式而没有变量表。没有定义变量的公式无法实现而不靠猜测。

**要问的问题**：
- 这个系统执行哪些核心计算？
- 缩放应该是线性的、对数的还是分段的？
- 早期/中期/后期游戏的输出范围应该是什么？

**Agent 委托（强制）**：在任何公式或平衡值之前，通过 Task 并行生成专家 agent：
- **始终生成 `systems-designer`**：提供部分 C 的 Core Rules、用户的调优目标、来自依赖 GDD 的平衡上下文。让他们提出带变量表和输出范围的公式。
- **对于经济/成本系统，也生成 `economy-designer`**：提供放置成本、升级成本意图和进度目标。让他们验证成本曲线和比率。
- 通过 `AskUserQuestion` 向用户呈现专家的建议供审查
- 用户决定；主会话写入文件
- **没有专家输入不要编造公式值或平衡数字。** 没有平衡设计专业知识的用户无法评估原始数字——他们需要专家的推理。

**交叉引用**：如果依赖 GDD 定义了输出馈入此系统的公式，明确引用它。不要重新发明——连接。

---

### 部分 E：Edge Cases

**目标**：明确处理异常情况，以免它们成为 bug。

**完成引导——将每个边界情况格式化为：**
- **如果 [condition]**：[确切结果]。[如果非显而易见则附上理由]

示例（将术语适应游戏的领域）：
- **如果 [resource] 在 [protective condition] 激活时达到 0**：保持最小值直到条件结束，然后应用后果。
- **如果两个 [triggers/events] 同时触发**：按 [定义优先级顺序] 解决；平局使用 [定义的平局决胜规则]。

不要写模糊的条目如"适当处理"——每个必须命名确切条件和确切解决方案。没有解决方案的边界情况是一个开放的 设计问题，不是规范。

**要问的问题**：
- 在零时、最大时、超出范围时会发生什么？
- 当两条规则同时适用时会发生什么？
- 如果玩家发现意外的交互会怎样？（识别退化策略）

**Agent 委托（强制）**：在最终确定边界情况之前，通过 Task 生成 `systems-designer`。提供：完成的 C 和 D 部分，让他们从公式和规则空间中识别主会话可能遗漏的边界情况。对于叙事系统，也生成 `narrative-director`。向用户呈现他们的发现并问哪些要包含。

**交叉引用**：根据依赖 GDD 检查边界情况。如果依赖定义了这个系统可能违反的 floor、cap 或解决规则，标记它。

---

### 部分 F：Dependencies

**目标**：用方向和性质映射每个系统连接。

此部分从上下文收集阶段预填充了部分内容。呈现来自系统索引的已知依赖并问：
- 我遗漏了依赖吗？
- 对于每个依赖，特定的数据接口是什么？
- 哪些依赖是硬性的（系统没有它无法运行）vs. 软性的（有它会增强但没有也能工作）？

**交叉引用**：此部分必须双向一致。如果此系统列出"依赖 Combat"，则 Combat GDD 应该列出"被 [此系统] 依赖"。标记任何单向依赖以进行更正。

---

### 部分 G：Tuning Knobs

**目标**：每个设计师可调整的值，包含安全范围和极端行为。

**要问的问题**：
- 哪些值应该让设计师无需更改代码就能调整？
- 对于每个旋钮，设置太高会破坏什么？太低呢？
- 哪些旋钮相互影响？（改 A 使 B 无关紧要）

**Agent 委托**：如果公式复杂，委托 `systems-designer` 从公式变量中推导调优旋钮。

**交叉引用**：如果依赖 GDD 列出了影响此系统的调优旋钮，在此引用它们。不要创建重复旋钮——指向权威来源。

---

### 部分 H：Acceptance Criteria

**目标**：可测试的条件，证明系统按设计工作。

**完成引导——将每个标准格式化为 Given-When-Then：**
- **GIVEN** [初始状态]**，WHEN** [动作或触发器]**，THEN** [可衡量结果]

示例（将术语适应游戏的领域）：
- **GIVEN** [初始状态]**，WHEN** [玩家动作或系统触发器]**，THEN** [特定可衡量结果]。
- **GIVEN** [约束激活时]**，WHEN** [玩家尝试动作]**，THEN** [显示的反馈和动作结果]。

至少包含：每个核心规则（来自 C）一个标准，每个公式（来自 D）一个标准。不要写"系统按设计工作"——每个标准必须可由 QA 测试人员独立验证，无需阅读 GDD。

**Agent 委托（强制）**：在最终确定验收标准之前，通过 Task 生成 `qa-lead`。提供：完成的 GDD 部分 C、D、E，让他们验证标准是否可独立测试并覆盖所有核心规则和公式。向用户呈现任何差距或不可测试的标准。

**要问的问题**：
- 证明这有效的最小测试集是什么？
- 这个系统的性能预算是多少？（帧时间、内存）
- QA 测试人员首先检查什么？

**交叉引用**：包含验证跨系统交互工作的标准，而不只是此系统本身。

---

### 可选部分：Visual/Audio、UI Requirements、Open Questions

这些部分包含在模板中。Visual/Audio 对于视觉系统类别是**必需的**——不是可选的。在询问之前确定要求级别：

**Visual/Audio 是必需的（强制——不要提供跳过）适用于以下系统类别：**
- Combat, damage, health
- UI systems (HUD, menus)
- Animation, character movement
- Visual effects, particles, shaders
- Character systems
- Dialogue, quests, lore
- Level/world systems

对于必需的系统：在起草此部分之前**生成 `art-director` via Task**。提供：系统名称、游戏概念、游戏支柱、艺术圣经第 1-4 节（如果存在）。让他们指定：(1) 此系统事件的 VFX 和视觉反馈要求，(2) 任何动画或视觉风格约束，(3) 哪些艺术圣经原则最直接适用于此系统。呈现他们的输出；不要将此部分留为 `[To be designed]`。

对于**所有其他系统类别**（Foundation/Infrastructure, Economy, AI/pathfinding, Camera/input），在必需部分之后提供可选部分：

使用 `AskUserQuestion`：
- "8 个必需部分已完成。你也想定义 Visual/Audio 要求、UI 要求，还是捕获开放问题？"
  - Options："是，三个都要"、"只要开放问题"、"跳过——我稍后添加"

对于**Visual/Audio**（非必需系统）：如果需要细节，与 `art-director` 和 `audio-director` 协调。在 GDD 阶段通常简短说明即可。

> **资产规格标志**：Visual/Audio 部分写入了真实内容后，输出此通知：
> "📌 **Asset Spec** — Visual/Audio 要求已定义。艺术圣经批准后，运行 `/asset-spec system:[system-name]` 从此部分产出每个资产的视觉描述、尺寸和生成提示。"

对于**UI Requirements**：对于复杂 UI 系统，与 `ux-designer` 协调。写入此部分后，检查它是否包含真实内容（不只是 `[To be designed]` 或说明此系统没有 UI 的备注）。如果它确实有真实的 UI 要求，立即输出此标志：

> **📌 UX Flag — [System Name]**：此系统有 UI 要求。在阶段 4（前期制作）中，为此系统贡献的每个屏幕或 HUD 元素运行 `/ux-design` 创建 UX 规格，**在撰写史诗之前**。引用 UI 的故事应引用 `design/ux/[screen].md`，而非直接引用 GDD。
>
> 如果更新了系统索引中的此系统，请注明。

对于**Open Questions**：捕获设计期间出现但未完全解决的任何内容。每个问题应该有所有者和目标解决日期。

---

## 5. 后期设计验证

所有部分写完后：

### 5a：自检

从文件回读完整 GDD（不是从对话记忆中——文件是权威来源）。验证：
- 所有 8 个必需部分都有真实内容（非占位符）
- 公式引用了定义的变量
- 边界情况有解决方案
- 依赖列出了接口
- 验收标准可测试

### 5a-bis：Creative Director 支柱审查

**审查模式检查**——在生成 CD-GDD-ALIGN 之前应用：
- `solo` → 跳过。注明："CD-GDD-ALIGN 跳过 — Solo 模式。"进入步骤 5b。
- `lean` → 跳过（不是 PHASE-GATE）。注明："CD-GDD-ALIGN 跳过 — Lean 模式。"进入步骤 5b。
- `full` → 正常生成。

在最终确定 GDD 之前，通过 Task 使用 gate **CD-GDD-ALIGN**（`.claude/docs/director-gates.md`）生成 `creative-director`。

传入：完成的 GDD 文件路径、游戏支柱（来自 `design/gdd/game-concept.md` 或 `design/gdd/game-pillars.md`）、目标 MDA 美学。

按照 `director-gates.md` 中的标准规则处理裁决。解决后，在 GDD 状态头中记录裁决：
`> **Creative Director Review (CD-GDD-ALIGN)**：APPROVED [date] / CONCERNS (accepted) [date] / REVISED [date]`

---

### 5b：更新实体注册表

扫描完成的 GDD 中应注册的跨系统事实：
- 具有属性或掉落的命名实体（敌人、NPC、boss）
- 具有值、权重或类别的命名物品
- 具有定义变量和输出范围的命名公式
- 在多个地方按值引用的命名常量

对于每个候选，检查它是否已存在于 `design/registry/entities.yaml` 中：
```
Grep pattern="  - name: [candidate_name]" path="design/registry/entities.yaml"
```

呈现摘要：
```
来自此 GDD 的注册表候选：
  新建（尚未注册）：
    - [entity_name] [entity]：[attribute]=[value]，[attribute]=[value]
    - [item_name] [item]：[attribute]=[value]，[attribute]=[value]
    - [formula_name] [formula]：变量=[列表]，输出=[最小–最大]
  已注册（referenced_by 将更新）：
    - [constant_name] [constant]：值=[N] ← 与注册表匹配 ✅
```

问："我可以更新 `design/registry/entities.yaml` 添加这 [N] 个新条目并更新现有条目的 `referenced_by` 吗？"

如果是：追加新条目并更新 `referenced_by` 数组。永远不要在先暴露冲突的情况下修改现有的 `value` / 属性字段。

### 5c：提供设计审查

呈现完成摘要：

> **GDD 完成：[System Name]**
> - 已撰写部分：[列表]
> - 暂定假设：[关于未设计依赖的列表]
> - 发现跨系统冲突：[列表或"无"]

> **要验证此 GDD，打开新的 Claude Code 会话并运行：**
> `/design-review design/gdd/[system-name].md`
>
> **不要在 `/design-system` 的同一会话中运行 `/design-review`。** 审查 agent 必须独立于创作上下文。在此运行会继承完整的设计历史，使独立批评不可能。

**永远不要在这里内联提供运行 `/design-review`**。始终引导用户到新窗口。

### 5d：更新系统索引

GDD 完成后（可选审查后）：

- 读取系统索引
- 更新目标系统的行：
  - 如果运行了 design-review 且裁决为 APPROVED：状态 → "Approved"
  - 如果运行了 design-review 且裁决为 NEEDS REVISION：状态 → "In Review"
  - 如果跳过了审查：状态 → "Designed"（待审查）
  - 如果用户选择"我先自己审查"：状态 → "Designed"
  - 设计文档：链接到 `design/gdd/[system-name].md`
- 更新进度追踪计数

问："我可以更新 `design/gdd/systems-index.md` 中的系统索引吗？"

### 5d：更新会话状态

用以下内容更新 `production/session-state/active.md`：
- Task：[system-name] GDD
- Status：Complete（或如果运行了 design-review 则为 In Review）
- File：design/gdd/[system-name].md
- Sections：全部 8 个已写
- Next：[从设计顺序中建议下一个系统]

### 5e：建议下一步

使用 `AskUserQuestion`：
- "接下来是什么？"
  - Options：
    - "运行 `/consistency-check` — 验证此 GDD 的值与其他 GDD 没有冲突（在设计下一个系统之前推荐）"
    - "设计下一个系统（[next-in-order]）"——如果有未设计的系统剩余
    - "修复审查发现"——如果 design-review 标记了问题
    - "本次会话到此为止"
    - "运行 `/gate-check`"——如果有足够的 MVP 系统已设计

---

## 6. 专家 Agent 路由

此技能委托专家 agent 提供领域专业知识。主会话编排整体流程；agent 提供专家内容。

| 系统类别 | 主要 Agent | 支持 Agent |
|----------------|---------------|---------------------|
| **Foundation/Infrastructure**（事件总线、存档/加载、场景管理、服务定位器） | `systems-designer` | `gameplay-programmer`（可行性）、`engine-programmer`（引擎集成） |
| Combat, damage, health | `game-designer` | `systems-designer`（公式）、`ai-programmer`（敌人 AI）、`art-director`（命中反馈视觉方向、VFX 意图） |
| Economy, loot, crafting | `economy-designer` | `systems-designer`（曲线）、`game-designer`（循环） |
| Progression, XP, skills | `game-designer` | `systems-designer`（曲线）、`economy-designer`（消耗） |
| Dialogue, quests, lore | `game-designer` | `narrative-director`（故事）、`writer`（内容）、`art-director`（角色视觉档案、电影感基调） |
| UI systems (HUD, menus) | `game-designer` | `ux-designer`（流程）、`ui-programmer`（可行性）、`art-director`（视觉风格方向）、`technical-artist`（渲染/着色器约束） |
| Audio systems | `game-designer` | `audio-director`（方向）、`sound-designer`（规格） |
| AI, pathfinding, behavior | `game-designer` | `ai-programmer`（实现）、`systems-designer`（评分） |
| Level/world systems | `game-designer` | `level-designer`（空间）、`world-builder`（世界观） |
| Camera, input, controls | `game-designer` | `ux-designer`（手感）、`gameplay-programmer`（可行性） |
| Animation, character movement | `game-designer` | `art-director`（动画风格、姿态语言）、`technical-artist`（绑定/混合约束）、`gameplay-programmer`（手感） |
| Visual effects, particles, shaders | `game-designer` | `art-director`（VFX 视觉方向）、`technical-artist`（性能预算、着色器复杂度）、`systems-designer`（触发器/状态集成） |
| Character systems (stats, archetypes) | `game-designer` | `art-director`（角色视觉原型）、`narrative-director`（角色弧线对齐）、`systems-designer`（属性公式） |

**通过 Task 工具委托时**：
- 提供：系统名称、游戏概念摘要、依赖 GDD 摘要、正在处理的具体部分、以及需要专家输入的问题
- agent 将分析/提案返回主会话
- 主会话通过 `AskUserQuestion` 向用户呈现 agent 的输出
- 用户决定；主会话写入文件
- agent 不直接写入文件——主会话拥有所有文件写入权限

---

## 7. 恢复与继续

如果会话中断（压缩、崩溃、新会话）：

1. 读取 `production/session-state/active.md`——它记录当前系统和哪些部分已完成
2. 读取 `design/gdd/[system-name].md`——有真实内容的部分已完成；仍有 `[To be designed]` 的部分需要工作
3. 从下一个未完成部分继续——无需重新讨论已完成的部分

这就是增量写作的意义：每个批准的部分都能在任何中断中存活。

---

## 协作协议

此技能在每一步都遵循协作设计原则：

1. **问题 -> 选项 -> 决策 -> 草稿 -> 批准** 用于每个部分
2. **在每个决策点使用 AskUserQuestion**（解释 -> 捕获模式）：
   - 阶段 2："准备好开始了，还是需要更多上下文？"
   - 阶段 3："我可以创建骨架吗？"
   - 阶段 4（每个部分）：设计问题、方法选项、草稿批准
   - 阶段 5："运行设计审查？更新系统索引？接下来是什么？"
3. **"我可以写入 [filepath] 吗？"** 在骨架前和每个部分写入前
4. **增量写入**：每个部分在批准后立即写入文件
5. **会话状态更新**：