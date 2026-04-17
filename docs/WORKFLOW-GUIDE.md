# Claude Code Game Studios — 完整工作流指南

> **如何使用智能体架构从零开始到游戏上线。**
>
> 本指南将带你了解使用 48 智能体系统、68 个斜杠命令和 12 个自动化钩子进行游戏开发的每个阶段。假设你已经安装了 Claude Code 并从项目根目录开始工作。
>
> 流程共有 7 个阶段。每个阶段都有一个正式关卡（`/gate-check`），必须通过才能进入下一阶段。权威的阶段顺序定义在 `.claude/docs/workflow-catalog.yaml` 中，由 `/help` 读取。

---

## 目录

1. [快速开始](#快速开始)
2. [第一阶段：概念](#第一阶段概念)
3. [第二阶段：系统设计](#第二阶段系统设计)
4. [第三阶段：技术搭建](#第三阶段技术搭建)
5. [第四阶段：预生产](#第四阶段预生产)
6. [第五阶段：生产](#第五阶段生产)
7. [第六阶段：打磨](#第六阶段打磨)
8. [第七阶段：发布](#第七阶段发布)
9. [跨领域关注点](#跨领域关注点)
10. [附录 A：智能体快速参考](#附录-a智能体快速参考)
11. [附录 B：斜杠命令快速参考](#附录-b斜杠命令快速参考)
12. [附录 C：常用工作流](#附录-c常用工作流)

---

## 快速开始

### 准备工作

开始之前，请确保你具备以下条件：

- 已安装并可正常运行的 **Claude Code**
- **Git**（Windows 上使用 Git Bash，Mac/Linux 使用标准终端）
- **jq**（可选但推荐——如果缺失，钩子会回退到 `grep`）
- **Python 3**（可选——部分钩子用它做 JSON 验证）

### 第一步：克隆并打开项目

```bash
git clone <repo-url> my-game
cd my-game
```

### 第二步：运行 /start

如果是首次会话：

```
/start
```

这个引导式入职流程会询问你目前处于哪个阶段，然后路由到正确的阶段：

- **路径 A** — 还没有想法：路由到 `/brainstorm`
- **路径 B** — 有模糊的想法：带着种子路由到 `/brainstorm`
- **路径 C** — 已有明确概念：路由到 `/setup-engine` 和 `/map-systems`
- **路径 D1** — 已有项目，但产物较少：正常流程
- **路径 D2** — 已有项目，存在 GDD/ADR：运行 `/project-stage-detect`，然后运行 `/adopt` 进行棕地迁移

### 第三步：验证钩子是否正常工作

开启一个新的 Claude Code 会话。你应该能看到 `session-start.sh` 钩子的输出：

```
=== Claude Code Game Studios -- Session Context ===
Branch: main
Recent commits:
  abc1234 Initial commit
===================================
```

如果看到这个输出，说明钩子正在工作。如果没有，请检查 `.claude/settings.json` 确保钩子路径对于你的操作系统是正确的。

### 第四步：随时获取帮助

在任何时候运行：

```
/help
```

这会从 `production/stage.txt` 读取你当前的阶段，检查存在哪些产物，并告诉你接下来具体要做什么。它会区分 REQUIRED（必须）下一步和 OPTIONAL（可选）机会。

### 第五步：创建目录结构

目录会在需要时自动创建。系统期望以下布局：

```
src/                  # 游戏源代码
  core/               # 引擎/框架代码
  gameplay/           # 游戏玩法系统
  ai/                 # AI 系统
  networking/         # 多人游戏代码
  ui/                 # UI 代码
  tools/              # 开发工具
assets/               # 游戏资源
  art/                # 精灵、模型、纹理
  audio/              # 音乐、音效
  vfx/                # 粒子效果
  shaders/            # 着色器文件
  data/               # JSON 配置/平衡数据
design/               # 设计文档
  gdd/                # 游戏设计文档
  narrative/          # 故事、世界观、对话
  levels/             # 关卡设计文档
  balance/            # 平衡表格和数据
  ux/                 # UX 规格说明
docs/                 # 技术文档
  architecture/       # 架构决策记录
  api/                # API 文档
  postmortems/        # 复盘文档
tests/                # 测试套件
prototypes/           # 临时原型
production/           # 冲刺计划、里程碑、发布
  sprints/
  milestones/
  releases/
  epics/              # 史诗和故事文件（来自 /create-epics + /create-stories）
  playtests/          # 测试报告
  session-state/      # 临时会话状态（gitignore）
  session-logs/       # 会话审计跟踪（gitignore）
```

> **提示：** 第一天你不需要所有这些目录。在需要该阶段的目录时才创建。重要的是在创建时遵循这个结构，因为**规则系统**基于文件路径强制执行标准。`src/gameplay/` 中的代码应用游戏玩法规则，`src/ai/` 中的代码应用 AI 规则，以此类推。

---

## 第一阶段：概念

### 这个阶段做什么

你从"完全没有想法"或"只有模糊想法"到一个包含定义好的支柱和玩家旅程的结构化游戏概念文档。这是弄清楚**你要做什么**和**为什么做**的阶段。

### 第一阶段流程

```
/brainstorm  -->  game-concept.md  -->  /design-review  -->  /setup-engine
     |                                        |                    |
     v                                        v                    v
  10 个概念      包含支柱、MDA、       概念文档       引擎在 technical-
  MDA 分析      核心循环、USP       验证            preferences.md 中固定
  玩家动机      的概念文档
                                                                   |
                                                                   v
                                                             /map-systems
                                                                   |
                                                                   v
                                                            systems-index.md
                                                            （所有系统、依赖、
                                                             优先级层级）
```

### 1.1：使用 /brainstorm 进行头脑风暴

这是你的起点。运行头脑风暴技能：

```
/brainstorm
```

或者带有类型提示：

```
/brainstorm roguelike deckbuilder
```

**发生了什么：** 头脑风暴技能引导你通过使用专业工作室技术的协作式六阶段构思流程：

1. 询问你的兴趣、主题和约束
2. 生成 10 个概念种子，并附 MDA（机制、动态、美学）分析
3. 你选择 2-3 个最喜欢的进行深度分析
4. 执行玩家动机映射和受众定位
5. 你选择获胜概念
6. 将其正式写入 `design/gdd/game-concept.md`

概念文档包括：

- 电梯演讲（一句话）
- 核心幻想（玩家想象自己在做什么）
- MDA 分解
- 目标受众（Bartle 类型、人口统计）
- 核心循环图
- 独特卖点
- 可比标题和差异化
- 游戏支柱（3-5 个不可妥协的设计价值）
- 反支柱（游戏有意避免的东西）

### 1.2：审查概念（可选但推荐）

```
/design-review design/gdd/game-concept.md
```

在继续之前验证结构和完整性。

### 1.3：选择你的引擎

```
/setup-engine
```

或者指定引擎：

```
/setup-engine godot 4.6
```

**/setup-engine 做什么：**

- 用命名约定、性能预算和引擎特定默认值填充 `.claude/docs/technical-preferences.md`
- 检测知识缺口（引擎版本比 LLM 训练数据更新），并建议交叉参考 `docs/engine-reference/`
- 在 `docs/engine-reference/` 中创建版本固定的参考文档

**为什么这很重要：** 一旦设置了引擎，系统就知道使用哪些引擎专家智能体。如果你选择 Godot，像 `godot-specialist`、`godot-gdscript-specialist` 和 `godot-shader-specialist` 这样的智能体就会成为你的首选专家。

### 1.4：将你的概念分解为系统

在编写单独的 GDD 之前，枚举你的游戏需要的所有系统：

```
/map-systems
```

这会创建 `design/gdd/systems-index.md` —— 一个主跟踪文档，包含：

- 列出你的游戏需要的每个系统（战斗、移动、UI 等）
- 映射系统之间的依赖关系
- 分配优先级层级（MVP、垂直切片、Alpha、完整愿景）
- 确定设计顺序（基础 > 核心 > 功能 > 表现 > 打磨）

这一步是进入第二阶段之前的**必须项**。来自 155 个游戏复盘的研究证实，跳过系统枚举会在生产中造成 5-10 倍的成本。

### 第一阶段关卡

```
/gate-check concept
```

**通过要求：**

- 引擎已在 `technical-preferences.md` 中配置
- `design/gdd/game-concept.md` 存在并包含支柱
- `design/gdd/systems-index.md` 存在并包含依赖排序

**裁定：** PASS / CONCERNS / FAIL。CONCERNS 如果有已确认的风险可通过。FAIL 阻止前进。

---

## 第二阶段：系统设计

### 这个阶段做什么

你创建定义游戏如何运作的所有设计文档。这个阶段还不写代码——纯粹是设计。系统索引中标识的每个系统都有自己的 GDD，逐节编写，逐个审查，然后所有 GDD 进行交叉检查以确保一致性。

### 第二阶段流程

```
/map-systems next  -->  /design-system  -->  /design-review
       |                     |                     |
       v                     v                     v
  从 systems-index    逐节 GDD 编写         验证 8 个
  选取下一个系统      （增量写入）          必需章节
                                             APPROVED/NEEDS REVISION
       |
       |  （对每个 MVP 系统重复）
       v
/review-all-gdds
       |
       v
  跨 GDD 一致性 + 设计理论审查
  PASS / CONCERNS / FAIL
```

### 2.1：编写系统 GDD

使用引导式工作流按依赖顺序设计每个系统：

```
/map-systems next
```

这会选取优先级最高但尚未设计的系统，然后交接给 `/design-system`，后者会引导你逐节创建其 GDD。

你也可以直接设计特定系统：

```
/design-system combat-system
```

**/design-system 做什么：**

1. 读取你的游戏概念、系统索引以及任何上游/下游 GDD
2. 运行技术可行性预检（领域映射 + 可行性简报）
3. 逐一引导你完成 8 个必需的 GDD 章节
4. 每个章节遵循：上下文 > 问题 > 选项 > 决策 > 起草 > 批准 > 写入
5. 每个章节在批准后立即写入文件（崩溃也不会丢失）
6. 标记与现有已批准 GDD 的冲突
7. 按类别路由到专家智能体（数学路由到 systems-designer，经济路由到 economy-designer，故事系统路由到 narrative-director）

**8 个必需的 GDD 章节：**

| # | 章节 | 内容 |
|---|------|------|
| 1 | **概述** | 系统的单段总结 |
| 2 | **玩家幻想** | 玩家使用该系统时想象/感受到的东西 |
| 3 | **详细规则** | 明确的机械规则 |
| 4 | **公式** | 每个计算，包含变量定义和范围 |
| 5 | **边缘情况** | 奇怪情况下会发生什么？明确解决。 |
| 6 | **依赖** | 连接到的其他系统（双向） |
| 7 | **调优旋钮** | 设计师可以安全更改的值，带安全范围 |
| 8 | **验收标准** | 如何测试它是否有效？具体、可衡量。 |

加上一个 **游戏手感** 章节：手感参考、输入响应（毫秒/帧）、动画手感目标（启动/激活/恢复）、冲击时刻、重量曲线。

### 2.2：审查每个 GDD

在下一个系统开始之前，验证当前的：

```
/design-review design/gdd/combat-system.md
```

检查所有 8 个章节的完整性、公式清晰度、边缘情况解决、双向依赖和可测试的验收标准。

**裁定：** APPROVED / NEEDS REVISION / MAJOR REVISION。只有 APPROVED 的 GDD 才能继续。

### 2.3：无需完整 GDD 的小更改

对于调优更改、小添加或不值得完整 GDD 的调整：

```
/quick-design "add 10% damage bonus for flanking attacks"
```

这会在 `design/quick-specs/` 中创建一个轻量级规格，而不是完整的 8 节 GDD。用于调优、数字更改和小添加。

### 2.4：跨 GDD 一致性审查

在所有 MVP 系统 GDD 单独批准之后：

```
/review-all-gdds
```

这会同时读取所有 GDD 并运行两个分析阶段：

**第一阶段——跨 GDD 一致性：**
- 依赖双向性（A 引用 B，B 是否也引用 A？）
- 系统之间的规则矛盾
- 对已重命名或已删除系统的过时引用
- 所有权冲突（两个系统声称承担同一责任）
- 公式范围兼容性（系统 A 的输出是否适合系统 B 的输入？）
- 验收标准交叉检查

**第二阶段——设计理论（游戏设计整体性）：**
- 竞争性进度循环（两个系统是否争夺相同的奖励空间？）
- 认知负荷（同时激活超过 4 个系统？）
- 主导策略（一种方法是否使所有其他方法无关？）
- 经济循环分析（来源和消耗平衡吗？）
- 系统间的难度曲线一致性
- 支柱一致性和反支柱违规
- 玩家幻想连贯性

**输出：** `design/gdd/gdd-cross-review-[date].md`，包含裁定。

### 2.5：叙事设计（如适用）

如果你的游戏有故事、世界观或对话，这就是你构建它的时候：

1. **世界构建** — 使用 `world-builder` 定义派系、历史、地理和你的世界的规则
2. **故事结构** — 使用 `narrative-director` 设计故事弧、角色弧和叙事节拍
3. **角色表** — 使用 `narrative-character-sheet.md` 模板

### 第二阶段关卡

```
/gate-check systems-design
```

**通过要求：**

- `systems-index.md` 中所有 MVP 系统的 `Status: Approved`
- 每个 MVP 系统都有已审查的 GDD
- 跨 GDD 审查报告存在（`design/gdd/gdd-cross-review-*.md`），裁定为 PASS 或 CONCERNS（不是 FAIL）

---

## 第三阶段：技术搭建

### 这个阶段做什么

你做出关键的技术决策，将其记录为架构决策记录（ADR），通过审查进行验证，并生成一个控制清单，为程序员提供直接的、可操作的规则。你还建立 UX 基础。

### 第三阶段流程

```
/create-architecture  -->  /architecture-decision (x N)  -->  /architecture-review
        |                          |                                   |
        v                          v                                   v
  覆盖所有系统       每个决策的 ADR        验证完整性、
  的主架构文档      在 docs/architecture/  依赖排序、
                   adr-*.md             引擎兼容性
                                                                      |
                                                                      v
                                                         /create-control-manifest
                                                                      |
                                                                      v
                                                         扁平的程序员规则
                                                         docs/architecture/
                                                         control-manifest.md
        此阶段同时进行：
        -------------------
        /ux-design  -->  /ux-review
        无障碍需求文档
        交互模式库
```

### 3.1：主架构文档

```
/create-architecture
```

在 `docs/architecture/architecture.md` 中创建涵盖系统边界、数据流和集成点的总体架构文档。

### 3.2：架构决策记录（ADR）

对于每个重要的技术决策：

```
/architecture-decision "State Machine vs Behavior Tree for NPC AI"
```

**发生了什么：** 该技能引导你创建 ADR，包含：
- 背景和决策驱动因素
- 所有选项及其优缺点和引擎兼容性
- 所选选项及理由
- 后果（正面、负面、风险）
- 依赖（依赖于、启用、阻塞、排序说明）
- 解决的 GDD 需求（通过 TR-ID 链接）

ADR 经历生命周期：Proposed > Accepted > Superseded/Deprecated。

**关卡检查前至少需要 3 个 Foundation 层 ADR。**

**改造现有 ADR：** 如果你已有棕地项目的 ADR：

```
/architecture-decision retrofit docs/architecture/adr-005.md
```

这会检测哪些模板章节缺失，只添加那些缺失的部分，绝不会覆盖现有内容。

### 3.3：架构审查

```
/architecture-review
```

一起验证所有 ADR：
- ADR 依赖的拓扑排序（检测循环）
- 引擎兼容性验证
- GDD 修订标志（根据 ADR 选择标记需要更新的 GDD 章节）
- TR-ID 注册表维护（`docs/architecture/tr-registry.yaml`）

### 3.4：控制清单

```
/create-control-manifest
```

获取所有已接受的 ADR 并生成扁平的程序员规则表：

```
docs/architecture/control-manifest.md
```

这包含按代码层组织 Required patterns、Forbidden patterns 和 Guardrails。稍后创建的故事会嵌入清单版本日期，以便检测过时情况。

### 3.5：无障碍需求

使用模板创建 `design/accessibility-requirements.md`。承诺一个层级（Basic / Standard / Comprehensive / Exemplary）并填写四轴功能矩阵（视觉、运动、认知、听觉）。

此文档在第三阶段是必需的，因为 UX 规格（在第四阶段编写）引用此层级——它是设计前提条件，不是 UX 可交付物。

### 第三阶段关卡

```
/gate-check technical-setup
```

**通过要求：**

- `docs/architecture/architecture.md` 存在
- 至少 3 个 ADR 存在且为 Accepted 状态
- 架构审查报告存在
- `docs/architecture/control-manifest.md` 存在
- `design/accessibility-requirements.md` 存在

---

## 第四阶段：预生产

### 这个阶段做什么

你为关键屏幕创建 UX 规格，为有风险的机制制作原型，将设计文档转化为可实现的故事，规划你的第一个冲刺，并构建一个证明核心循环有趣的垂直切片。

### 第四阶段流程

```
/ux-design  -->  /prototype  -->  /create-epics  -->  /create-stories  -->  /sprint-plan
    |                |                  |                   |                       |
    v                v                  v                   v                       v
  UX 规格       临时原型         production/          production/           第一个冲刺，
  design/ux/    在 prototypes/   epics/*/EPIC.md      epics/*/story-*.md    包含优先级排序
                （每个模块一个）   （每个行为一个）      的故事             production/
                                                                              sprints/sprint-*.md
    |                                                      |
    v                                                      v
 /ux-review                                         /story-readiness
 （验证规格                                           （验证每个故事
  再进入 epics）                                      再领取）
                                                           |
                                                           v
                                                       /dev-story
                                                     （实现故事，
                                                      路由到正确的智能体）
                         |
                         v
                   Vertical Slice
                   （可玩构建，
                    3 次无人引导会话）
```

### 4.1：关键屏幕的 UX 规格

在编写 epic 之前，创建 UX 规格，以便故事作者知道存在哪些屏幕以及他们必须支持哪些玩家交互。

**UX 规格：**

```
/ux-design main-menu
/ux-design core-gameplay-hud
```

三种模式：屏幕/流程、HUD 和交互模式。输出到 `design/ux/`。每个规格包括：玩家需求、布局区域、状态、交互地图、数据需求、触发事件、无障碍、本地化。

读取你在第三阶段编写的 `accessibility-requirements.md` 和 `technical-preferences.md` 中的输入方法配置，以驱动无障碍和输入覆盖检查——无需每个屏幕重新指定。

> **提示：** `/design-system` 为每个有 UI 需求的功能发出 📌 UX Flag。使用这些标志作为检查清单，以确定哪些屏幕需要规格。

**交互模式库：**

```
/ux-design interaction-patterns
```

创建 `design/ux/interaction-patterns.md` — 16 个标准控件加上游戏特定模式（物品栏槽、技能图标、HUD 条、对话框等），带有动画和声音标准。

**UX 审查：**

```
/ux-review all
```

验证 UX 规格的 GDD 对齐和无障碍层级合规性。生成 APPROVED / NEEDS REVISION / MAJOR REVISION NEEDED 裁定。

### 4.2：风险机制原型

并非所有东西都需要原型。只有在以下情况下才做原型：
- 一个机制是新颖的，你不确定它是否有趣
- 一个技术方法有风险，你不确定它是否可行
- 两个设计选项看起来都可行，你需要感受差异

```
/prototype "grappling hook movement with momentum"
```

**发生了什么：** 该技能与你协作定义假设、成功标准和最小范围。`prototyper` 智能体在隔离的 git worktree 中工作（`isolation: worktree`），因此临时代码永远不会污染 `src/`。

**关键规则：** `prototype-code` 规则有意放宽编码标准——硬编码值可以接受，不需要测试——但必须包含假设和发现的 README。

### 4.3：从设计产物创建 Epic 和 Story

```
/create-epics layer: foundation
/create-stories [epic-slug]   # 对每个 epic 重复
/create-epics layer: core
/create-stories [epic-slug]   # 对每个核心 epic 重复
```

`/create-epics` 读取你的 GDD、ADR 和架构来定义 epic 范围——每个架构模块一个 epic。然后 `/create-stories` 将每个 epic 分解为 `production/epics/[slug]/` 中的可实现故事文件。每个故事嵌入：
- GDD 需求引用（TR-ID，而非引用文本——保持新鲜）
- ADR 引用（仅来自 Accepted ADR；Proposed ADR 导致 `Status: Blocked`）
- 控制清单版本日期（用于检测过时）
- 引擎特定实现说明
- 来自 GDD 的验收标准

一旦故事存在，运行 `/dev-story [story-path]` 来实现一个——它会自动路由到正确的程序员智能体。

### 4.4：领取前验证故事

```
/story-readiness production/stories/combat-damage-calc.md
```

检查：设计完整性、架构覆盖、范围清晰、定义完成。裁定：READY / NEEDS WORK / BLOCKED。

### 4.5：工作量估算

```
/estimate production/stories/combat-damage-calc.md
```

提供带风险评估的工作量估算。

### 4.6：规划你的第一个冲刺

```
/sprint-plan new
```

**发生了什么：** `producer` 智能体协作进行冲刺规划：
- 询问冲刺目标和可用时间
- 将目标分解为 Must Have / Should Have / Nice to Have 任务
- 识别风险和阻碍
- 创建 `production/sprints/sprint-01.md`
- 填充 `production/sprint-status.yaml`（机器可读的故事跟踪）

### 4.7：垂直切片（硬关卡）

在进入生产阶段之前，你必须构建并测试垂直切片：

- 一个完整的端到端核心循环，从头到尾可玩
- 代表性质量（不是占位符）
- 至少 3 次会话无人引导游玩
- 编写测试报告（`/playtest-report`）

这是一个**硬关卡**——如果人类没有无人引导地玩过构建，`/gate-check` 将自动 FAIL。

### 第四阶段关卡

```
/gate-check pre-production
```

**通过要求：**

- `design/ux/` 中至少 1 个 UX 规格已审查
- UX 审查完成（APPROVED 或 NEEDS REVISION 且有记录的风险）
- 至少 1 个带 README 的原型
- `production/stories/` 中存在故事文件
- 至少 1 个冲刺计划存在
- 至少 1 个测试报告存在（垂直切片在 3+ 会话中游玩）

---

## 第五阶段：生产

### 这个阶段做什么

这是核心生产循环。你以冲刺（通常 1-2 周）为单位工作，逐故事实现功能，跟踪进度，并通过结构化完成审查关闭故事。这个阶段重复，直到你的游戏内容完整。

### 第五阶段流程（每个冲刺）

```
/sprint-plan new  -->  /story-readiness  -->  实现  -->  /story-done
       |                     |                    |                |
       v                     v                    v                v
  冲刺创建           故事验证              代码编写          8 阶段审查：
  sprint-status.yaml  READY 裁定           测试通过          验证标准，
  已填充                                                  检查偏差，
                                                         更新故事状态
       |
       |  （对每个故事重复，直到冲刺完成）
       v
  /sprint-status  （随时快速 30 行快照）
  /scope-check    （如果范围在增长）
  /retrospective  （冲刺结束时）
```

### 5.1：故事生命周期

生产阶段以**故事生命周期**为中心：

```
/story-readiness  -->  实现  -->  /story-done  -->  下一个故事
```

**1. 故事就绪：** 领取故事之前，验证它：

```
/story-readiness production/stories/combat-damage-calc.md
```

这会检查设计完整性、架构覆盖、ADR 状态（如果 ADR 仍是 Proposed 则阻止）、控制清单版本（如果过时则警告）和范围清晰度。裁定：READY / NEEDS WORK / BLOCKED。

**2. 实现：** 与适当的智能体协作：

- `gameplay-programmer` 用于游戏玩法系统
- `engine-programmer` 用于核心引擎工作
- `ai-programmer` 用于 AI 行为
- `network-programmer` 用于多人游戏
- `ui-programmer` 用于 UI 代码
- `tools-programmer` 用于开发工具

所有智能体都遵循协作协议：它们阅读设计文档，提出澄清问题，展示架构选项，获得你的批准，然后实现。

**3. 故事完成：** 当一个故事完成时：

```
/story-done production/stories/combat-damage-calc.md
```

这会运行 8 阶段完成审查：
1. 找到并读取故事文件
2. 加载引用的 GDD、ADR 和控制清单
3. 验证验收标准（可自动检查的、手动的、延后的）
4. 检查 GDD/ADR 偏差（BLOCKING / ADVISORY / OUT OF SCOPE）
5. 提示进行代码审查
6. 生成完成报告（COMPLETE / COMPLETE WITH NOTES / BLOCKED）
7. 更新故事 `Status: Complete` 及完成笔记
8. 浮现下一个就绪故事

审查期间发现的技术债务会记录到 `docs/tech-debt-register.md`。

### 5.2：冲刺跟踪

随时检查进度：

```
/sprint-status
```

快速 30 行快照，从 `production/sprint-status.yaml` 读取。

如果范围在增长：

```
/scope-check production/sprints/sprint-03.md
```

这会将当前范围与原始计划进行比较，并标记范围增加、建议削减。

### 5.3：内容跟踪

```
/content-audit
```

比较 GDD 指定的内容与已实现的内容。及早发现内容差距。

### 5.4：设计变更传播

当 GDD 在创建故事后发生变化时：

```
/propagate-design-change design/gdd/combat-system.md
```

Git-diff GDD，找到受影响的 ADR，生成影响报告，并引导你完成 Superseded/update/keep 决策。

### 5.5：多系统功能（团队协调）

对于跨多个领域的功能，使用团队技能：

```
/team-combat "healing ability with HoT and cleanse"
/team-narrative "Act 2 story content"
/team-ui "inventory screen redesign"
/team-level "forest dungeon level"
/team-audio "combat audio pass"
```

每个团队技能协调 6 阶段协作工作流：
1. **设计** — game-designer 提问，展示选项
2. **架构** — lead-programmer 提出代码结构
3. **并行实现** — 专家同时工作
4. **集成** — gameplay-programmer 连接一切
5. **验证** — qa-tester 根据验收标准运行
6. **报告** — 协调员总结状态

协调是自动化的，但**决策点由你决定**。

### 5.6：冲刺审查和下一个冲刺

在一个冲刺结束时：

```
/retrospective
```

分析计划与完成、速度、阻碍和可操作的改进。

然后规划下一个冲刺：

```
/sprint-plan new
```

### 5.7：里程碑审查

在里程碑检查点：

```
/milestone-review "alpha"
```

生成功能完整性、质量指标、风险评估和 go/no-go 建议。

### 第五阶段关卡

```
/gate-check production
```

**通过要求：**

- 所有 MVP 故事完成
- 测试：3 次会话覆盖新玩家、中期游戏和难度曲线
- 乐趣假设已验证
- 测试数据中没有困惑循环

---

## 第六阶段：打磨

### 这个阶段做什么

你的游戏功能完整了。现在你要让它变好。这个阶段专注于性能、平衡、无障碍、音频、视觉打磨和测试。

### 第六阶段流程

```
/perf-profile  -->  /balance-check  -->  /asset-audit  -->  /playtest-report (x3)
       |                  |                    |                    |
       v                  v                    v                    v
  分析 CPU/GPU      分析公式和数据       验证命名、           覆盖：新玩家、
  内存，优化         是否有断裂的进度       格式、尺寸           中期游戏、难度曲线

  /tech-debt  -->  /team-polish
       |                |
       v                v
  跟踪和          协调打磨：
  优先排序        性能 + 美术 +
  债务项          音频 + UX + QA
```

### 6.1：性能分析

```
/perf-profile
```

引导你完成结构化性能分析：
- 建立目标（FPS、内存、平台）
- 按影响排名识别瓶颈
- 生成带有代码位置和预期收益的可操作优化任务

### 6.2：平衡分析

```
/balance-check assets/data/combat_damage.json
```

分析平衡数据中的统计异常值、断裂的进度曲线、堕落策略和经济失衡。

### 6.3：资源审计

```
/asset-audit
```

验证所有资源的命名约定、文件格式标准和尺寸预算。

### 6.4：测试（必需：3 次会话）

```
/playtest-report
```

生成结构化测试报告。需要三次会话，覆盖：
- 新玩家体验
- 中期游戏系统
- 难度曲线

### 6.5：技术债务评估

```
/tech-debt
```

扫描 TODO/FIXME/HACK 注释、代码重复、过度复杂的函数、缺失测试和过时依赖。每个项目分类和优先排序。

### 6.6：协调打磨

```
/team-polish "combat system"
```

并行协调 4 个专家：
1. 性能优化（performance-analyst）
2. 视觉打磨（technical-artist）
3. 音频打磨（sound-designer）
4. 手感/多汁（gameplay-programmer + technical-artist）

你设置优先级；团队在你的每个步骤批准下执行。

### 6.7：本地化和无障碍

```
/localize src/
```

扫描硬编码字符串、破坏翻译的连接、不考虑扩展的文本和缺失的 locale 文件。

无障碍根据第三阶段承诺的无障碍需求文档中的层级进行审计。

### 第六阶段关卡

```
/gate-check polish
```

**通过要求：**

- 至少 3 个测试报告存在
- 协调打磨完成（`/team-polish`）
- 没有阻塞性性能问题
- 无障碍层级要求已满足

---

## 第七阶段：发布

### 这个阶段做什么

你的游戏已打磨、测试完毕，准备好了。现在你要发布它。

### 第七阶段流程

```
/release-checklist  -->  /launch-checklist  -->  /team-release
        |                       |                      |
        v                       v                      v
  发布前验证：           完整跨部门             协调：
  代码、内容、           验证（每个部门         构建、QA 签字、
  商店、法律             的 Go/No-Go）          部署、发布
                    同时：/changelog、/patch-notes、/hotfix
```

### 7.1：发布清单

```
/release-checklist v1.0.0
```

生成全面的发布前清单，涵盖：
- 构建验证（所有平台编译并运行）
- 认证要求（平台特定）
- 商店元数据（描述、截图、预告片）
- 法律合规（EULA、隐私政策、评级）
- 保存游戏兼容性
- 分析验证

### 7.2：发布就绪（完整验证）

```
/launch-checklist
```

完整的跨部门验证：

| 部门 | 检查内容 |
|------|---------|
| **工程** | 构建稳定性、崩溃率、内存泄漏、加载时间 |
| **设计** | 功能完整性、教程流程、难度曲线 |
| **美术** | 资源质量、缺失纹理、LOD 级别 |
| **音频** | 缺失声音、混音级别、空间音频 |
| **QA** | 按严重程度划分的开放 bug 数量、回归套件通过率 |
| **叙事** | 对话完整性、世界观一致性、拼写错误 |
| **本地化** | 所有字符串已翻译、无截断、locale 测试 |
| **无障碍** | 合规清单、辅助功能测试 |
| **商店** | 元数据完整、截图批准、定价设置 |
| **营销** | 新闻资料包就绪、发布预告片、社交媒体排期 |
| **社区** | 补丁说明草稿、FAQ 已准备、支持渠道就绪 |
| **基础设施** | 服务器已扩展、CDN 已配置、监控已激活 |
| **法务** | EULA 已定稿、隐私政策、COPPA/GDPR 合规 |

每个项目获得 **Go / No-Go** 状态。所有项目必须为 Go 才能发布。

### 7.3：生成玩家面向内容

```
/patch-notes v1.0.0
```

从 git 历史和冲刺数据生成面向玩家的补丁说明。将开发者语言翻译为玩家语言。

```
/changelog v1.0.0
```

生成内部变更日志（更技术化，面向团队）。

### 7.4：协调发布

```
/team-release
```

协调 release-manager、QA 和 DevOps 完成：
1. 发布前验证
2. 构建管理
3. 最终 QA 签字
4. 部署准备
5. Go/No-Go 决策

### 7.5：发布

当推送到 `main` 或 `develop` 时，`validate-push` 钩子会警告你。这是故意的——发布推送应该是深思熟虑的：

```bash
git tag v1.0.0
git push origin main --tags
```

### 7.6：发布后

关键生产 bug 的**热修复工作流**：

```
/hotfix "Players losing save data when inventory exceeds 99 items"
```

绕过正常冲刺流程，保留完整审计跟踪：
1. 创建热修复分支
2. 实现修复
3. 确保回退到开发分支
4. 记录事件

发布稳定后进行**复盘**：

```
让 Claude 使用 .claude/docs/templates/post-mortem.md 中的模板创建复盘
```

---

## 跨领域关注点

这些主题适用于所有阶段。

### 总监审查模式

总监关卡是在关键工作流步骤审查你工作的专家智能体。默认情况下，它们在每个检查点运行。你可以控制你获得多少审查。

**在 `/start` 期间设置你的审查强度。** 保存到 `production/review-mode.txt`。

| 模式 | 运行什么 | 适合场景 |
|------|---------|---------|
| `full` | 每个步骤的所有总监关卡 | 新项目、学习系统 |
| `lean` | 仅在阶段转换时（`/gate-check`）运行总监 | 有经验的开发者 |
| `solo` | 无总监审查 | 游戏 jam、原型、最大速度 |

**为单次运行覆盖**而不改变你的全局设置：

```
/brainstorm space horror --review full
/architecture-decision --review solo
```

`--review` 标志适用于所有使用关卡的技能。随时通过直接编辑 `production/review-mode.txt` 或重新运行 `/start` 来更改全局模式。

完整关卡定义和检查模式：`.claude/docs/director-gates.md`

---

### 协作协议

这个系统是**用户驱动的协作**，不是自主的。

**模式：** 问题 > 选项 > 决策 > 起草 > 批准

每个智能体交互都遵循这个模式：
1. 智能体提出澄清问题
2. 智能体展示 2-4 个带权衡和理由的选项
3. 你决定
4. 智能体根据你的决定起草
5. 你审查和精炼
6. 智能体在写入之前问"可以写入 [filepath] 吗？"

参见 `docs/COLLABORATIVE-DESIGN-PRINCIPLE.md` 了解带示例的完整协议。

### AskUserQuestion 工具

智能体使用 `AskUserQuestion` 工具进行结构化选项展示。模式是解释然后捕获：首先在对话文本中完整分析，然后是清晰的 UI 选择器供决策。将其用于设计选择、架构决策和战略问题。不要将其用于开放式发现问题或简单的肯定/否定确认。

### 智能体协调（三层等级）

```
第一层（总监）：    creative-director, technical-director, producer
                                          |
第二层（主管）：    game-designer, lead-programmer, art-director,
                   audio-director, narrative-director, qa-lead,
                   release-manager, localization-lead
                                          |
第三层（专家）：    gameplay-programmer, engine-programmer,
                   ai-programmer, network-programmer, ui-programmer,
                   tools-programmer, systems-designer, level-designer,
                   economy-designer, world-builder, writer,
                   technical-artist, sound-designer, ux-designer,
                   qa-tester, performance-analyst, devops-engineer,
                   analytics-engineer, accessibility-specialist,
                   live-ops-designer, prot