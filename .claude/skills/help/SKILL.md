---
name: help
description: "分析已完成的工作和用户的疑问，提供下一步建议。当用户说接下来该怎么办、现在做什么、卡住了、或不知道该做什么时使用。"
argument-hint: "[optional: 你刚完成的工作，例如 'finished design-review' 或 'stuck on ADRs']"
user-invocable: true
allowed-tools: Read, Glob, Grep
context: |
  !echo "=== Live Project State ===" && echo "Stage: $(cat production/stage.txt 2>/dev/null | tr -d '[:space:]' || echo 'not set')" && echo "Latest sprint: $(ls -t production/sprints/*.md 2>/dev/null | head -1 || echo 'none')" && echo "Session state: $(head -5 production/session-state/active.md 2>/dev/null || echo 'none')"
model: haiku
---

# 工作室助手——下一步做什么？

此技能是只读的——它报告发现但不写入文件。

此技能能准确判断你在游戏开发管线中的位置，并告诉你接下来该做什么。它是**轻量级**的——不是全面审计。要做全面缺口分析，使用 `/project-stage-detect`。

---

## 步骤 1：读取目录

读取 `.claude/docs/workflow-catalog.yaml`。这是所有阶段、它们的步骤（按顺序）、每个步骤是必需还是可选、以及表示完成的制品 glob 的权威列表。

---

## 步骤 1b：查找目录中未列出的技能

读取目录后，Glob `.claude/skills/*/SKILL.md` 获取已安装技能的完整列表。对每个文件，从 frontmatter 中提取 `name:` 字段。

与目录中的 `command:` 值进行比较。任何名称未作为目录命令出现的技能都是**未编入目录的技能**——仍然可用，但不是阶段门控工作流的一部分。

将这些收集到步骤 7 的输出中——将它们显示为页脚块：

```
### Also installed (not in workflow)
- `/skill-name` — [description from SKILL.md frontmatter]
- `/skill-name` — [description]
```

仅在至少有一个未编入目录的技能时才显示此块。根据用户当前阶段限制为最相关的 10 个（生产中的 QA 技能、团队技能在生产/打磨中等）。

---

## 步骤 2：确定当前阶段

按此顺序检查：

1. **读取 `production/stage.txt`** — 如果存在且有内容，这是权威的阶段名称。将其映射到目录阶段键：
   - "Concept" → `concept`
   - "Systems Design" → `systems-design`
   - "Technical Setup" → `technical-setup`
   - "Pre-Production" → `pre-production`
   - "Production" → `production`
   - "Polish" → `polish`
   - "Release" → `release`

2. **如果 stage.txt 缺失**，从制品推断阶段（最晚完成的匹配胜出）：
   - `src/` 有 10+ 个源文件 → `production`
   - `production/stories/*.md` 存在 → `pre-production`
   - `docs/architecture/adr-*.md` 存在 → `technical-setup`
   - `design/gdd/systems-index.md` 存在 → `systems-design`
   - `design/gdd/game-concept.md` 存在 → `concept`
   - 什么都没有 → `concept`（全新项目）

---

## 步骤 3：读取会话上下文

如果存在，读取 `production/session-state/active.md`。提取：
- 最近处理的工作
- 任何进行中的任务或开放问题
- 当前 epic/feature/task 来自 STATUS 块（如果存在）

这告诉你用户刚完成什么或卡在哪里——用它来个性化输出。

---

## 步骤 4：检查当前阶段的步骤完成情况

对于当前阶段中的每个步骤（来自目录）：

### 基于制品的检查

如果步骤有 `artifact.glob`：
- 使用 Glob 检查是否有匹配模式的文件
- 如果指定了 `min_count`，验证至少匹配该数量的文件
- 如果指定了 `artifact.pattern`，使用 Grep 验证模式在匹配的文件中存在
- **完成** = 制品条件满足
- **未完成** = 制品缺失或模式未找到

如果步骤有 `artifact.note`（没有 glob）：
- 标记为 **MANUAL** — 无法自动检测，将询问用户

如果步骤没有 `artifact` 字段：
- 标记为 **UNKNOWN** — 完成状态不可追踪（例如可重复的实施工作）

### 特殊情况：production 阶段 — 读取 `sprint-status.yaml`

当当前阶段是 `production` 时，在做任何基于 glob 的故事检查之前，检查 `production/sprint-status.yaml` 是否存在。如果存在，直接读取：

- 状态为 `status: in-progress` 的故事 → 浮出为"当前进行中"
- 状态为 `status: ready-for-dev` 的故事 → 浮出为"下一个"
- 状态为 `status: done` 的故事 → 计为完成
- 状态为 `status: blocked` 的故事 → 以 `blocker` 字段浮出为阻碍

这提供了精确的按故事状态，而非 markdown 扫描。跳过 `implement` 和 `story-done` 步骤的 glob 制品检查——YAML 是权威的。

### 特殊情况：`repeatable: true`（非生产阶段）

对于生产阶段之外的可重复步骤（例如"系统 GDD"），制品检查告诉你是否有*任何*工作已完成，而非是否完成。将检测到的情况显示出来，然后注明可能正在进行中。

---

## 步骤 5：找到位置并识别下一步

从完成数据中确定：

1. **最后确认完成的步骤** — 最远的已完成必需步骤
2. **当前阻碍** — 第一个未完成的*必需*步骤（这是用户下一步必须做的）
3. **可选机会** — 可在阻碍之前或同时完成的未完成*可选*步骤
4. **即将到来的必需步骤** — 阻碍之后的必需步骤（显示为"即将到来"以便用户规划）

如果用户提供了参数（例如"刚完成 design-review"），用它跳过用户命名的步骤，即使制品检查不明确。

---

## 步骤 6：检查进行中的工作

如果 `active.md` 显示有活跃任务或 epic：
- 在顶部突出显示："看起来你在做 [X]"
- 建议继续它或确认是否完成

---

## 步骤 7：呈现输出

保持**简短直接**。这是快速定位，不是报告。

```
## 你在哪里：[阶段标签]

**进行中：** [来自 active.md，如果有的话]

### ✓ 已完成
- [已完成步骤名称]
- [已完成步骤名称]

### → 下一个（必需）
**[步骤名称]** — [描述]
命令：`[/command]`

### ~ 也有空做（可选）
- **[步骤名称]** — [描述] → `/command`
- **[步骤名称]** — [描述] → `/command`

### 之后会到来
- [下一个必需步骤名称] (`/command`)
- [下一个必需步骤名称] (`/command`)

---
快到 **[当前] → [下一个]** 门禁了 → 准备就绪时运行 `/gate-check`。
```

**格式规则：**
- `✓` 表示确认完成
- `→` 表示当前必需的下一个步骤（只有一个——第一个阻碍）
- `~` 表示现在可用的可选步骤
- 命令以内联代码形式显示
- 如果步骤没有命令（例如"实施故事"），解释该怎么做而非显示斜杠命令
- 对于 MANUAL 步骤，询问用户："我无法判断 [步骤] 是否完成——它完成了吗？"

裁决：**COMPLETE**——下一步已识别。

---

## 步骤 8：门禁警告（如果接近）

在当前阶段的步骤之后，检查用户是否可能接近门禁：
- 如果当前阶段的所有必需步骤都完成（或接近完成），添加："你快到 **[当前] → [下一个]** 门禁了。准备就绪时运行 `/gate-check`。"
- 如果还有多个必需步骤未完成，跳过门禁警告——现在还不相关。

---

## 步骤 9：升级路径

建议之后，如果用户似乎卡住或困惑，添加：

```
---
需要更多细节？
- `/project-stage-detect` — 完整缺口分析，列出所有缺失的制品
- `/gate-check` — 下一阶段的正式就绪检查
- `/start` — 从头重新定位
```

仅在用户的输入表明困惑时显示（例如"我不知道"、"卡住了"、"迷路了"、"不确定"）。对于简单的"接下来是什么？"不要显示。

---

## 协作协议

- **永远不要自动运行下一个技能。** 推荐它，让用户调用它。
- **对于 MANUAL 步骤询问用户**，而非假设完成或未完成。
- **匹配用户的语气**——如果他们听起来压力大（"我完全迷路了"），要安抚他们并给出一个行动，而非六个。
- **一个主要建议**——用户离开时应该确切知道下一步做什么。可选步骤和"即将到来"是次要上下文。
