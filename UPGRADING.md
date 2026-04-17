# 升级 Claude Code Game Studios

本指南涵盖如何将你现有的游戏项目仓库从模板的一个版本升级到下一个版本。

**查看当前版本**：在 git log 中查找：
```bash
git log --oneline | grep -i "release\|setup"
```
或在 `README.md` 中查看版本徽章。

---

## 目录

- [升级策略](#升级策略)
- [v0.4.x → v1.0](#v04x--v10)
- [v0.4.0 → v0.4.1](#v040--v041)
- [v0.3.0 → v0.4.0](#v030--v040)
- [v0.2.0 → v0.3.0](#v020--v030)
- [v0.1.0 → v0.2.0](#v010--v020)

---

## 升级策略

有三种方式拉取模板更新。根据你的仓库设置方式选择。

### 策略 A — Git 远程合并（推荐）

适用于：你克隆了模板并在上面有自己的提交。

```bash
# 将模板添加为远程（一次性设置）
git remote add template https://github.com/Donchitos/Claude-Code-Game-Studios.git

# 获取新版本
git fetch template main

# 合并到你的分支
git merge template/main --allow-unrelated-histories
```

Git 只会标记同时被模板和你修改过的文件的冲突。逐个解决——你的游戏内容保留，结构性改进一起进来。然后提交合并。

**提示：** 最可能冲突的文件是 `CLAUDE.md` 和 `.claude/docs/technical-preferences.md`，因为你已经填入了引擎和项目设置。保留你的内容；接受结构性变更。

---

### 策略 B — 挑选特定提交

适用于：你只需要一个特定功能（例如只要新技能，不要完整更新）。

```bash
git remote add template https://github.com/Donchitos/Claude-Code-Game-Studios.git
git fetch template main

# 挑选你想要的特定提交
git cherry-pick <commit-sha>
```

每个版本的提交 SHA 在下面的版本部分列出。

---

### 策略 C — 手动复制文件

适用于：你没有用 git 设置模板（只是下载了 zip）。

1. 在你的仓库旁边下载或克隆新版本。
2. 直接复制"可安全覆盖"下列出的文件。
3. 对于"需仔细合并"下的文件，两边版本并排打开，手动合并结构性变更同时保留你的内容。

---

## v0.4.1

**发布日期：** 2026-04-02
**主要主题：** 美术方向整合、资源规格流水线

### 变更内容

| 类别 | 变更 |
|------|------|
| **新技能** | `/art-bible`——分节引导式视觉身份编写（9 个章节）。每节必需的艺术总监 Task 产生。AD-ART-BIBLE 签核关卡。在技术准备阶段必需。 |
| **新技能** | `/asset-spec`——每个资源的视觉规格和 AI 生成提示词生成器。读取美术圣经 + GDD/关卡/角色文档。写入 `design/assets/specs/` 文件和 `design/assets/asset-manifest.md`。完整/精简/独立模式。 |
| **新总监关卡（3 个）** | `AD-CONCEPT-VISUAL`（头脑风暴阶段 4）、`AD-ART-BIBLE`（美术圣经签核）、`AD-PHASE-GATE`（gate-check 面板） |
| **`/brainstorm` 更新** | 在 allowed-tools 中添加了 `Task`（之前缺失——阻止了所有总监产生）。艺术总监现在在核心支柱锁定后与创意总监并行产生。视觉身份锚点写入 game-concept.md。 |
| **`/gate-check` 更新** | 艺术总监作为第 4 个并行总监添加（AD-PHASE-GATE）。视觉产物检查：视觉身份锚点（概念关卡）、美术圣经（技术准备关卡）、AD-ART-BIBLE 签核 + 角色视觉档案（预制作关卡）。 |
| **`/team-level` 更新** | 艺术总监添加到第 1 步并行产生中（布局前的视觉方向）。关卡设计师现在接收艺术总监目标作为明确约束。第 4 步艺术总监角色修正为仅生产概念。 |
| **`/team-narrative` 更新** | 艺术总监添加到第 2 阶段并行产生中（角色视觉设计、环境叙事、电影基调）。 |
| **`/design-system` 更新** | 路由表扩展了艺术总监 + 技术美术，用于战斗、UI、对话、动画/VFX、角色类别。7 个系统类别的视觉/音频部分现为必需（包含艺术总监 Task 产生）。 |
| **`workflow-catalog.yaml`** | `/art-bible` 添加到技术准备（必需）。`/asset-spec` 添加到预制作（可选，可重复）。 |

### 文件：可安全覆盖

**新增文件：**
```
.claude/skills/art-bible/SKILL.md
.claude/skills/asset-spec/SKILL.md
.claude/docs/director-gates.md
```

**可覆盖的现有文件（无用户内容）：**
```
.claude/skills/brainstorm/SKILL.md
.claude/skills/gate-check/SKILL.md
.claude/skills/team-level/SKILL.md
.claude/skills/team-narrative/SKILL.md
.claude/skills/design-system/SKILL.md
.claude/docs/workflow-catalog.yaml
README.md
UPGRADING.md
```

### 文件：需仔细合并

无——所有变更都是对基础设施文件的修改，无用户内容。

---

## v0.4.x → v1.0

**发布日期：** 2026-03-29
**提交范围：** `6c041ac..HEAD`
**主要主题：** 总监关卡系统、关卡强度模式、Godot C# 专家

### 变更内容

| 类别 | 变更 |
|------|------|
| **新系统** | 总监关卡——跨所有工作流技能共享的命名评审检查点。定义在 `.claude/docs/director-gates.md` |
| **新功能** | 关卡强度模式：`full`（所有总监关卡）、`lean`（仅阶段关卡）、`solo`（无总监）。通过 `/start` 期间全局设置，写入 `production/review-mode.txt`，或在任意使用关卡的技能运行时用 `--review [mode]` 覆盖 |
| **新 Agent** | `godot-csharp-specialist`——Godot 4 项目中的 C# 代码质量 |
| **技能更新（13 个）** | 所有使用关卡的技能现在解析 `--review [full|lean|solo]` 并包含在它们的 argument-hint 中：`brainstorm`、`map-systems`、`design-system`、`architecture-decision`、`create-architecture`、`create-epics`、`create-stories`、`sprint-plan`、`milestone-review`、`playtest-report`、`prototype`、`story-done`、`gate-check` |
| **`/start` 更新** | 添加阶段 3b——在入职期间设置评审模式，写入 `production/review-mode.txt` |
| **`/setup-engine` 更新** | Godot 的语言选择步骤（GDScript vs C#） |
| **文档** | `director-gates.md`——完整关卡目录；`WORKFLOW-GUIDE.md`——总监评审模式部分；`README.md`——评审强度自定义 |

---

### 文件：可安全覆盖

**新增文件：**
```
.claude/agents/godot-csharp-specialist.md
.claude/docs/director-gates.md
```

**可覆盖的现有文件（无用户内容）：**
```
.claude/skills/brainstorm/SKILL.md
.claude/skills/map-systems/SKILL.md
.claude/skills/design-system/SKILL.md
.claude/skills/architecture-decision/SKILL.md
.claude/skills/create-architecture/SKILL.md
.claude/skills/create-epics/SKILL.md
.claude/skills/create-stories/SKILL.md
.claude/skills/sprint-plan/SKILL.md
.claude/skills/milestone-review/SKILL.md
.claude/skills/playtest-report/SKILL.md
.claude/skills/prototype/SKILL.md
.claude/skills/story-done/SKILL.md
.claude/skills/gate-check/SKILL.md
.claude/skills/start/SKILL.md
.claude/skills/quick-design/SKILL.md
.claude/skills/setup-engine/SKILL.md
README.md
docs/WORKFLOW-GUIDE.md
UPGRADING.md
```

---

### 文件：需仔细合并

此版本无需手动合并的文件。所有变更都是对基础设施文件的修改，无用户内容。

---

### 新功能

#### 总监关卡系统

所有主要工作流技能现在引用定义在 `.claude/docs/director-gates.md` 中的命名关卡检查点。关卡由领域前缀和名称标识（例如 `CD-CONCEPT`、`TD-ARCHITECTURE`、`LP-CODE-REVIEW`）。每个关卡定义要产生哪个总监、传递什么输入、什么裁决意味着什么，以及 lean/solo 模式如何影响它。

技能使用 `Task` 和关卡 ID 及文档化输入来产生关卡，而不是将总监提示内联在技能主体中。这保持了技能体简洁，并使关卡行为在所有工作流阶段保持一致。

#### 关卡强度模式

三种模式让你控制获取多少总监评审：

- **`full`**（默认）——每个评审检查点运行所有总监关卡
- **`lean`**——跳过每个技能的总监评审；`/gate-check` 的阶段关卡仍然运行
- **`solo`**——任何地方都没有总监关卡；`/gate-check` 仅检查产物存在性

在 `/start` 期间全局设置（写入 `production/review-mode.txt`）。在任意使用关卡的技能运行时用 `--review [mode]` 覆盖任何单独运行：

```
/design-system combat --review lean
/gate-check concept --review full
/brainstorm my-game-idea --review solo
```

---

### 升级后

1. 运行 `/start` 一次来设置你偏好的评审模式——或者手动创建 `production/review-mode.txt`，内容为 `full`、`lean` 或 `solo`。
2. 如果你正在项目中途，查看 `.claude/docs/director-gates.md` 了解哪些关卡适用于你当前阶段。
3. 运行 `/skill-test static all` 验证所有技能通过结构性检查。

---

## v0.4.0 → v0.4.1

**发布日期：** 2026-03-26
**提交范围：** `04ed5d5..HEAD`
**主要主题：** 通用领域 Agent、新技能、技能修复

### 变更内容

| 类别 | 变更 |
|------|------|
| **新技能（1）** | `/consistency-check`——跨 GDD 实体一致性扫描器 |
| **技能修复（所有 team-*）** | 添加无参数守卫、正式的 `Verdict: COMPLETE / BLOCKED` 关键词、每步 AskUserQuestion 关卡、邻近区域依赖检查（team-level）、道德 enforcement（team-live-ops）、带阶段跳过的 NO-GO 路径（team-release） |
| **Agent 修复（4 个）** | game-designer、systems-designer、economy-designer、live-ops-designer 的通用领域语言——移除了 RPG 特定术语 |

---

### 文件：可安全覆盖

**新增文件：**
```
.claude/skills/consistency-check/SKILL.md
```

**可覆盖的现有文件（无用户内容）：**
```
.claude/skills/team-combat/SKILL.md      ← 无参守卫、裁决关键词、关卡改进
.claude/skills/team-narrative/SKILL.md   ← 无参守卫、裁决关键词、关卡改进
.claude/skills/team-ui/SKILL.md          ← 无参守卫、裁决关键词、关卡改进
.claude/skills/team-release/SKILL.md     ← 无参守卫、裁决关键词、NO-GO 路径
.claude/skills/team-polish/SKILL.md      ← 无参守卫、裁决关键词、关卡改进
.claude/skills/team-audio/SKILL.md       ← 无参守卫、裁决关键词、关卡改进
.claude/skills/team-level/SKILL.md       ← 无参守卫、裁决关键词、邻近区域检查
.claude/skills/team-live-ops/SKILL.md    ← 无参守卫、裁决关键词、道德 enforcement
.claude/skills/team-qa/SKILL.md          ← 无参守卫、裁决关键词、关卡改进
.claude/skills/map-systems/SKILL.md      ← 裁决关键词
.claude/skills/create-epics/SKILL.md     ← "May I write" 协议修复、裁决关键词
.claude/skills/create-stories/SKILL.md   ← 裁决关键词
.claude/agents/game-designer.md          ← 通用领域语言
.claude/agents/systems-designer.md       ← 通用领域语言
.claude/agents/economy-designer.md       ← 通用领域语言
.claude/agents/live-ops-designer.md      ← 通用领域语言
```

---

### 文件：需仔细合并

此版本无需手动合并的文件。所有变更都是对基础设施文件的修改，无用户内容。

---

### 升级后

1. 运行 `/skill-test catalog` 验证所有技能已编入索引。
2. 任何技能编辑后运行 `/skill-test lint [skill-name]` 检查结构合规性。
3. 如果你自定义了任何 team-* 技能，查看更新后的版本——无参数守卫和 `Verdict:` 关键词现在是所有 team-* 技能必需的。

---

## v0.3.0 → v0.4.0

**发布日期：** 2026-03-21
**提交范围：** `b1cad29..HEAD`
**主要主题：** 完整 UX/UI 流水线、完整故事生命周期、棕地项目采用、全面 QA/测试框架、流水线完整性、29 个新技能

### 变更内容

| 类别 | 变更 |
|------|------|
| **新技能（17 个）** | `/ux-design`、`/ux-review`、`/help`、`/quick-design`、`/review-all-gdds`、`/story-readiness`、`/story-done`、`/sprint-status`、`/adopt`、`/create-architecture`、`/create-control-manifest`、`/create-epics`、`/create-stories`、`/dev-story`、`/propagate-design-change`、`/content-audit`、`/architecture-review` |
| **新 QA 技能（12 个）** | `/qa-plan`、`/smoke-check`、`/soak-test`、`/regression-suite`、`/test-setup`、`/test-helpers`、`/test-evidence-review`、`/test-flakiness`、`/skill-test`、`/bug-triage`、`/team-live-ops`、`/team-qa` |
| **新 Hooks（4 个）** | `log-agent-stop.sh`——Agent 审计跟踪停止；`notify.sh`——Windows toast 通知；`post-compact.sh`——压缩后会话恢复提醒；`validate-skill-change.sh`——技能编辑后建议 `/skill-test` |
| **新模板（8 个）** | `ux-spec.md`、`hud-design.md`、`accessibility-requirements.md`、`interaction-pattern-library.md`、`player-journey.md`、`difficulty-curve.md`，以及 2 个采用计划模板 |
| **新基础设施** | `workflow-catalog.yaml`（7 阶段流水线，由 `/help` 读取）、`docs/architecture/tr-registry.yaml`（稳定 TR-ID）、`production/sprint-status.yaml` schema |
| **技能更新** | `/gate-check`——3 个关卡现在需要 UX 产物；预制作关卡需要垂直切片（HARD 关卡） |
| **技能更新** | `/sprint-plan`——写入 `sprint-status.yaml`；`/sprint-status` 读取它 |
| **技能更新** | `/story-done`——8 阶段完成评审，更新故事文件，展示下一个就绪故事 |
| **技能更新** | `/design-review`——移除了架构差距检查（错误的阶段） |
| **技能更新** | `/team-ui`——完整 UX 流水线（ux-design → ux-review → 团队阶段） |
| **Agent 更新** | 14 个专家 Agent——添加了 `memory: project` |
| **Agent 更新** | `prototyper`——`isolation: worktree`（在隔离 git 分支中丢弃工作） |
| **模型路由** | Haiku/Sonnet/Opus 层分配记录在协调规则中；技能在前置元数据中声明它们的层 |
| **目录 CLAUDE.md** | 搭建了 `design/CLAUDE.md`、`src/CLAUDE.md`、`docs/CLAUDE.md`——每个目录的路径作用域指令 |
| **流水线完整性** | TR-ID 稳定性、清单版本控制、ADR 状态关卡、TR-ID 引用不引用 |
| **GDD 模板** | 添加了 `## Game Feel` 章节（输入响应性、动画目标、冲击瞬间） |

---

### 文件：可安全覆盖

**新增文件：**
```
.claude/skills/ux-design/SKILL.md
.claude/skills/ux-review/SKILL.md
.claude/skills/help/SKILL.md
.claude/skills/quick-design/SKILL.md
.claude/skills/review-all-gdds/SKILL.md
.claude/skills/story-readiness/SKILL.md
.claude/skills/story-done/SKILL.md
.claude/skills/sprint-status/SKILL.md
.claude/skills/adopt/SKILL.md
.claude/skills/create-architecture/SKILL.md
.claude/skills/create-control-manifest/SKILL.md
.claude/skills/create-epics/SKILL.md
.claude/skills/create-stories/SKILL.md
.claude/skills/dev-story/SKILL.md
.claude/skills/propagate-design-change/SKILL.md
.claude/skills/content-audit/SKILL.md
.claude/skills/architecture-review/SKILL.md
.claude/skills/qa-plan/SKILL.md
.claude/skills/smoke-check/SKILL.md
.claude/skills/soak-test/SKILL.md
.claude/skills/regression-suite/SKILL.md
.claude/skills/test-setup/SKILL.md
.claude/skills/test-helpers/SKILL.md
.claude/skills/test-evidence-review/SKILL.md
.claude/skills/test-flakiness/SKILL.md
.claude/skills/skill-test/SKILL.md
.claude/skills/bug-triage/SKILL.md
.claude/skills/team-live-ops/SKILL.md
.claude/skills/team-qa/SKILL.md
.claude/hooks/log-agent-stop.sh
.claude/hooks/notify.sh
.claude/hooks/post-compact.sh
.claude/hooks/validate-skill-change.sh
.claude/docs/workflow-catalog.yaml
.claude/docs/templates/ux-spec.md
.claude/docs/templates/hud-design.md
.claude/docs/templates/accessibility-requirements.md
.claude/docs/templates/interaction-pattern-library.md
.claude/docs/templates/player-journey.md
.claude/docs/templates/difficulty-curve.md
design/CLAUDE.md
src/CLAUDE.md
docs/CLAUDE.md
```

**可覆盖的现有文件（无用户内容）：**
```
.claude/skills/gate-check/SKILL.md
.claude/skills/sprint-plan/SKILL.md
.claude/skills/sprint-status/SKILL.md
.claude/skills/design-review/SKILL.md
.claude/skills/team-ui/SKILL.md
.claude/skills/story-readiness/SKILL.md
.claude/skills/story-done/SKILL.md
.claude/docs/templates/game-design-document.md    ← 添加 Game Feel 章节
README.md
docs/WORKFLOW-GUIDE.md
UPGRADING.md
```

**可覆盖的 Agent 文件**（如果你没有在其中编写自定义提示）：
```
.claude/agents/prototyper.md         ← 添加 isolation: worktree
.claude/agents/art-director.md       ← 添加 memory: project
.claude/agents/audio-director.md     ← 添加 memory: project
.claude/agents/economy-designer.md   ← 添加 memory: project
.claude/agents/game-designer.md      ← 添加 memory: project
.claude/agents/gameplay-programmer.md ← 添加 memory: project
.claude/agents/lead-programmer.md    ← 添加 memory: project
.claude/agents/level-designer.md     ← 添加 memory: project
.claude/agents/narrative-director.md ← 添加 memory: project
.claude/agents/systems-designer.md   ← 添加 memory: project
.claude/agents/technical-artist.md   ← 添加 memory: project
.claude/agents/ui-programmer.md      ← 添加 memory: project
.claude/agents/ux-designer.md        ← 添加 memory: project
.claude/agents/world-builder.md      ← 添加 memory: project
```

---

### 文件：需仔细合并

#### `.claude/settings.json`

此版本注册了四个新 hook。如果你没有自定义 `settings.json`，覆盖是安全的。否则，手动添加以下 hook 条目：

- `log-agent-stop.sh`——`SubagentStop` 事件（Agent 审计跟踪停止）
- `notify.sh`——`Notification` 事件（Windows toast 通知）
- `post-compact.sh`——`PostCompact` 事件（会话恢复提醒）
- `validate-skill-change.sh`——`PostToolUse` 事件，过滤到 `.claude/skills/` 写入

#### 自定义的 Agent 文件

如果你向 Agent `.md` 文件添加了项目特定知识，做一个 diff 并在 YAML 前置元数据中适当位置手动添加 `memory: project` 行。创意总监和技术总监 Agent 故意保留 `memory: user`——只有专家 Agent 获取 `memory: project`。

---

### 新功能

#### 完整故事生命周期

故事现在有正式的生命周期，由两个技能强制执行：

- **`/story-readiness`**——验证故事在开发者接手前是否已准备好实现。检查设计（GDD req 已链接）、架构（ADR 已接受）、范围（标准可测试）和 DoD（清单版本当前）。裁决：READY / NEEDS WORK / BLOCKED。
- **`/story-done`**——实现后的 8 阶段完成评审。验证每个验收标准，检查 GDD/ADR 偏差，提示代码审查，将故事文件更新为 `Status: Complete`，并展示下一个就绪故事。

流程：`/story-readiness` → 实现 → `/story-done` → 下一个故事

#### 完整 UX/UI 流水线

- **`/ux-design`**——分节引导式 UX 规格编写。三种模式：屏幕/流程、HUD 或交互模式库。读取 GDD UI 要求和玩家旅程。输出到 `design/ux/`。
- **`/ux-review`**——根据 GDD 一致性、无障碍层级和模式库验证 UX 规格。裁决：APPROVED / NEEDS REVISION / MAJOR REVISION。
- **`/team-ui` 更新：** 第一阶段现在在视觉设计开始前运行 `/ux-design` + `/ux-review` 作为硬关卡。

#### 棕地项目采用

**`/adopt`** 将现有项目纳入模板格式。审计 GDD、ADR、故事、系统索引和基础设施的内部结构。对差距进行分类（BLOCKING/HIGH/MEDIUM/LOW）。构建有序的迁移计划。永不重新生成现有产物——只填补差距。

参数模式：`full | gdds | adrs | stories | infra`

另外：`/design-system retrofit [path]` 和 `/architecture-decision retrofit [path]` 检测现有文件并仅添加缺失的章节。

#### Sprint 追踪 YAML

`production/sprint-status.yaml` 现在是权威的故事追踪格式：
- 由 `/sprint-plan` 写入（初始化所有故事）和 `/story-done` 写入（设置状态为 `done`）
- 由 `/sprint-status` 读取（快速快照）和 `/help` 读取（生产阶段每个故事的状态）
- 状态值：`backlog | ready-for-dev | in-progress | review | done | blocked`
- 如果文件不存在优雅降级到 markdown 扫描

#### `/help`——上下文感知下一步

`/help` 读取你当前阶段和进行中的工作，检查哪些产物已完成，并准确告诉你下一步做什么——一个主要必需步骤，加上可选机会。区别于 `/start`（仅首次）和 `/project-stage-detect`（完整审计）。

#### 全面的 QA 和测试框架

涵盖完整测试生命周期的九个新 QA/测试技能：

- **`/test-setup`**——为你的引擎搭建测试框架和 CI/CD 流水线
- **`/test-helpers`**——生成引擎特定的测试辅助库（GDUnit4、NUnit 等）
- **`/qa-plan`**——为 sprint 或功能生成 QA 测试计划，按测试类型对故事分类
- **`/smoke-check`**——在 QA 交接前运行关键路径冒烟测试关卡
- **`/soak-test`**——为长时间游戏会话生成浸泡测试协议（稳定性、内存泄漏）
- **`/regression-suite`**——将测试覆盖映射到 GDD 关键路径，识别缺乏回归测试的已修复 bug
- **`/test-evidence-review`**——测试文件和手动证据文档的质量评审
- **`/test-flakiness`**——通过读取 CI 运行日志检测非确定性测试
- **`/skill-test`**——验证技能文件的结构合规性和行为正确性（三种模式：lint、spec、catalog）

另外新增加：**`/bug-triage`** 重新评估所有开放 bug 的优先级、严重性和所有权。

#### 技能验证器（`/skill-test`）

`/skill-test` 是一个验证工具本身的元技能。在编辑任何技能文件后运行它。三种模式：
- `lint`——验证 YAML 前置元数据和必需字段
- `spec [skill-name]`——针对特定技能运行行为规范测试
- `catalog`——检查 `.claude/skills/` 中的所有技能是否已编入索引

新的 `validate-skill-change.sh` hook 会在技能文件被修改时自动提醒你运行 `/skill-test`。

#### 团队 Live-Ops 和团队 QA 编排

- **`/team-live-ops`**——协调 live-ops-designer + economy-designer + community-manager + analytics-engineer 进行发布后内容规划（季活动、战斗通行证、留存）
- **`/team-qa`**——编排 qa-lead + qa-tester + gameplay-programmer + producer 完成完整 QA 周期：策略、执行、覆盖和签核

#### 模型层路由

技能现在根据任务复杂度显式分配到 Haiku、Sonnet 或 Opus 层。只读状态检查使用 Haiku；复杂多文档综合使用 Opus；其他所有默认 Sonnet。层分配记录在 `.claude/docs/coordination-rules.md` 中。

#### 目录 CLAUDE.md 文件

三个新的目录作用域 CLAUDE.md 文件（`design/`、`src/`、`docs/`）为在这些目录中工作的 Agent 提供路径特定指令。这些在 Claude Code 读取该目录中的文件时自动加载。

---

### 升级后

1. **验证新 hooks** 已在 `.claude/settings.json` 中注册——检查所有四个：`log-agent-stop.sh`、`notify.sh`、`post-compact.sh`、`validate-skill-change.sh`。

2. **测试审计跟踪**——通过产生任何子 Agent 来测试——开始和停止事件都应出现在 `production/session-logs/` 中。

3. **如果你处于活跃生产中**，生成 sprint-status.yaml：
   ```
   /sprint-plan status
   ```

4. **如果你有早于此模板版本存在的 GDD 或 ADR**，运行 `/adopt`——它将识别需要添加哪些章节而不覆盖你的内容。

5. **任何技能编辑后**用 `/skill-test` 验证你的技能——新的 `validate-skill-change.sh` hook 会自动提醒你这样做。

---

## v0.2.0 → v0.3.0

**发布日期：** 2026-03-09
**提交范围：** `e289ce9..HEAD`
**主要主题：** `/design-system` GDD 编写、`/map-systems` 重命名、自定义状态行

### 破坏性变更

#### `/design-systems` 重命名为 `/map-systems`

`/design-systems` 技能被重命名为 `/map-systems` 以提高清晰度（分解 = *映射*，不是 *设计*）。

**需要采取的行动：** 更新任何调用 `/design-systems` 的文档、笔记或脚本。新的调用是 `/map-systems`。

### 变更内容

| 类别 | 变更 |
|------|------|
| **新技能** | `/design-system`（引导式 GDD 编写，分节进行） |
| **重命名的技能** | `/design-systems` → `/map-systems`（破坏性重命名） |
| **新文件** | `.claude/statusline.sh`、`.claude/settings.json` statusline 配置 |
| **技能更新** | `/gate-check`——在 PASS 时写入 `production/stage.txt`，新阶段定义 |
| **技能更新** | `brainstorm`、`start`、`design-review`、`project-stage-detect`、`setup-engine`——交叉引用修复 |
| **Bug 修复** | `log-agent.sh`、`validate-commit.sh`——hook 执行修复 |
| **文档** | 添加 `UPGRADING.md`，更新 `README.md`，更新 `WORKFLOW-GUIDE.md` |

---

### 文件：可安全覆盖

**新增文件：**
```
.claude/skills/design-system/SKILL.md
.claude/statusline.sh
```

**可覆盖的现有文件（无用户内容）：**
```
.claude/skills/map-systems/SKILL.md      ← 原 design-systems/SKILL.md
.claude/skills/gate-check/SKILL.md
.claude/skills/brainstorm/SKILL.md
.claude/skills/start/SKILL.md
.claude/skills/design-review/SKILL.md
.claude/skills/project-stage-detect/SKILL.md
.claude/skills/setup-engine/SKILL.md
.claude/hooks/log-agent.sh
.claude/hooks/validate-commit.sh
README.md
docs/WORKFLOW-GUIDE.md
UPGRADING.md
```

**删除（被重命名取代）：**
```
.claude/skills/design-systems/   ← 整个目录；被 map-systems/ 取代
```

---

### 文件：需仔细合并

#### `.claude/settings.json`

新版本添加了指向 `.claude/statusline.sh` 的 `statusLine` 配置块。如果你没有自定义 `settings.json`，覆盖是安全的。否则，手动添加此块：

```json
"statusLine": {
  "script": ".claude/statusline.sh"
}
```

---

### 新功能

#### 自定义状态行

`.claude/statusline.sh` 在终端状态行显示 7 阶段生产流水线面包屑：

```
ctx: 42% | claude-sonnet-4-6 | Systems Design
```

在 Production/Polish/Release 阶段，如果存在 `<!-- STATUS -->` 块，它还显示来自 `production/session-state/active.md` 的活动 Epic/Feature/Task：

```
ctx: 42% | claude-sonnet-4-6 | Production | Combat System > Melee Combat > Hitboxes
```

当前阶段从项目产物自动检测，也可以通过将阶段名称写入 `production/stage.txt` 来固定。

#### `/gate-check` 阶段推进

当关卡 PASS 裁决确认时，`/gate-check` 现在将新阶段名称写入 `production/stage.txt`。这会立即为所有未来会话更新状态行，无需手动文件编辑。

---

### 升级后

1. **删除旧的技能目录：**
   ```bash
   rm -rf .claude/skills/design-systems/
   ```

2. **测试状态行**——通过启动 Claude Code 会话来测试——你应该在终端页脚看到阶段面包屑。

3. **验证 hook 执行**仍然有效：
   ```bash
   bash .claude/hooks/log-agent.sh '{}' '{}'
   bash .claude/hooks/validate-commit.sh '{}' '{}'
   ```

---

## v0.1.0 → v0.2.0

**发布日期：** 2026-02-21
**提交范围：** `ad540fe..e289ce9`
**主要主题：** 上下文弹性、AskUserQuestion 集成、`/map-systems` 技能

### 变更内容

| 类别 | 变更 |
|------|------|
| **新技能** | `/start`（入职）、`/map-systems`（系统分解）、`/design-system`（引导式 GDD 编写） |
| **新 Hooks** | `session-start.sh`（恢复）、`detect-gaps.sh`（差距检测） |
| **新模板** | `systems-index.md`、3 个协作协议模板 |
| **上下文管理** | 主要重写——添加了文件支持的状态策略 |
| **Agent 更新** | 14 个设计/创意 Agent——AskUserQuestion 集成 |
| **技能更新** | 所有 7 个 `team-*` 技能 + `brainstorm`——在阶段转换时 AskUserQuestion |
| **CLAUDE.md** | 从约 159 行精简到约 60 行；5 个文档导入而不是 10 个 |
| **Hook 更新** | 所有 8 个 hooks——Windows 兼容性修复、新功能 |
| **文档删除** | `docs/IMPROVEMENTS-PROPOSAL.md`、`docs/MULTI-STAGE-DOCUMENT-WORKFLOW.md` |

---

### 文件：可安全覆盖

这些是纯基础设施——你没有自定义它们。直接复制新版本，没有项目内容风险。

**新增文件：**
```
.claude/skills/start/SKILL.md
.claude/skills/map-systems/SKILL.md
.claude/skills/design-system/SKILL.md
.claude/docs/templates/systems-index.md
.claude/docs/templates/collaborative-protocols/design-agent-protocol.md
.claude/docs/templates/collaborative-protocols/implementation-agent-protocol.md
.claude/docs/templates/collaborative-protocols/leadership-agent-protocol.md
.claude/hooks/detect-gaps.sh
.claude/hooks/session-start.sh
production/session-state/.gitkeep
docs/examples/README.md
.github/ISSUE_TEMPLATE/bug_report.md
.github/ISSUE_TEMPLATE/feature_request.md
.github/PULL_REQUEST_TEMPLATE.md
```

**可覆盖的现有文件（无用户内容）：**
```
.claude/skills/brainstorm/SKILL.md
.claude/skills/design-review/SKILL.md
.claude/skills/gate-check/SKILL.md
.claude/skills/project-stage-detect/SKILL.md
.claude/skills/setup-engine/SKILL.md
.claude/skills/team-audio/SKILL.md
.claude/skills/team-combat/SKILL.md
.claude/skills/team-level/SKILL.md
.claude/skills/team-narrative/SKILL.md
.claude/skills/team-polish/SKILL.md
.claude/skills/team-release/SKILL.md
.claude/skills/team-ui/SKILL.md
.claude/hooks/log-agent.sh
.claude/hooks/pre-compact.sh
.claude/hooks/session-stop.sh
.claude/hooks/validate-assets.sh
.claude/hooks/validate-commit.sh
.claude/hooks/validate-push.sh
.claude/rules/design-docs.md
.claude/docs/hooks-reference.md
.claude/docs/skills-reference.md
.claude/docs/quick-start.md
.claude/docs/directory-structure.md
.claude/docs/context-management.md
docs/COLLABORATIVE-DESIGN-PRINCIPLE.md
docs/WORKFLOW-GUIDE.md
README.md
```

**可覆盖的 Agent 文件**（如果你没有在其中编写自定义提示）：
```
.claude/agents/art-director.md
.claude/agents/audio-director.md
.claude/agents/creative-director.md
.claude/agents/economy-designer.md
.claude/agents/game-designer.md
.claude/agents/level-designer.md
.claude/agents/live-ops-designer.md
.claude/agents/narrative-director.md
.claude/agents/producer.md
.claude/agents/systems-designer.md
.claude/agents/technical-director.md
.claude/agents/ux-designer.md
.claude/agents/world-builder.md
.claude/agents/writer.md
```

如果你*已经*自定义了 Agent 提示，请参阅下面的"需仔细合并"。

---

### 文件：需仔细合并

这些文件同时包含模板结构和你的项目特定内容。**不要**覆盖它们——手动合并变更。

#### `CLAUDE.md`

模板版本从约 159 行精简到约 60 行。关键结构变更：移除了 5 个 doc 导入，因为 Claude Code 会自动加载它们（agent-roster、skills-reference、hooks-reference、rules-reference、review-workflow）。

**从你的版本中保留：**
- `## Technology Stack` 章节（你的引擎/语言选择）
- 你做的任何项目特定添加

**从新版本中采用：**
- 更精简的导入列表（如果存在，删除 5 个冗余的 `@` 导入）
- 更新的协作协议措辞

#### `.claude/docs/technical-preferences.md`

如果你运行过 `/setup-engine`，这个文件有你的引擎配置、命名约定和性能预算。保留所有内容。模板版本只是空的占位符。

#### `.claude/docs/templates/game-concept.md`

小的结构更新——添加了指向 `/map-systems` 的 `## Next Steps` 章节。如果你想要更新的指导，将该章节添加到你的副本，但不是必需的。

#### `.claude/settings.json`

检查新版本是否添加了你想要的任何权限规则。变更很小（schema 更新）。如果你没有自定义 `settings.json`，覆盖是安全的。

#### 自定义的 Agent 文件

如果你向任何 Agent `.md` 文件添加了项目特定知识或自定义行为，做一个 diff 并手动添加新的 AskUserQuestion 集成部分，而不是覆盖。每个 Agent 的变更是系统提示末尾标准化的