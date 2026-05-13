<p align="center">
  <h1 align="center">Claude Code Game Studios</h1>
  <p align="center">
    将单个 Claude Code 会话变成一个完整的游戏开发工作室。
    <br />
    49 个代理。72 项技能。一个协同的 AI 团队。
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

## 为什么存在

用 AI 独自开发游戏虽然强大——但单一的聊天会话缺乏结构。没有人阻止你对魔法数字进行硬编码、跳过设计文档或编写意大利面条式代码。没有质量保证检查，没有设计评审，也没有人问"这真的符合游戏的愿景吗？"

**Claude Code Game Studios** 通过为你的 AI 会话赋予真实工作室的结构来解决这个问题。你不是只有一个通用助手，而是拥有 49 个专业代理，按照工作室层级组织——有守护愿景的总监、掌管各自领域的部门负责人、以及负责具体执行的专业人员。每个代理都有明确的职责、升级路径和质量关卡。

结果是：你仍然做出所有决策，但你现在拥有一个会提出正确问题、及早发现错误、并从最初的头脑风暴到发布全程保持项目有序的团队。

---

## 目录

- [包含内容](#whats-included)
- [工作室层级](#studio-hierarchy)
- [斜杠命令](#slash-commands)
- [快速开始](#getting-started)
- [升级指南](#upgrading)
- [项目结构](#project-structure)
- [工作原理](#how-it-works)
- [设计理念](#design-philosophy)
- [自定义](#customization)
- [平台支持](#platform-support)
- [社区](#community)
- [支持本项目](#supporting-this-project)
- [许可证](#license)

---

## 包含内容

| 类别 | 数量 | 描述 |
|----------|-------|-------------|
| **代理** | 49 | 涵盖设计、编程、美术、音频、叙事、质量保证和制作的专业子代理 |
| **技能** | 72 | 每个工作流阶段的斜杠命令（`/start`、`/design-system`、`/create-epics`、`/create-stories`、`/dev-story`、`/story-done` 等） |
| **钩子** | 12 | 提交、推送、资源变更、会话生命周期、代理审计追踪和差距检测的自动验证 |
| **规则** | 11 | 在编辑游戏逻辑、引擎、AI、UI、网络代码等时强制执行的路径范围编码标准 |
| **模板** | 39 | 用于 GDD、UX 规范、ADR、冲刺计划、HUD 设计、可访问性等的文档模板 |

## 工作室层级

代理按三个层级组织，与真实工作室的运作方式相匹配：

```
第 1 层 — 总监（Opus）
  creative-director    technical-director    producer

第 2 层 — 部门负责人（Sonnet）
  game-designer        lead-programmer       art-director
  audio-director       narrative-director    qa-lead
  release-manager      localization-lead

第 3 层 — 专业人员（Sonnet/Haiku）
  gameplay-programmer  engine-programmer     ai-programmer
  network-programmer   tools-programmer      ui-programmer
  systems-designer     level-designer        economy-designer
  technical-artist     sound-designer        writer
  world-builder        ux-designer           prototyper
  performance-analyst  devops-engineer       analytics-engineer
  security-engineer    qa-tester             accessibility-specialist
  live-ops-designer    community-manager
```

### 引擎专家

模板包含所有三大引擎的代理集。使用与你项目匹配的那一套：

| 引擎 | 主管代理 | 子专家 |
|--------|-----------|-----------------|
| **Godot 4** | `godot-specialist` | GDScript、着色器、GDExtension |
| **Unity** | `unity-specialist` | DOTS/ECS、着色器/VFX、Addressables、UI Toolkit |
| **Unreal Engine 5** | `unreal-specialist` | GAS、蓝图、复制、UMG/CommonUI |

## 斜杠命令

在 Claude Code 中键入 `/` 以访问全部 72 项技能：

**入门与导航**
`/start` `/help` `/project-stage-detect` `/setup-engine` `/adopt`

**游戏设计**
`/brainstorm` `/map-systems` `/design-system` `/quick-design` `/review-all-gdds` `/propagate-design-change`

**美术与资源**
`/art-bible` `/asset-spec` `/asset-audit`

**用户体验与界面设计**
`/ux-design` `/ux-review`

**架构**
`/create-architecture` `/architecture-decision` `/architecture-review` `/create-control-manifest`

**故事与冲刺**
`/create-epics` `/create-stories` `/dev-story` `/sprint-plan` `/sprint-status` `/story-readiness` `/story-done` `/estimate`

**评审与分析**
`/design-review` `/code-review` `/balance-check` `/content-audit` `/scope-check` `/perf-profile` `/tech-debt` `/gate-check` `/consistency-check`

**质量保证与测试**
`/qa-plan` `/smoke-check` `/soak-test` `/regression-suite` `/test-setup` `/test-helpers` `/test-evidence-review` `/test-flakiness` `/skill-test` `/skill-improve`

**制作**
`/milestone-review` `/retrospective` `/bug-report` `/bug-triage` `/reverse-document` `/playtest-report`

**发布**
`/release-checklist` `/launch-checklist` `/changelog` `/patch-notes` `/hotfix`

**创意与内容**
`/prototype` `/onboard` `/localize`

**团队编排**（协调多个代理处理单个功能）
`/team-combat` `/team-narrative` `/team-ui` `/team-release` `/team-polish` `/team-audio` `/team-level` `/team-live-ops` `/team-qa`

## 快速开始

### 前提条件

- [Git](https://git-scm.com/)
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)（`npm install -g @anthropic-ai/claude-code`）
- **推荐**：[jq](https://jqlang.github.io/jq/)（用于钩子验证）和 Python 3（用于 JSON 验证）

所有钩子在缺少可选工具时都会优雅降级——不会破坏任何功能，只是失去验证能力。

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

3. **运行 `/start`** — 系统会询问你当前的状态（没有想法、模糊概念、明确设计、已有项目），并引导你进入正确的工作流程。不做任何假设。

   如果你已经知道自己需要什么，也可以直接跳转到特定技能：
   - `/brainstorm` — 从零开始探索游戏创意
   - `/setup-engine godot 4.6` — 如果你已经知道，直接配置引擎
   - `/project-stage-detect` — 分析现有项目

## 升级指南

已经在使用此模板的旧版本？请参阅 [UPGRADING.md](UPGRADING.md) 了解分步迁移说明、版本间变更明细，以及哪些文件可以安全覆盖、哪些需要手动合并。

## 项目结构

```
CLAUDE.md                           # 主配置文件
.claude/
  settings.json                     # 钩子、权限、安全规则
  agents/                           # 49 个代理定义（Markdown + YAML 前置元数据）
  skills/                           # 72 个斜杠命令（每个技能一个子目录）
  hooks/                            # 12 个钩子脚本（bash，跨平台）
  rules/                            # 11 个路径范围的编码标准
  statusline.sh                     # 状态栏脚本（上下文百分比、模型、阶段、史诗面包屑）
  docs/
    workflow-catalog.yaml           # 7 阶段管线定义（由 /help 读取）
    templates/                      # 39 个文档模板
src/                                # 游戏源代码
assets/                             # 美术、音频、VFX、着色器、数据文件
design/                             # GDD、叙事文档、关卡设计
docs/                               # 技术文档和 ADR
tests/                              # 测试套件（单元测试、集成测试、性能测试、玩法测试）
tools/                              # 构建和管线工具
prototypes/                         # 一次性原型（与 src/ 隔离）
production/                         # 冲刺计划、里程碑、发布跟踪
```

## 工作原理

### 代理协调

代理遵循结构化的委派模型：

1. **垂直委派** — 总监委派给负责人，负责人委派给专业人员
2. **水平咨询** — 同层代理可以相互咨询，但不能做出具有约束力的跨领域决策
3. **冲突解决** — 分歧上报给共同的上级（设计问题上报 `creative-director`，技术问题上报 `technical-director`）
4. **变更传播** — 跨部门变更由 `producer` 协调
5. **领域边界** — 未经明确委派，代理不得修改其领域之外的文件

### 协作而非自主

这**不是**自动驾驶系统。每个代理都遵循严格的协作协议：

1. **提问** — 代理在提出解决方案之前先提问
2. **提供方案** — 代理展示 2-4 个方案及其优缺点
3. **你来做决定** — 用户始终做出最终决定
4. **草稿** — 代理在最终确定之前展示工作成果
5. **批准** — 未经你的同意，任何内容都不会被写入

你始终掌控全局。代理提供结构和专业知识，而非自主权。

### 自动化安全

**钩子**在每次会话中自动运行：

| 钩子 | 触发条件 | 功能说明 |
|------|---------|--------------|
| `validate-commit.sh` | PreToolUse (Bash) | 检查硬编码值、TODO 格式、JSON 有效性、设计文档章节——如果命令不是 `git commit` 则提前退出 |
| `validate-push.sh` | PreToolUse (Bash) | 推送到保护分支时发出警告——如果命令不是 `git push` 则提前退出 |
| `validate-assets.sh` | PostToolUse (Write/Edit) | 验证命名规范和 JSON 结构——如果文件不在 `assets/` 中则提前退出 |
| `session-start.sh` | 会话打开 | 显示当前分支和最近提交以提供方向感 |
| `detect-gaps.sh` | 会话打开 | 检测新项目（建议运行 `/start`），以及在存在代码或原型时检测缺失的设计文档 |
| `pre-compact.sh` | 压缩前 | 保留会话进度记录 |
| `post-compact.sh` | 压缩后 | 提醒 Claude 从 `active.md` 恢复会话状态 |
| `notify.sh` | 通知事件 | 通过 PowerShell 显示 Windows 弹窗通知 |
| `session-stop.sh` | 会话关闭 | 将 `active.md` 归档到会话日志并记录 git 活动 |
| `log-agent.sh` | 代理生成 | 审计追踪开始——记录子代理调用 |
| `log-agent-stop.sh` | 代理停止 | 审计追踪结束——完成子代理记录 |
| `validate-skill-change.sh` | PostToolUse (Write/Edit) | 建议在对 `.claude/skills/` 进行任何更改后运行 `/skill-test` |

> **注意**：`validate-commit.sh`、`validate-assets.sh` 和 `validate-skill-change.sh` 在每次 Bash/Write 工具调用时都会触发，并在命令或文件路径不相关时立即退出（exit 0）。这是正常的钩子行为——不是性能问题。

**权限规则**在 `settings.json` 中自动允许安全操作（git status、测试运行）并阻止危险操作（强制推送、`rm -rf`、读取 `.env` 文件）。

### 路径范围规则

编码标准根据文件位置自动执行：

| 路径 | 强制规则 |
|------|----------|
| `src/gameplay/**` | 数据驱动值、delta time 使用、无 UI 引用 |
| `src/core/**` | 热路径零分配、线程安全、API 稳定性 |
| `src/ai/**` | 性能预算、可调试性、数据驱动参数 |
| `src/networking/**` | 服务器权威、版本化消息、安全 |
| `src/ui/**` | 无游戏状态所有权、本地化就绪、可访问性 |
| `design/gdd/**` | 必需 8 个章节、公式格式、边界情况 |
| `tests/**` | 测试命名、覆盖要求、夹具模式 |
| `prototypes/**` | 宽松标准、需要 README、记录假设 |

## 设计理念

此模板基于专业游戏开发实践：

- **MDA 框架** — 机制、动态、美学分析用于游戏设计
- **自我决定理论** — 自主性、胜任感、归属感用于玩家动机
- **心流状态设计** — 挑战与技能的平衡用于玩家参与
- **Bartle 玩家类型** — 受众定位和验证
- **验证驱动开发** — 先测试，后实现

## 自定义

这是一个**模板**，而非锁定框架。所有内容都旨在被自定义：

- **添加/移除代理** — 删除不需要的代理文件，为你的领域添加新的代理
- **编辑代理提示词** — 调整代理行为，添加项目特定知识
- **修改技能** — 调整工作流程以匹配你的团队流程
- **添加规则** — 为你的项目目录结构创建新的路径范围规则
- **调整钩子** — 调整验证严格程度，添加新的检查
- **选择你的引擎** — 使用 Godot、Unity 或 Unreal 代理集（或不使用）
- **设置评审强度** — `full`（所有总监关卡）、`lean`（仅阶段关卡）或 `solo`（无）。在 `/start` 期间设置或编辑 `production/review-mode.txt`。可在任何技能上使用 `--review solo` 按次覆盖。

## 平台支持

在 **Windows 10** 上使用 Git Bash 测试通过。所有钩子使用 POSIX 兼容模式（`grep -E`，而非 `grep -P`），并包含缺失工具的备用方案。在 macOS 和 Linux 上无需修改即可运行。

## 社区

- **讨论** — [GitHub Discussions](https://github.com/Donchitos/Claude-Code-Game-Studios/discussions) 用于提问、创意分享和展示你的作品
- **问题反馈** — [Bug 报告和功能请求](https://github.com/Donchitos/Claude-Code-Game-Studios/issues)

---

## 支持本项目

Claude Code Game Studios 是免费开源项目。如果它为你节省了时间或帮助你发布了游戏，请考虑支持持续开发：

<p>
  <a href="https://www.buymeacoffee.com/donchitos3"><img src="https://img.shields.io/badge/Buy%20Me%20a%20Coffee-FFDD00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black" alt="Buy Me a Coffee"></a>
  &nbsp;
  <a href="https://github.com/sponsors/Donchitos"><img src="https://img.shields.io/badge/GitHub%20Sponsors-ea4aaa?style=for-the-badge&logo=githubsponsors&logoColor=white" alt="GitHub Sponsors"></a>
</p>

- **[Buy Me a Coffee](https://www.buymeacoffee.com/donchitos3)** — 一次性支持
- **[GitHub Sponsors](https://github.com/sponsors/Donchitos)** — 通过 GitHub 的定期支持

赞助有助于资助维护技能、添加新代理、跟进 Claude Code 和引擎 API 变更，以及回应社区问题所需的时间投入。

---

*为 Claude Code 构建。维护和扩展中——欢迎通过 [GitHub Discussions](https://github.com/Donchitos/Claude-Code-Game-Studios/discussions) 贡献。*

## 许可证

MIT 许可证。详见 [LICENSE](LICENSE)。
