# Claude Code Game Studios — 完整工作流指南

> **如何使用 Agent 架构从零到发布游戏。**
>
> 本指南带您了解使用 48-Agent 系统、68 个斜杠命令和 12 个自动化钩子的
> 游戏开发每个阶段。假设您已安装 Claude Code 并从项目根目录工作。
>
> 流程有 7 个阶段。每个阶段都有一个正式关卡（`/gate-check`），
> 必须通过才能进入下一阶段。权威的阶段顺序定义在
> `.claude/docs/workflow-catalog.yaml` 中，由 `/help` 读取。
>
> ---
>
> **本指南已翻译为中文。** 如有歧义，请以英文原版为准。
>
> ---

## 目录

1. [快速开始](#快速开始)
2. [阶段 1：概念](#阶段-1-概念)
3. [阶段 2：系统设计](#阶段-2-系统设计)
4. [阶段 3：技术准备](#阶段-3-技术准备)
5. [阶段 4：预生产](#阶段-4-预生产)
6. [阶段 5：生产](#阶段-5-生产)
7. [阶段 6：打磨](#阶段-6-打磨)
8. [阶段 7：发布](#阶段-7-发布)
9. [跨领域关注点](#跨领域关注点)
10. [附录 A：Agent 速查](#附录-a-agent-速查)
11. [附录 B：斜杠命令速查](#附录-b-斜杠命令速查)
12. [附录 C：常见工作流](#附录-c-常见工作流)

---

## 快速开始

### 准备工作

在开始之前，请确保您拥有：

- **Claude Code** 已安装并正常运行
- **Git**（Windows 上用 Git Bash，Mac/Linux 用标准终端）
- **jq**（可选但推荐 — 钩子在缺少时回退到 `grep`）
- **Python 3**（可选 — 部分钩子用于 JSON 验证）

### 步骤 1：克隆并打开

```bash
git clone <repo-url> my-game
cd my-game
```

### 步骤 2：运行 /start

如果是首次会话：

```
/start
```

此引导入职会询问您所在阶段并路由到正确的流程：

- **路径 A** — 还没有想法：路由到 `/brainstorm`
- **路径 B** — 有模糊想法：路由到带种子的 `/brainstorm`
- **路径 C** — 已有清晰概念：路由到 `/setup-engine` 和 `/map-systems`
- **路径 D1** — 已有项目，产出少：正常流程
- **路径 D2** — 已有项目，存在 GDD/ADR：运行 `/project-stage-detect`
  然后运行 `/adopt` 进行棕地迁移

### 步骤 3：验证钩子是否正常工作

启动新的 Claude Code 会话。您应该看到
`session-start.sh` 钩子的输出：

```
=== Claude Code Game Studios -- Session Context ===
Branch: main
Recent commits:
 abc1234 Initial commit
===================================
```

如果看到此信息，说明钩子正常工作。如果没有，
请检查 `.claude/settings.json` 确保钩子路径对您的操作系统正确。

### 步骤 4：随时获取帮助

在任何时候运行：

```
/help
```

这会从 `production/stage.txt` 读取您当前的阶段，
检查存在的产出，并告诉您接下来该做什么。
它会区分 REQUIRED（必须）下一步和 OPTIONAL（可选）机会。

### 步骤 5：创建目录结构

目录在需要时创建。系统期望以下布局：

```
src/                  # 游戏源代码
  core/               # 引擎/框架代码
  gameplay/           # 游戏系统
  ai/                 # AI 系统
  networking/         # 多人游戏代码
  ui/                 # UI 代码
  tools/              # 开发工具
assets/               # 游戏资源
  art/                # 精灵、模型、纹理
  audio/              # 音乐、音效
  vfx/                # 粒子特效
  shaders/            # 着色器文件
  data/               # JSON 配置/平衡数据
design/               # 设计文档
  gdd/                # 游戏设计文档
  narrative/          # 故事、 lore、对话
  levels/             # 关卡设计文档
  balance/            # 平衡表格和数据
  ux/                 # UX 规格说明
docs/                 # 技术文档
  architecture/       # 架构决策记录
  api/                # API 文档
  postmortems/        # 项目复盘
tests/                # 测试套件
prototypes/           # 一次性原型
production/           # 迭代计划、里程碑、发布
  sprints/
  milestones/
  releases/
  epics/              # 史诗和故事文件（来自 /create-epics + /create-stories）
  playtests/          # 测试报告
  session-state/      # 临时会话状态（gitignored）
  session-logs/       # 会话审计跟踪（gitignored）
```

> **提示：** 第一天不需要所有这些。到达需要该阶段的目录时再创建。
> 重要的是在创建时遵循此结构，因为**规则系统**根据文件路径强制执行标准。
> `src/gameplay/` 中的代码应用 gameplay 规则，
> `src/ai/` 中的代码应用 AI 规则，以此类推。

---

## 阶段 1：概念

### 此阶段做什么

您从"没有想法"或"模糊想法"到一个结构化的游戏概念文档，
包含已定义的支柱和玩家旅程。这是要弄清楚**您要做什么**和**为什么**。

### 阶段 1 流程

```
/brainstorm  -->  game-concept.md  -->  /design-review  -->  /setup-engine
     |                                        |                    |
     v                                        v                    v
  10 个概念       含支柱、MDA、         概念文档验证         引擎固定在
  MDA 分析        核心循环、USP         验证                  technical-preferences.md
  玩家动机        分析                                                              |
                                                                   v
                                                             /map-systems
                                                                   |
                                                                   v
                                                            systems-index.md
                                                            （所有系统、依赖关系、
                                                             优先级层级）
```

### 步骤 1.1：用 /brainstorm 头脑风暴

这是您的起点。运行 brainstorm 技能：

```
/brainstorm
```

或带类型提示：

```
/brainstorm roguelike deckbuilder
```

**会发生什么：** brainstorm 技能引导您完成协作式 6 阶段构思过程，
使用专业工作室技术：

1. 询问您的兴趣、主题和约束
2. 生成 10 个概念种子及 MDA（机制、动态、美学）分析
3. 您选择 2-3 个最喜欢的进行深度分析
4. 执行玩家动机映射和受众定位
5. 您选择获胜概念
6. 将其正式化为 `design/gdd/game-concept.md`

概念文档包括：

- 电梯演讲（一句话）
- 核心幻想（玩家想象自己在做什么）
- MDA 分解
- 目标受众（Bartle 类型、人口统计）
- 核心循环图
- 独特销售主张
- 可比标题和差异化
- 游戏支柱（3-5 个不可妥协的设计价值观）
- 反支柱（游戏刻意避免的事物）

### 步骤 1.2：审核概念（可选但推荐）

```
/design-review design/gdd/game-concept.md
```

在继续之前验证结构和完整性。

### 步骤 1.3：选择引擎

```
/setup-engine
```

或指定引擎：

```
/setup-engine godot 4.6
```

**/setup-engine 的作用：**

- 在 `.claude/docs/technical-preferences.md` 中填充命名规范、
  性能预算和引擎特定默认值
- 检测知识差距（引擎版本新于 LLM 训练数据）并建议交叉参考 `docs/engine-reference/`
- 在 `docs/engine-reference/` 中创建版本固定参考文档

**为什么重要：** 一旦设置了引擎，系统就知道使用哪些引擎专家 Agent。
如果您选择 Godot，像 `godot-specialist`、`godot-gdscript-specialist`
和 `godot-shader-specialist` 这样的 Agent 将成为您的首选专家。

### 步骤 1.4：将概念分解为系统

在编写单独的 GDD 之前，枚举您的游戏需要的所有系统：

```
/map-systems
```

这会创建 `design/gdd/systems-index.md` — 一个主跟踪文档：

- 列出游戏需要的每个系统（战斗、移动、UI 等）
- 映射系统间的依赖关系
- 分配优先级层级（MVP、垂直切片、Alpha、完整愿景）
- 确定设计顺序（Foundation > Core > Feature > Presentation > Polish）

此步骤是进入阶段 2 之前的**必需要求**。
来自 155 个游戏项目复盘的研究证实，
跳过系统枚举会在生产阶段多花费 5-10 倍的成本。

### 阶段 1 关卡

```
/gate-check concept
```

**通过要求：**

- `technical-preferences.md` 中配置了引擎
- `design/gdd/game-concept.md` 存在且含支柱
- `design/gdd/systems-index.md` 存在且含依赖排序

**判定：** PASS / CONCERNS / FAIL。
CONCERNS 在有记录风险的情况下可以通过。
FAIL 阻塞进入。

---

## 阶段 2：系统设计

### 此阶段做什么

您创建定义游戏如何工作的所有设计文档。此阶段不写代码 — 纯粹是设计。
systems-index 中识别的每个系统都有自己的 GDD，
逐节编写，逐个审核，然后所有 GDD 进行交叉一致性检查。

### 阶段 2 流程

```
/map-systems next  -->  /design-system  -->  /design-review
       |                     |                     |
       v                     v                     v
  从 systems-index     逐节 GDD 创作         验证 8 个
  挑选下一个系统       （增量写入）          必填部分
                       创作指南              APPROVED/NEEDS REVISION
       |
       |  （对每个 MVP 系统重复）
       v
/review-all-gdds
       |
       v
  跨 GDD 一致性 + 设计理论审核
  PASS / CONCERNS / FAIL
```

### 步骤 2.1：编写系统 GDD

使用引导式工作流按依赖顺序设计每个系统：

```
/map-systems next
```

这会选择最高优先级的未设计系统并交给 `/design-system`，
引导您逐节创建其 GDD。

您也可以直接设计特定系统：

```
/design-system combat-system
```

**/design-system 的作用：**

1. 读取您的游戏概念、systems-index 和任何上游/下游 GDD
2. 运行技术可行性预检（领域映射 + 可行性简介）
3. 逐步引导您完成 8 个必填 GDD 部分
4. 每节遵循：背景 > 问题 > 选项 > 决策 > 草稿 > 批准 > 写入
5. 每节在批准后立即写入文件（崩溃可恢复）
6. 标记与现有已批准 GDD 的冲突
7. 按类别路由到专家 Agent（systems-designer 处理数学，
   economy-designer 处理经济，narrative-director 处理故事系统）

**8 个必填 GDD 部分：**

| # | 部分 | 内容要求 |
|---|---------|------------|
| 1 | **概述** | 系统的一段落总结 |
| 2 | **玩家幻想** | 玩家使用此系统时的想象/感受 |
| 3 | **详细规则** | 无歧义的机械规则 |
| 4 | **公式** | 每个计算，含变量定义和范围 |
| 5 | **边界情况** | 奇怪情况下会发生什么？明确解决。 |
| 6 | **依赖关系** | 连接到的其他系统（双向） |
| 7 | **调优旋钮** | 设计师可以安全更改的值，含安全范围 |
| 8 | **验收标准** | 如何测试它是否有效？具体、可衡量。 |

加上一个 **Game Feel 部分：** 感受参考、输入响应（毫秒/帧）、
动画感受目标（启动/活动/恢复）、冲击时刻、重量配置。

### 步骤 2.2：审核每个 GDD

在下一个系统开始前验证当前系统：

```
/design-review design/gdd/combat-system.md
```

检查所有 8 个部分的完整性、公式清晰度、边界情况解决、
双向依赖和可测试验收标准。

**判定：** APPROVED / NEEDS REVISION / MAJOR REVISION。
只有 APPROVED 的 GDD 才能继续。

### 步骤 2.3：无需完整 GDD 的小更改

对于调优更改、小添加或不值得完整 GDD 的调整：

```
/quick-design "add 10% damage bonus for flanking attacks"
```

这会在 `design/quick-specs/` 创建一个轻量级规格，
而不是完整的 8 节 GDD。用于调优、数字更改和小添加。

### 步骤 2.4：跨 GDD 一致性审核

所有 MVP 系统 GDD 单独获得 APPROVED 后：

```
/review-all-gdds
```

这会同时读取所有 GDD 并运行两个分析阶段：

**阶段 1 — 跨 GDD 一致性：**

- 依赖双向性（A 引用 B，B 引用 A 吗？）
- 系统间的规则矛盾
- 对重命名或删除系统的过时引用
- 所有权冲突（两个系统声称同一职责）
- 公式范围兼容性（系统 A 的输出适合系统 B 的输入吗？）
- 验收标准交叉检查

**阶段 2 — 设计理论（游戏设计整体性）：**

- 竞争性进度循环（两个系统争夺同一奖励空间吗？）
- 认知负荷（同时激活超过 4 个系统？）
- 主导策略（一种方法使所有其他方法无关吗？）
- 经济循环分析（来源和消耗平衡吗？）
- 系统间的难度曲线一致性
- 支柱对齐和反支柱违反
- 玩家幻想一致性

**输出：** `design/gdd/gdd-cross-review-[date].md` 含判定。

### 步骤 2.5：叙事设计（如适用）

如果您的游戏有故事、lore 或对话，这是构建它的时机：

1. **世界构建** — 使用 `world-builder` 定义派系、历史、
   地理和世界规则
2. **故事结构** — 使用 `narrative-director` 设计故事弧线、
   角色弧线和叙事节拍
3. **角色表** — 使用 `narrative-character-sheet.md` 模板

### 阶段 2 关卡

```
/gate-check systems-design
```

**通过要求：**

- `systems-index.md` 中所有 MVP 系统状态为 `Status: Approved`
- 每个 MVP 系统有已审核的 GDD
- 存在跨 GDD 审核报告（`design/gdd/gdd-cross-review-*.md`）
  判定为 PASS 或 CONCERNS（不是 FAIL）

---

## 阶段 3：技术准备

### 此阶段做什么

您做出关键技术决策，将其记录为架构决策记录（ADR），
通过审核进行验证，并生成控制清单，
为程序员提供平面化、可操作的规则。您还建立 UX 基础。

### 阶段 3 流程

```
/create-architecture  -->  /architecture-decision (x N)  -->  /architecture-review
        |                          |                                   |
        v                          v                                   v
  覆盖所有系统的主架构       每个决策的 ADR                验证完整性、
  文档在 docs/architecture/  在 docs/architecture/         依赖排序、
  architecture.md            adr-*.md                     引擎兼容性
                                                                      |
                                                                      v
                                                         /create-control-manifest
                                                                      |
                                                                      v
                                                         平面化程序员规则
                                                         docs/architecture/
                                                         control-manifest.md
        此阶段同时进行：
        -------------------
        /ux-design  -->  /ux-review
        无障碍需求文档
        交互模式库
```

### 步骤 3.1：主架构文档

```
/create-architecture
```

在 `docs/architecture/architecture.md` 创建涵盖所有系统的总体架构文档，
包含系统边界、数据流和集成点。

### 步骤 3.2：架构决策记录（ADR）

对于每个重大技术决策：

```
/architecture-decision "State Machine vs Behavior Tree for NPC AI"
```

**会发生什么：** 技能引导您创建含以下内容的 ADR：
- 背景和决策驱动因素
- 所有选项含优缺点和引擎兼容性
- 所选选项及理由
- 后果（正面、负面、风险）
- 依赖关系（取决于、启用、阻塞、排序说明）
- 已解决的 GDD 需求（通过 TR-ID 链接）

ADR 经历生命周期：Proposed > Accepted > Superseded/Deprecated。

**最低需要 3 个 Foundation 层 ADR** 才能通过关卡检查。

**改造现有 ADR：** 如果您已有棕地项目的 ADR：

```
/architecture-decision retrofit docs/architecture/adr-005.md
```

这会检测哪些模板部分缺失，只添加那些部分，绝不覆盖现有内容。

### 步骤 3.3：架构审核

```
/architecture-review
```

整体验证所有 ADR：
- ADR 依赖的拓扑排序（检测循环）
- 引擎兼容性验证
- GDD 修订标志（根据 ADR 决策标记需要更新的 GDD 部分）
- TR-ID 注册表维护（`docs/architecture/tr-registry.yaml`）

### 步骤 3.4：控制清单

```
/create-control-manifest
```

将所有 Accepted ADR 合并为平面化程序员规则表：

```
docs/architecture/control-manifest.md
```

包含按代码层组织的 Required 模式、Forbidden 模式和 Guardrails。
后续创建的故事嵌入清单版本日期，以便检测过期。

### 步骤 3.5：无障碍需求

使用模板创建 `design/accessibility-requirements.md`。
承诺一个层级（Basic / Standard / Comprehensive / Exemplary）
并填写 4 轴特性矩阵（视觉、运动、认知、听觉）。

此文档在阶段 3 是必需的，因为 UX 规格（在阶段 4 编写）引用此层级 —
它是设计前置条件，不是 UX 产出。

### 阶段 3 关卡

```
/gate-check technical-setup
```

**通过要求：**

- `docs/architecture/architecture.md` 存在
- 至少 3 个 ADR 存在且为 Accepted
- 存在架构审核报告
- `docs/architecture/control-manifest.md` 存在
- `design/accessibility-requirements.md` 存在

---

## 阶段 4：预生产

### 此阶段做什么

您为主要界面创建 UX 规格，为有风险的机制制作原型，
将设计文档转化为可实现的故事，规划第一个迭代，
并构建一个垂直切片，证明核心循环是有趣的。

### 阶段 4 流程

```
/ux-design  -->  /prototype  -->  /create-epics  -->  /create-stories  -->  /sprint-plan
    |                |                  |                   |                       |
    v                v                  v                   v                       v
  UX 规格       一次性原型        production/            production/           第一个迭代含
  design/ux/   在 prototypes/   epics/*/EPIC.md        epics/*/story-*.md     优先级故事
                                 （每个模块一个）       （每个行为一个）        production/sprints/
    |                                                      |                sprint-*.md
    v                                                      v
 /ux-review                                        /story-readiness
 （验证规格                                        （验证每个故事
  在 epics 之前）                                   在拾取前）
                                                           |
                                                           v
                                                       /dev-story
                                                    （实现故事，
                                                     路由到正确 agent）
                         |
                         v
                   垂直切片
                   （可玩构建，
                    3 次无指导会话）
```

### 步骤 4.1：主要界面的 UX 规格

在编写 epics 之前，创建 UX 规格，以便故事作者知道存在哪些界面
以及他们必须支持哪些玩家交互。

**UX 规格：**

```
/ux-design main-menu
/ux-design core-gameplay-hud
```

三种模式：screen/flow、HUD 和交互模式。输出到 `design/ux/`。
每个规格包括：玩家需求、布局区域、状态、交互地图、数据需求、
触发的事件、无障碍、本地化。

读取您在阶段 3 编写的 `accessibility-requirements.md`
和 `technical-preferences.md` 中的输入方法配置，
以驱动无障碍和输入覆盖检查 — 不需要每个界面重复指定。

> **提示：** `/design-system` 为每个有 UI 需求的系统发出 📌 UX Flag。
> 使用这些 flag 作为检查清单，确定哪些界面需要规格。

**交互模式库：**

```
/ux-design interaction-patterns
```

创建 `design/ux/interaction-patterns.md` — 16 个标准控件
加上游戏特定模式（物品栏槽位、技能图标、HUD 条、对话框等），
含动画和声音标准。

**UX 审核：**

```
/ux-review all
```

验证 UX 规格的 GDD 对齐和无障碍层级合规性。
产生 APPROVED / NEEDS REVISION / MAJOR REVISION NEEDED 判定。

### 步骤 4.2：为有风险的机制制作原型

不是所有东西都需要原型。只有在以下情况下制作原型：
- 一个机制是新颖的，您不确定它是否有趣
- 一个技术方法有风险，您不确定它是否可行
- 两个设计选项看起来都可行，您需要感受差异

```
/prototype "grappling hook movement with momentum"
```

**会发生什么：** 技能与您协作定义假设、成功标准和最小范围。
`prototyper` agent 在隔离的 git worktree（`isolation: worktree`）中工作，
因此一次性代码永远不会污染 `src/`。

**关键规则：** `prototype-code` 规则故意放宽编码标准 —
允许硬编码值，不需要测试 — 但必须包含假设和发现的 README。

### 步骤 4.3：从设计产出创建 Epics 和 Stories

```
/create-epics layer: foundation
/create-stories [epic-slug]   # 每个 epic 重复
/create-epics layer: core
/create-stories [epic-slug]   # 每个 core epic 重复
```

`/create-epics` 读取您的 GDD、ADR 和架构来定义 epic 范围 —
每个架构模块一个 epic。然后 `/create-stories` 将每个 epic
分解为 `production/epics/[slug]/` 中的可实现故事文件。每个故事嵌入：
- GDD 需求引用（TR-ID，不是引用文本 — 保持新鲜）
- ADR 引用（仅来自 Accepted ADR；Proposed ADR 导致 `Status: Blocked`）
- 控制清单版本日期（用于过期检测）
- 引擎特定实现说明
- 来自 GDD 的验收标准

故事存在后，运行 `/dev-story [story-path]` 来实现一个 —
它自动路由到正确的程序员 agent。

### 步骤 4.4：在拾取前验证故事

```
/story-readiness production/stories/combat-damage-calc.md
```

检查：设计完整性、架构覆盖、范围清晰度、Definition of Done。
判定：READY / NEEDS WORK / BLOCKED。

### 步骤 4.5：工作量估算

```
/estimate production/stories/combat-damage-calc.md
```

提供带风险评估的工作量估算。

### 步骤 4.6：规划您的第一个迭代

```
/sprint-plan new
```

**会发生什么：** `producer` agent 协作进行迭代规划：
- 询问迭代目标和可用时间
- 将目标分解为 Must Have / Should Have / Nice to Have 任务
- 识别风险和阻塞
- 创建 `production/sprints/sprint-01.md`
- 填充 `production/sprint-status.yaml`（机器可读的故事跟踪）

### 步骤 4.7：垂直切片（硬关卡）

在进入生产阶段之前，您必须构建和测试垂直切片：

- 一个完整的端到端核心循环，可从头玩到尾
- 代表性质量（不是占位符满天飞）
- 至少 3 次无指导会话中玩过
- 编写测试报告（`/playtest-report`）

这是一个**硬关卡** — 如果人类没有无指导地玩过构建，
`/gate-check` 会自动 FAIL。

### 阶段 4 关卡

```
/gate-check pre-production
```

**通过要求：**

- `design/ux/` 中至少 1 个 UX 规格已审核
- UX 审核完成（APPROVED 或 NEEDS REVISION 且有记录风险）
- 至少 1 个原型含 README
- `production/stories/` 中存在故事文件
- 至少 1 个迭代计划存在
- 至少 1 个测试报告存在（垂直切片在 3+ 次会话中玩过）

---

## 阶段 5：生产

### 此阶段做什么

这是核心生产循环。您按迭代（通常 1-2 周）工作，
逐故事实现功能、跟踪进度，并通过结构化完成审核关闭故事。
此阶段重复直到您的游戏内容完整。

### 阶段 5 流程（每个迭代）

```
/sprint-plan new  -->  /story-readiness  -->  implement  -->  /story-done
       |                     |                    |                |
       v                     v                    v                v
  迭代创建             故事验证            代码编写          8 阶段审核：
  sprint-status.yaml   READY 判定          测试通过         验证标准、
  已填充                                                    检查偏离、
                                                             更新故事状态
       |
       |  （对每个故事重复直到迭代完成）
       v
  /sprint-status  （随时快速 30 行快照）
  /scope-check    （如果范围增长）
  /retrospective  （迭代结束时）
```

### 步骤 5.1：故事生命周期

生产阶段以**故事生命周期**为中心：

```
/story-readiness  -->  implement  -->  /story-done  -->  next story
```

**1. Story Readiness：** 在拾取故事前验证它：

```
/story-readiness production/stories/combat-damage-calc.md
```

这会检查设计完整性、架构覆盖、ADR 状态（如果 ADR 仍是 Proposed 则阻塞）、
控制清单版本（如果过期则警告）和范围清晰度。
判定：READY / NEEDS WORK / BLOCKED。

**2. 实现：** 与适当的 agent 协作工作：

- `gameplay-programmer` 用于游戏系统
- `engine-programmer` 用于核心引擎工作
- `ai-programmer` 用于 AI 行为
- `network-programmer` 用于多人游戏
- `ui-programmer` 用于 UI 代码
- `tools-programmer` 用于开发工具

所有 agent 遵循协作协议：他们读取设计文档，
提出澄清性问题，呈现架构选项，获得您的批准，然后实现。

**3. 故事完成：** 当一个故事完成时：

```
/story-done production/stories/combat-damage-calc.md
```

这会运行 8 阶段完成审核：
1. 查找并读取故事文件
2. 加载引用的 GDD、ADR 和控制清单
3. 验证验收标准（自动检查、手动、延期）
4. 检查 GDD/ADR 偏离（BLOCKING / ADVISORY / OUT OF SCOPE）
5. 提示代码审核
6. 生成完成报告（COMPLETE / COMPLETE WITH NOTES / BLOCKED）
7. 更新故事 `Status: Complete` 及完成备注
8. 呈现下一个就绪故事

审核期间发现的技术债务记录到 `docs/tech-debt-register.md`。

### 步骤 5.2：迭代跟踪

随时检查进度：

```
/sprint-status
```

从 `production/sprint-status.yaml` 读取快速 30 行快照。

如果范围增长：

```
/scope-check production/sprints/sprint-03.md
```

这会比较当前范围与原始计划，并标记范围增加、建议削减。

### 步骤 5.3：内容跟踪

```
/content-audit
```

比较 GDD 指定的内容与已实现的内容。及早发现内容差距。

### 步骤 5.4：设计变更传播

当 GDD 在故事创建后发生变化时：

```
/propagate-design-change design/gdd/combat-system.md
```

Git-diff GDD，查找受影响的 ADR，生成影响报告，
并引导您完成 Superseded/update/keep 决策。

### 步骤 5.5：多系统功能（团队编排）

对于跨多个领域的功能，使用团队技能：

```
/team-combat "healing ability with HoT and cleanse"
/team-narrative "Act 2 story content"
/team-ui "inventory screen redesign"
/team-level "forest dungeon level"
/team-audio "combat audio pass"
```

每个团队技能协调 6 阶段协作工作流：
1. **设计** — game-designer 提问，呈现选项
2. **架构** — lead-programmer 提议代码结构
3. **并行实现** — specialists 同时工作
4. **集成** — gameplay-programmer 连接一切
5. **验证** — qa-tester 按验收标准运行
6. **报告** — coordinator 总结状态

编排是自动化的，但**决策点由您决定**。

### 步骤 5.6：迭代审核和下一迭代

迭代结束时：

```
/retrospective
```

分析计划 vs 完成、速度、阻塞和可操作的改进。

然后规划下一迭代：

```
/sprint-plan new
```

### 步骤 5.7：里程碑审核

在里程碑检查点：

```
/milestone-review "alpha"
```

产生功能完整性、质量指标、风险评估和 go/no-go 建议。

### 阶段 5 关卡

```
/gate-check production
```

**通过要求：**

- 所有 MVP 故事完成
- 测试：涵盖新玩家、中期游戏和难度曲线的 3 次会话
- 乐趣假设已验证
- 测试数据中没有困惑循环

---

## 阶段 6：打磨

### 此阶段做什么

您的游戏功能完整。现在让它变好。此阶段专注于
性能、平衡、无障碍、音频、视觉打磨和测试。

### 阶段 6 流程

```
/perf-profile  -->  /balance-check  -->  /asset-audit  -->  /playtest-report (x3)
       |                  |                    |                    |
       v                  v                    v                    v
  分析 CPU/GPU       分析公式            验证命名、           覆盖：新玩家、
  内存，优化         和数据中的           格式、尺寸           中期游戏、难度
  瓶颈               破坏性进度                                曲线

  /tech-debt  -->  /team-polish
       |                |
       v                v
  跟踪和          协调轮次：
  优先化          性能 + 美术 +
  债务项          音频 + UX + QA
```

### 步骤 6.1：性能分析

```
/perf-profile
```

引导您完成结构化性能分析：
- 设定目标（FPS、内存、平台）
- 按影响排序识别瓶颈
- 生成含代码位置和预期收益的可操作优化任务

### 步骤 6.2：平衡分析

```
/balance-check assets/data/combat_damage.json
```

分析平衡数据中的统计异常值、破坏性进度曲线、
退化策略和经济失衡。

### 步骤 6.3：资源审核

```
/asset-audit
```

验证所有资源的命名规范、文件格式标准和尺寸预算。

### 步骤 6.4：测试（必需：3 次会话）

```
/playtest-report
```

生成结构化测试报告。需要 3 次会话，涵盖：
- 新玩家体验
- 中期游戏系统
- 难度曲线

### 步骤 6.5：技术债务评估

```
/tech-debt
```

扫描 TODO/FIXME/HACK 注释、代码重复、过于复杂的函数、
缺失测试和过时依赖。每个项目分类和优先化。

### 步骤 6.6：协调打磨轮次

```
/team-polish "combat system"
```

并行协调 4 个专家：
1. 性能优化（performance-analyst）
2. 视觉打磨（technical-artist）
3. 音频打磨（sound-designer）
4. 感受/ juice（gameplay-programmer + technical-artist）

您设置优先级；团队在每个步骤获得您的批准后执行。

### 步骤 6.7：本地化和无障碍

```
/localize src/
```

扫描硬编码字符串、破坏翻译的连接、不考虑扩展的文本
和缺失的 locale 文件。

无障碍根据阶段 3 无障碍需求文档中承诺的层级进行审核。

### 阶段 6 关卡

```
/gate-check polish
```

**通过要求：**

- 至少 3 个测试报告存在
- 协调打磨轮次完成（`/team-polish`）
- 无阻塞性能问题
- 无障碍层级要求已满足

---

## 阶段 7：发布

### 此阶段做什么

您的游戏已打磨、测试完毕、准备发货。现在发货。

### 阶段 7 流程

```
/release-checklist  -->  /launch-checklist  -->  /team-release
        |                       |                      |
        v                       v                      v
  发布前验证           完整跨部门验证         协调：
  代码、内容、         （各部门 Go/No-Go）    构建、QA 签字、
  商店、法律                                  部署、发布
                    另外：/changelog、/patch-notes、/hotfix
```

### 步骤 7.1：发布清单

```
/release-checklist v1.0.0
```

生成全面的发布前清单，涵盖：
- 构建验证（所有平台编译并运行）
- 认证要求（平台特定）
- 商店元数据（描述、截图、预告片）
- 法律合规（EULA、隐私政策、评级）
- 存档兼容性
- 分析验证

### 步骤 7.2：发布就绪（完整验证）

```
/launch-checklist
```

完整跨部门验证：

| 部门 | 检查内容 |
|-----------|------------|
| **工程** | 构建稳定性、崩溃率、内存泄漏、加载时间 |
| **设计** | 功能完整性、教程流程、难度曲线 |
| **美术** | 资源质量、缺失纹理、LOD 级别 |
| **音频** | 缺失声音、混音级别、空间音频 |
| **QA** | 按严重性的开放 bug 数量、回归套件通过率 |
| **叙事** | 对话完整性、lore 一致性、拼写错误 |
| **本地化** | 所有字符串已翻译、无截断、locale 测试 |
| **无障碍** | 合规清单、辅助功能测试 |
| **商店** | 元数据完整、截图批准、定价已定 |
| **营销** | 新闻包就绪、发布预告片、社交媒体排期 |
| **社区** | 补丁说明草稿、FAQ 已准备、支持渠道就绪 |
| **基础设施** | 服务器已扩展、CDN 已配置、监控已激活 |
| **法务** | EULA 定稿、隐私政策、COPPA/GDPR 合规 |

每项获得 **Go / No-Go** 状态。全部为 Go 才能发货。

### 步骤 7.3：生成玩家面向内容

```
/patch-notes v1.0.0
```

从 git 历史和迭代数据生成玩家友好的补丁说明。
将开发者语言翻译为玩家语言。

```
/changelog v1.0.0
```

生成内部变更日志（更技术化，供团队使用）。

### 步骤 7.4：协调发布

```
/team-release
```

协调 release-manager、QA 和 DevOps 通过：
1. 发布前验证
2. 构建管理
3. 最终 QA 签字
4. 部署准备
5. Go/No-Go 决策

### 步骤 7.5：发货

`validate-push` 钩子会在推送到 `main` 或 `develop` 时警告您。
这是故意的 — 发布推送应该深思熟虑：

```bash
git tag v1.0.0
git push origin main --tags
```

### 步骤 7.6：发布后

**关键生产 bug 的热修复工作流：**

```
/hotfix "Players losing save data when inventory exceeds 99 items"
```

绕过正常迭代流程，保留完整审计跟踪：
1. 创建热修复分支
2. 实现修复
3. 确保回退到开发分支
4. 记录事件

**发布稳定后的复盘：**

让 Claude 使用 `.claude/docs/templates/post-mortem.md` 中的模板创建复盘

---

## 跨领域关注点

这些主题适用于所有阶段。

### 导演审核模式

导演门是 specialist agent，在关键工作流步骤审核您的工作。
默认情况下它们在每个检查点运行。您可以控制获得的审核量。

**在 `/start` 期间设置审核强度。** 保存到 `production/review-mode.txt`。

| 模式 | 运行内容 | 适合场景 |
|------|-----------|----------|
| `full` | 每步运行所有导演门 | 新项目、学习系统 |
| `lean` | 仅在阶段转换时运行导演（`/gate-check`） | 有经验的开发者 |
| `solo` | 无导演审核 | 游戏 jam、原型、最大速度 |

**单次运行覆盖**而不更改全局设置：

```
/brainstorm space horror --review full
/architecture-decision --review solo
```

`--review` 标志适用于所有使用门的技能。
随时通过直接编辑 `production/review-mode.txt` 或重新运行 `/start` 更改全局模式。

完整门定义和检查模式：`.claude/docs/director-gates.md`

---

### 协作协议

本系统是**用户驱动的协作**，不是自主的。

**模式：** 提问 > 选项 > 决策 > 起草 > 批准

每个 Agent 交互遵循此模式：
1. Agent 提出澄清性问题
2. Agent 呈现 2-4 个带权衡和推理的选项
3. 您做决策
4. Agent 基于您的决策起草
5. 您审核并细化
6. Agent 在写入前问"我可以写入 [filepath] 吗？"

详见 `docs/COLLABORATIVE-DESIGN-PRINCIPLE.md` 的完整协议和示例。

### AskUserQuestion 工具

Agent 使用 `AskUserQuestion` 工具进行结构化选项呈现。
