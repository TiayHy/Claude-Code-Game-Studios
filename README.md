# Claude Code Game Studios

<p align="center">
  <h1 align="center">Claude Code Game Studios</h1>
  <p align="center">
    将一个单独的 Claude Code 会话变成一个完整的游戏开发工作室。
    <br />
    49 个 Agent。72 个技能。一支协调的 AI 团队。
  </p>
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License"></a>
  <a href=".claude/agents"><img src="https://img.shields.io/badge/agents-49-blueviolet" alt="49 Agents"></a>
  <a href=".claude/skills"><img src="https://img.shields.io/badge/skills-72-green" alt="72 Skills"></a>
  <a href=".claude/hooks"><img src="https://img.shields.io/badge/hooks-12-orange" alt="12 Hooks"></a>
  <a href=".claude/rules"><img src="https://img.shields.io/badge/rules-11-red" alt="11 Rules"></a>
  <a href="https://docs.anthropic.com/en/docs/claude-code"><img src="https://img.shields.io/badge/built%20for-Claude%20Code-f5f5f5?logo=anthropic" alt="Built for Claude Code"></a>
  <a href="https://www.buymeacoffee.com/donchitos3"><img src="https://img.shields.io/badge/Buy%20Me%20a%20Coffee-Support%20this%20project-FFDD00?logo=buymeacoffee&logoColor=black" alt="Buy Me a Coffee"></a>
  <a href="https://github.com/sponsors/Donchitos"><img src="https://img.shields.io/badge/GitHub%20Sponsors-Support%20this%20project-ea4aaa?logo=githubsponsors&logoColor=white" alt="GitHub Sponsors"></a>
</p>

---

## 为什么存在这个项目

用 AI 独立开发游戏很强大——但单个聊天会话没有结构。没有人会阻止你硬编码魔法数字、跳过设计文档、或写出意大利面条式代码。没有 QA 审查、没有设计评审、也没有人问"这真的符合游戏的愿景吗？"

**Claude Code Game Studios** 通过给你的 AI 会话提供真实工作室的结构来解决这个问题。不再是单一通用助手，而是 49 个专业化 Agent 按照工作室层级组织——守护愿景的总监、管理各自领域的主管、以及从事实际工作的专家。每个 Agent 都有明确的职责、升级路径和质量关卡。

结果是：你仍然做出每一个决定，但现在你有一个团队会问正确的问题、在早期发现错误、并从第一次头脑风暴到发布都保持项目井然有序。

---

## 目录

- [包含内容](#包含内容)
- [工作室层级](#工作室层级)
- [斜杠命令](#斜杠命令)
- [快速开始](#快速开始)
- [升级指南](#升级指南)
- [项目结构](#项目结构)
- [工作原理](#工作原理)
- [设计理念](#设计理念)
- [自定义](#自定义)
- [平台支持](#平台支持)
- [社区](#社区)
- [支持这个项目](#支持这个项目)
- [许可证](#许可证)

---

## 包含内容

| 类别 | 数量 | 描述 |
|------|------|------|
| **Agent** | 49 | 跨设计、程序、美术、音效、叙事、QA 和制作的专业化子 Agent |
| **技能** | 72 | 覆盖各工作流程阶段的斜杠命令（`/start`、`/design-system`、`/create-epics`、`/create-stories`、`/dev-story`、`/story-done` 等）|
| **Hooks** | 12 | 在提交、推送、资源变更、会话生命周期、Agent 审计跟踪和差距检测上的自动化验证 |
| **规则** | 11 | 编辑游戏玩法、引擎、AI、UI、网络代码等时强制执行的路径作用域编码标准 |
| **模板** | 39 | 适用于 GDD、UX 规格、ADR、冲刺计划、HUD 设计、无障碍等的文档模板 |

## 工作室层级

Agent 按三个层级组织，参考真实工作室的运作方式：

```
第一层 — 总监（Opus）
  creative-director    technical-director    producer

第二层 — 部门主管（Sonnet）
  game-designer        lead-programmer       art-director
  audio-director       narrative-director     qa-lead
  release-manager      localization-lead

第三层 — 专家（Sonnet/Haiku）
  gameplay-programmer  engine-programmer     ai-programmer
  network-programmer   tools-programmer       ui-programmer
  systems-designer     level-designer         economy-designer
  technical-artist     sound-designer         writer
  world-builder        ux-designer            prototyper
  performance-analyst  devops-engineer        analytics-engineer
  security-engineer    qa-tester              accessibility-specialist
  live-ops-designer    community-manager
```

### 引擎专家

模板包含三个主流引擎的 Agent 套件。使用与你的项目匹配的套件：

| 引擎 | 主导 Agent | 子专家 |
|------|-----------|--------|
| **Godot 4** | `godot-specialist` | GDScript、Shader、GDExtension |
| **Unity** | `unity-specialist` | DOTS/ECS、Shader/VFX、Addressables、UI Toolkit |
| **Unreal Engine 5** | `unreal-specialist` | GAS、Blueprints、Replication、UMG/CommonUI |

## 斜杠命令

在 Claude Code 中输入 `/` 访问所有 72 个技能：

**入职与导航**
`/start` `/help` `/project-stage-detect` `/setup-engine` `/adopt`

**游戏设计**
`/brainstorm` `/map-systems` `/design-system` `/quick-design` `/review-all-gdds` `/propagate-design-change`

**美术与资源**
`/art-bible` `/asset-spec` `/asset-audit`

**UX 与界面设计**
`/ux-design` `/ux-review`

**架构**
`/create-architecture` `/architecture-decision` `/architecture-review` `/create-control-manifest`

**故事与冲刺**
`/create-epics` `/create-stories` `/dev-story` `/sprint-plan` `/sprint-status` `/story-readiness` `/story-done` `/estimate`

**评审与分析**
`/design-review` `/code-review` `/balance-check` `/content-audit` `/scope-check` `/perf-profile` `/tech-debt` `/gate-check` `/consistency-check`

**QA 与测试**
`/qa-plan` `/smoke-check` `/soak-test` `/regression-suite` `/test-setup` `/test-helpers` `/test-evidence-review` `/test-flakiness` `/skill-test` `/skill-improve`

**制作**
`/milestone-review` `/retrospective` `/bug-report` `/bug-triage` `/reverse-document` `/playtest-report`

**发布**
`/release-checklist` `/launch-checklist` `/changelog` `/patch-notes` `/hotfix`

**创意与内容**
`/prototype` `/onboard` `/localize`

**团队编排**（协调多个 Agent 共同完成单一功能）
`/team-combat` `/team-narrative` `/team-ui` `/team-release` `/team-polish` `/team-audio` `/team-level` `/team-live-ops` `/team-qa`

## 快速开始

### 前置要求

- [Git](https://git-scm.com/)
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)（`npm install -g @anthropic-ai/claude-code`）
- **推荐**：[jq](https://jqlang.github.io/jq/)（用于 hook 验证）和 Python 3（用于 JSON 验证）

所有 hook 在可选工具缺失时都会优雅失败——不会出问题，只是失去验证功能。

### 设置

1. **克隆或用作模板**：
   ```bash
   git clone https://github.com/Donchitos/Claude-Code-Game-Studios.git my-game
   cd my-game
   ```

2. **打开 Claude Code** 并启动会话：
   ```bash
   claude
   ```

3. **运行 `/start`**——系统会询问你目前处于什么阶段（完全没想法、模糊概念、清晰设计、已有工作），然后引导你进入正确的工作流程。不做任何假设。

   或者如果你已经知道需要什么，可以直接跳转到特定技能：
   - `/brainstorm`——从零开始探索游戏创意
   - `/setup-engine godot 4.6`——如果你已经确定引擎，进行配置
   - `/project-stage-detect`——分析已有项目

## 升级指南

已经在使用旧版本这个模板了？查看 [UPGRADING.md](UPGRADING.md) 获取分步骤迁移说明、版本间变更详情，以及哪些文件可以安全覆盖、哪些需要手动合并。

## 项目结构

```
CLAUDE.md                           # 主配置
.claude/
  settings.json                     # Hooks、权限、安全规则
  agents/                           # 49 个 Agent 定义（markdown + YAML 前置元数据）
  skills/                           # 72 个斜杠命令（每个技能一个子目录）
  hooks/                            # 12 个 hook 脚本（bash，跨平台）
  rules/                            # 11 个路径作用域编码标准
  statusline.sh                     # 状态行脚本（context%、model、阶段、epic 面包屑）
  docs/
    workflow-catalog.yaml           # 7 阶段流水线定义（由 /help 读取）
    templates/                      # 39 个文档模板
src/                                # 游戏源代码
assets/                             # 美术、音效、VFX、shader、数据文件
design/                             # GDD、叙事文档、关卡设计
docs/                               # 技术文档和 ADR
tests/                              # 测试套件（单元、集成、性能、试玩）
tools/                              # 构建和流水线工具
prototypes/                         # 临时原型（与 src/ 隔离）
production/                         # 冲刺计划、里程碑、发布追踪
```

## 工作原理

### Agent 协调

Agent 遵循结构化委托模型：

1. **纵向委托**——总监委托给主管，主管委托给专家
2. **横向协商**——同层级的 Agent 可以互相协商，但不能做跨领域的约束性决策
3. **冲突解决**——分歧升级到共同上级（设计领域升级到 `creative-director`，技术领域升级到 `technical-director`）
4. **变更传播**——跨部门变更由 `producer` 协调
5. **领域边界**——Agent 未经明确委托不能修改其领域外的文件

### 协作而非自主

这是**不是**一个自动驾驶系统。每个 Agent 都遵循严格的协作协议：

1. **提问**——Agent 在提出解决方案之前先提问
2. **展示选项**——Agent 展示 2-4 个选项及优缺点
3. **你来决定**——用户始终做决定
4. **草稿**——Agent 在定稿前展示工作成果
5. **批准**——没有你的签字什么都不会被写入

你保持控制权。Agent 提供的是结构和专业知识，而非自主权。

### 自动化安全

**Hooks** 在每个会话中自动运行：

| Hook | 触发时机 | 功能 |
|------|---------|------|
| `validate-commit.sh` | PreToolUse (Bash) | 检查硬编码值、TODO 格式、JSON 有效性、设计文档章节——如果命令不是 `git commit` 则提前退出 |
| `validate-push.sh` | PreToolUse (Bash) | 警告推送到受保护分支——如果命令不是 `git push` 则提前退出 |
| `validate-assets.sh` | PostToolUse (Write/Edit) | 验证命名约定和 JSON 结构——如果文件不在 `assets/` 中则提前退出 |
| `session-start.sh` | 会话打开 | 显示当前分支和最近提交以供定向 |
| `detect-gaps.sh` | 会话打开 | 检测新项目（建议 `/start`）以及代码或原型存在时缺失设计文档的情况 |
| `pre-compact.sh` | 压缩前 | 保存会话进度笔记 |
| `post-compact.sh` | 压缩后 | 提醒 Claude 从 `active.md` 恢复会话状态 |
| `notify.sh` | 通知事件 | 通过 PowerShell 显示 Windows toast 通知 |
| `session-stop.sh` | 会话关闭 | 将 `active.md` 归档到会话日志并记录 git 活动 |
| `log-agent.sh` | Agent 产生 | 审计跟踪开始——记录子 Agent 调用 |
| `log-agent-stop.sh` | Agent 停止 | 审计跟踪停止——完成子 Agent 记录 |
| `validate-skill-change.sh` | PostToolUse (Write/Edit) | 在任何 `.claude/skills/` 变更后建议运行 `/skill-test` |

> **注意**：`validate-commit.sh`、`validate-assets.sh` 和 `validate-skill-change.sh` 在每次 Bash/Write 工具调用时触发，但当命令或文件路径不相关时会立即退出（exit 0）。这是正常的 hook 行为——不是性能问题。

**权限规则**在 `settings.json` 中自动允许安全操作（git status、测试运行）并阻止危险操作（强制推送、`rm -rf`、读取 `.env` 文件）。

### 路径作用域规则

编码标准根据文件位置自动强制执行：

| 路径 | 强制执行 |
|------|---------|
| `src/gameplay/**` | 数据驱动值、delta time 使用、无 UI 引用 |
| `src/core/**` | 热路径零分配、线程安全、API 稳定性 |
| `src/ai/**` | 性能预算、可调试性、数据驱动参数 |
| `src/networking/**` | 服务器权威、版本化消息、安全 |
| `src/ui/**` | 无游戏状态所有权、本地化就绪、无障碍 |
| `design/gdd/**` | 必需的 8 个章节、公式格式、边缘情况 |
| `tests/**` | 测试命名、覆盖率要求、fixture 模式 |
| `prototypes/**` | 宽松标准、需要 README、记录假设 |

## 设计理念

这个模板基于专业游戏开发实践：

- **MDA 框架**——游戏设计的 Mechanics、Dynamics、Aesthetics 分析
- **自我决定理论**——玩家动机的自主性、能力感、归属感
- **心流状态设计**——挑战-技能平衡以维持玩家参与度
- **Bartle 玩家类型**——受众定位和验证
- **验证驱动开发**——测试优先，然后实现

## 自定义

这是一个**模板**，不是锁定框架。所有内容都可以自定义：

- **添加/移除 Agent**——删除你不需要的 Agent 文件，为你的领域添加新的
- **编辑 Agent 提示**——调整 Agent 行为，添加项目特定知识
- **修改技能**——调整工作流程以匹配你团队的过程
- **添加规则**——为你的项目目录结构创建新的路径作用域规则
- **调整 hooks**——调整验证严格度，添加新的检查
- **选择你的引擎**——使用 Godot、Unity 或 Unreal Agent 套件（或都不用）
- **设置评审强度**——`full`（所有总监关卡）、`lean`（仅阶段关卡）或 `solo`（无）。在 `/start` 期间设置或编辑 `production/review-mode.txt`。在任意技能运行时用 `--review solo` 覆盖。

## 平台支持

已在 **Windows 10** + Git Bash 上测试。所有 hook 使用 POSIX 兼容模式（`grep -E`，不是 `grep -P`）并在工具缺失时包含备用方案。无需修改即可在 macOS 和 Linux 上运行。

## 社区

- **讨论**——[GitHub Discussions](https://github.com/Donchitos/Claude-Code-Game-Studios/discussions) 用于提问、想法和展示你做的东西
- **Issues**——[Bug 报告和功能请求](https://github.com/Donchitos/Claude-Code-Game-Studios/issues)

---

## 支持这个项目

Claude Code Game Studios 是免费开源的。如果它节省了你的时间或帮助你发布了游戏，考虑支持持续开发：

<p>
  <a href="https://www.buymeacoffee.com/donchitos3"><img src="https://img.shields.io/badge/Buy%20Me%20a%20Coffee-FFDD00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black" alt="Buy Me a Coffee"></a>
  &nbsp;
  <a href="https://github.com/sponsors/Donchitos"><img src="https://img.shields.io/badge/GitHub%20Sponsors-ea4aaa?style=for-the-badge&logo=githubsponsors&logoColor=white" alt="GitHub Sponsors"></a>
</p>

- **[Buy Me a Coffee](https://www.buymeacoffee.com/donchitos3)**——一次性支持
- **[GitHub Sponsors](https://github.com/sponsors/Donchitos)**——通过 GitHub 持续支持

赞助帮助资助维护技能、添加新 Agent、跟进 Claude Code 和引擎 API 变更，以及回应社区问题。

---

*为 Claude Code 而建。通过 [GitHub Discussions](https://github.com/Donchitos/Claude-Code-Game-Studios/discussions) 欢迎贡献和扩展。*

## 许可证

MIT 许可证。详见 [LICENSE](LICENSE)。
