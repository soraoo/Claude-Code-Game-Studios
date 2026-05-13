# 升级 Claude Code Game Studios

本指南涵盖了将现有游戏项目仓库从一个模板版本升级到下一个版本的操作步骤。

**在 git 日志中查找当前版本**：
```bash
git log --oneline | grep -i "release\|setup"
```
或者检查 `README.md` 中的版本徽章。

---

## 目录

- [升级策略](#upgrade-strategies)
- [v0.4.x → v1.0](#v04x--v10)
- [v0.4.0 → v0.4.1](#v040--v041)
- [v0.3.0 → v0.4.0](#v030--v040)
- [v0.2.0 → v0.3.0](#v020--v030)
- [v0.1.0 → v0.2.0](#v010--v020)

---

## 升级策略

有三种方式可以拉取模板更新。根据你的仓库设置方式选择。

### 策略 A — Git 远程合并（推荐）

最适用于：你克隆了模板并在其上进行了自己的提交。

```bash
# 将模板添加为远程仓库（一次性设置）
git remote add template https://github.com/Donchitos/Claude-Code-Game-Studios.git

# 获取新版本
git fetch template main

# 合并到你的分支
git merge template/main --allow-unrelated-histories
```

Git 只会在你*和*模板都更改过的文件中标记冲突。逐个解决——保留你的游戏内容，同时纳入结构性改进。然后提交合并。

**提示：** 最可能冲突的文件是 `CLAUDE.md` 和 `.claude/docs/technical-preferences.md`，因为你已经用你的引擎和项目设置填写了它们。保留你的内容；接受结构性更改。

---

### 策略 B — 精选特定提交

最适用于：你只想要某个特定功能（例如，只想要新技能，而不是完整更新）。

```bash
git remote add template https://github.com/Donchitos/Claude-Code-Game-Studios.git
git fetch template main

# 精选你想要的特定提交
git cherry-pick <commit-sha>
```

每个版本的提交 SHA 列在下面的版本章节中。

---

### 策略 C — 手动文件复制

最适用于：你没有使用 git 来设置模板（只是下载了 zip 包）。

1. 将新版本下载或克隆到你的仓库旁边。
2. 将**"安全覆盖"**下列出的文件直接复制。
3. 对于**"仔细合并"**下的文件，同时打开两个版本，手动合并结构性更改，同时保留你的内容。

---

## v0.4.1

**发布：** 2026-04-02
**关键主题：** 美术方向集成，资源规范管线

### 变更内容

| 类别 | 变更 |
|----------|---------|
| **新技能** | `/art-bible` — 引导式逐章节视觉身份创作（9 个章节）。每章节必须生成 art-director 任务。AD-ART-BIBLE 签收关卡。在技术设置阶段必需。 |
| **新技能** | `/asset-spec` — 每项资源的视觉规范和 AI 生成提示词生成器。读取艺术圣经 + GDD/关卡/角色文档。写入 `design/assets/specs/` 文件和 `design/assets/asset-manifest.md`。支持 full/lean/solo 模式。 |
| **新总监关卡（3 个）** | `AD-CONCEPT-VISUAL`（头脑风暴阶段 4）、`AD-ART-BIBLE`（艺术圣经签收）、`AD-PHASE-GATE`（关卡检查面板） |
| **`/brainstorm` 更新** | 将 `Task` 添加到允许工具列表（之前缺失——阻止了所有总监生成）。美术总监现在与创意总监在核心支柱锁定后并行生成。视觉身份锚点写入 game-concept.md。 |
| **`/gate-check` 更新** | 美术总监作为第 4 个并行总监加入（AD-PHASE-GATE）。视觉产物检查：视觉身份锚点（概念关卡）、艺术圣经（技术设置关卡）、AD-ART-BIBLE 签收 + 角色视觉档案（预制作关卡）。 |
| **`/team-level` 更新** | 在步骤 1 并行生成中添加美术总监（视觉方向先于布局）。关卡设计师现在接收美术总监目标作为显式约束。步骤 4 美术总监角色修正为仅生产概念。 |
| **`/team-narrative` 更新** | 在阶段 2 并行生成中添加美术总监（角色视觉设计、环境叙事、电影基调）。 |
| **`/design-system` 更新** | 路由表扩展，为战斗、UI、对话、动画/VFX、角色类别添加美术总监 + 技术美术师。视觉/音频部分现在为 7 个系统类别强制要求（附带美术总监任务生成）。 |
| **`workflow-catalog.yaml`** | `/art-bible` 添加到技术设置（必需）。`/asset-spec` 添加到预制作（可选，可重复）。 |

### 文件：安全覆盖

**新增文件：**
```
.claude/skills/art-bible/SKILL.md
.claude/skills/asset-spec/SKILL.md
.claude/docs/director-gates.md
```

**要覆盖的现有文件（无用户内容）：**
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

### 文件：仔细合并

无——所有更改都是基础设施文件，没有用户内容。

---

## v0.4.x → v1.0

**发布：** 2026-03-29
**提交范围：** `6c041ac..HEAD`
**关键主题：** 总监关卡系统、关卡强度模式、Godot C# 专家

### 变更内容

| 类别 | 变更 |
|----------|---------|
| **新系统** | 总监关卡——所有工作流技能共享的命名审查检查点。定义在 `.claude/docs/director-gates.md` |
| **新功能** | 关卡强度模式：`full`（所有总监关卡）、`lean`（仅阶段关卡）、`solo`（无总监）。在 `/start` 期间通过 `production/review-mode.txt` 全局设置，或在任何使用关卡的技能上通过 `--review [mode]` 按次覆盖 |
| **新代理** | `godot-csharp-specialist` — Godot 4 项目中的 C# 代码质量 |
| **技能更新（13 个）** | 所有使用关卡的技能现在解析 `--review [full\|lean\|solo]` 并将其包含在参数提示中：`brainstorm`、`map-systems`、`design-system`、`architecture-decision`、`create-architecture`、`create-epics`、`create-stories`、`sprint-plan`、`milestone-review`、`playtest-report`、`prototype`、`story-done`、`gate-check` |
| **`/start` 更新** | 新增阶段 3b——在入门引导期间设置审查模式，写入 `production/review-mode.txt` |
| **`/setup-engine` 更新** | Godot 的语言选择步骤（GDScript vs C#） |
| **文档** | `director-gates.md` — 完整关卡目录；`WORKFLOW-GUIDE.md` — 总监审查模式章节；`README.md` — 审查强度自定义 |

---

### 文件：安全覆盖

**新增文件：**
```
.claude/agents/godot-csharp-specialist.md
.claude/docs/director-gates.md
```

**要覆盖的现有文件（无用户内容）：**
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

### 文件：仔细合并

此版本中无需手动合并的文件。所有更改都是基础设施文件，没有用户内容。

---

### 新功能

#### 总监关卡系统

所有主要工作流技能现在引用定义在 `.claude/docs/director-gates.md` 中的命名关卡检查点。关卡通过领域前缀和名称标识（例如 `CD-CONCEPT`、`TD-ARCHITECTURE`、`LP-CODE-REVIEW`）。每个关卡定义了要生成哪个总监、传递什么输入、判定结果的含义，以及 lean/solo 模式如何影响它。

技能使用带有关卡 ID 和文档化输入的 `Task` 来生成关卡，而不是内联嵌入总监提示。这保持了技能主体的整洁，并使关卡行为在所有工作流阶段保持一致。

#### 关卡强度模式

三种模式让你控制获得多少总监审查：

- **`full`**（默认）——所有总监关卡在每个审查检查点都运行
- **`lean`**——按技能进行的总监审查被跳过；`/gate-check` 处的阶段关卡仍会运行
- **`solo`**——任何地方都没有总监关卡；`/gate-check` 仅检查产物的存在

在 `/start` 期间全局设置（写入 `production/review-mode.txt`）。在任何使用关卡的技能上使用 `--review [mode]` 覆盖单次运行：

```
/design-system combat --review lean
/gate-check concept --review full
/brainstorm my-game-idea --review solo
```

---

### 升级后

1. 运行 `/start` 一次以设置你的首选审查模式——或手动创建 `production/review-mode.txt`，内容为 `full`、`lean` 或 `solo`。
2. 如果你正在进行中的项目，请查看 `.claude/docs/director-gates.md` 以了解哪些关卡适用于你当前阶段。
3. 运行 `/skill-test static all` 以验证所有技能通过结构检查。

---

## v0.4.0 → v0.4.1

**发布：** 2026-03-26
**提交范围：** `04ed5d5..HEAD`
**关键主题：** 类型无关代理、新技能、技能修复

### 变更内容

| 类别 | 变更 |
|----------|---------|
| **新技能（1 个）** | `/consistency-check` — 跨 GDD 实体一致性扫描器 |
| **技能修复（所有 team-*）** | 添加无参数守卫、正式 `Verdict: COMPLETE / BLOCKED` 关键词、每步 AskUserQuestion 关卡、相邻区域依赖检查（team-level）、道德执行（team-live-ops）、带有阶段跳过的 NO-GO 路径（team-release） |
| **代理修复（4 个）** | game-designer、systems-designer、economy-designer、live-ops-designer 中的类型无关语言——移除了 RPG 特定术语 |

---

### 文件：安全覆盖

**新增文件：**
```
.claude/skills/consistency-check/SKILL.md
```

**要覆盖的现有文件（无用户内容）：**
```
.claude/skills/team-combat/SKILL.md      ← 无参数守卫、判决关键词、关卡改进
.claude/skills/team-narrative/SKILL.md   ← 无参数守卫、判决关键词、关卡改进
.claude/skills/team-ui/SKILL.md          ← 无参数守卫、判决关键词、关卡改进
.claude/skills/team-release/SKILL.md     ← 无参数守卫、判决关键词、NO-GO 路径
.claude/skills/team-polish/SKILL.md      ← 无参数守卫、判决关键词、关卡改进
.claude/skills/team-audio/SKILL.md       ← 无参数守卫、判决关键词、关卡改进
.claude/skills/team-level/SKILL.md       ← 无参数守卫、判决关键词、相邻区域检查
.claude/skills/team-live-ops/SKILL.md    ← 无参数守卫、判决关键词、道德执行
.claude/skills/team-qa/SKILL.md          ← 无参数守卫、判决关键词、关卡改进
.claude/skills/map-systems/SKILL.md      ← 判决关键词
.claude/skills/create-epics/SKILL.md     ← "May I write" 协议修复、判决关键词
.claude/skills/create-stories/SKILL.md   ← 判决关键词
.claude/agents/game-designer.md          ← 类型无关语言
.claude/agents/systems-designer.md       ← 类型无关语言
.claude/agents/economy-designer.md       ← 类型无关语言
.claude/agents/live-ops-designer.md      ← 类型无关语言
```

---

### 文件：仔细合并

此版本中无需手动合并的文件。所有更改都是基础设施文件，没有用户内容。

---

### 升级后

1. 运行 `/skill-test catalog` 以验证所有技能都已索引。
2. 在对技能进行任何编辑后运行 `/skill-test lint [skill-name]` 以检查结构合规性。
3. 如果你自定义过任何 team-* 技能，请查看更新后的版本——所有 team-* 技能现在都需要无参数守卫和 `Verdict:` 关键词。

---

## v0.3.0 → v0.4.0

**发布：** 2026-03-21
**提交范围：** `b1cad29..HEAD`
**关键主题：** 完整 UX/UI 管线、完整故事生命周期、棕地采用、全面质量保证/测试框架、管线完整性、29 项新技能

### 变更内容

| 类别 | 变更 |
|----------|---------|
| **新技能（17 个）** | `/ux-design`、`/ux-review`、`/help`、`/quick-design`、`/review-all-gdds`、`/story-readiness`、`/story-done`、`/sprint-status`、`/adopt`、`/create-architecture`、`/create-control-manifest`、`/create-epics`、`/create-stories`、`/dev-story`、`/propagate-design-change`、`/content-audit`、`/architecture-review` |
| **新技能 - 质量保证（12 个）** | `/qa-plan`、`/smoke-check`、`/soak-test`、`/regression-suite`、`/test-setup`、`/test-helpers`、`/test-evidence-review`、`/test-flakiness`、`/skill-test`、`/bug-triage`、`/team-live-ops`、`/team-qa` |
| **新钩子（4 个）** | `log-agent-stop.sh` — 代理审计追踪停止；`notify.sh` — Windows 弹窗通知；`post-compact.sh` — 压缩后会话恢复提醒；`validate-skill-change.sh` — 在技能编辑后建议运行 `/skill-test` |
| **新模板（8 个）** | `ux-spec.md`、`hud-design.md`、`accessibility-requirements.md`、`interaction-pattern-library.md`、`player-journey.md`、`difficulty-curve.md` 和 2 个采用计划模板 |
| **新基础设施** | `workflow-catalog.yaml`（7 阶段管线，由 `/help` 读取）、`docs/architecture/tr-registry.yaml`（稳定 TR-ID）、`production/sprint-status.yaml` 模式 |
| **技能更新** | `/gate-check` — 3 个关卡现在需要 UX 产物；预制作关卡需要垂直切片（硬关卡） |
| **技能更新** | `/sprint-plan` — 写入 `sprint-status.yaml`；`/sprint-status` 读取它 |
| **技能更新** | `/story-done` — 8 阶段完成审查，更新故事文件，显示下一个就绪的故事 |
| **技能更新** | `/design-review` — 移除了架构差距检查（错误阶段） |
| **技能更新** | `/team-ui` — 完整 UX 管线（ux-design → ux-review → team 阶段） |
| **代理更新** | 14 个专业代理——添加了 `memory: project` |
| **代理更新** | `prototyper` — 添加了 `isolation: worktree`（在隔离的 git 分支中进行一次性工作） |
| **模型路由** | Haiku/Sonnet/Opus 层级分配记录在协调规则中；技能在前置元数据中声明其层级 |
| **目录 CLAUDE.md** | 搭建了 `design/CLAUDE.md`、`src/CLAUDE.md`、`docs/CLAUDE.md`——每个目录的路径范围指令 |
| **管线完整性** | TR-ID 稳定性、清单版本控制、ADR 状态关卡、TR-ID 引用而非引用原文 |
| **GDD 模板** | 添加了 `## Game Feel` 章节（输入响应、动画目标、冲击时刻） |

---

### 文件：安全覆盖

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

**要覆盖的现有文件（无用户内容）：**
```
.claude/skills/gate-check/SKILL.md
.claude/skills/sprint-plan/SKILL.md
.claude/skills/sprint-status/SKILL.md
.claude/skills/design-review/SKILL.md
.claude/skills/team-ui/SKILL.md
.claude/skills/story-readiness/SKILL.md
.claude/skills/story-done/SKILL.md
.claude/docs/templates/game-design-document.md    ← 添加了 Game Feel 章节
README.md
docs/WORKFLOW-GUIDE.md
UPGRADING.md
```

**要覆盖的代理文件**（如果你没有在其中编写自定义提示词）：
```
.claude/agents/prototyper.md         ← 添加了 isolation: worktree
.claude/agents/art-director.md       ← 添加了 memory: project
.claude/agents/audio-director.md     ← 添加了 memory: project
.claude/agents/economy-designer.md   ← 添加了 memory: project
.claude/agents/game-designer.md      ← 添加了 memory: project
.claude/agents/gameplay-programmer.md ← 添加了 memory: project
.claude/agents/lead-programmer.md    ← 添加了 memory: project
.claude/agents/level-designer.md     ← 添加了 memory: project
.claude/agents/narrative-director.md ← 添加了 memory: project
.claude/agents/systems-designer.md   ← 添加了 memory: project
.claude/agents/technical-artist.md   ← 添加了 memory: project
.claude/agents/ui-programmer.md      ← 添加了 memory: project
.claude/agents/ux-designer.md        ← 添加了 memory: project
.claude/agents/world-builder.md      ← 添加了 memory: project
```

---

### 文件：仔细合并

#### `.claude/settings.json`

此版本注册了四个新钩子。如果你没有自定义 `settings.json`，覆盖是安全的。否则，请手动添加以下钩子条目：

- `log-agent-stop.sh` — `SubagentStop` 事件（代理审计追踪停止）
- `notify.sh` — `Notification` 事件（Windows 弹窗通知）
- `post-compact.sh` — `PostCompact` 事件（会话恢复提醒）
- `validate-skill-change.sh` — 过滤到 `.claude/skills/` 写入的 `PostToolUse` 事件

#### 自定义代理文件

如果你已向代理 `.md` 文件添加了项目特定知识，请进行差异比较并在适当的 YAML 前置元数据中手动添加 `memory: project` 行。创意和技术总监代理有意保持 `memory: user`——只有专业代理才获得 `memory: project`。

---

### 新功能

#### 完整故事生命周期

故事现在拥有由两项技能强制执行的正式生命周期：

- **`/story-readiness`** — 在开发者接手故事之前验证其是否已准备好实现。检查设计（关联的 GDD 需求）、架构（已接受的 ADR）、范围（标准可测试）和完成定义（清单版本当前）。判定：READY / NEEDS WORK / BLOCKED。
- **`/story-done`** — 实现后的 8 阶段完成审查。验证每个验收标准，检查 GDD/ADR 偏差，提示代码审查，将故事文件更新为 `Status: Complete`，并显示下一个就绪的故事。

流程：`/story-readiness` → 实现 → `/story-done` → 下一个故事

#### 完整 UX/UI 管线

- **`/ux-design`** — 引导式逐章节 UX 规范创作。三种模式：屏幕/流程、HUD 或交互模式库。读取 GDD UI 需求和玩家旅程。输出到 `design/ux/`。
- **`/ux-review`** — 验证 UX 规范与 GDD 对齐、可访问性层级和模式库。判定：APPROVED / NEEDS REVISION / MAJOR REVISION。
- **`/team-ui`** 更新：阶段 1 现在在视觉设计开始前将 `/ux-design` + `/ux-review` 作为硬关卡运行。

#### 棕地采用

**`/adopt`** 将现有项目接入模板格式。审计 GDD、ADR、故事、系统索引和基础设施的内部结构。分类差距（BLOCKING/HIGH/MEDIUM/LOW）。构建有序的迁移计划。从不重新生成现有产物——仅填补空白。

参数模式：`full | gdds | adrs | stories | infra`

另见：`/design-system retrofit [path]` 和 `/architecture-decision retrofit [path]` 检测现有文件并仅添加缺失的章节。

#### 冲刺跟踪 YAML

`production/sprint-status.yaml` 现在是权威的故事跟踪格式：
- 由 `/sprint-plan`（初始化所有故事）和 `/story-done`（将状态设置为 `done`）写入
- 由 `/sprint-status`（快速快照）和 `/help`（生产阶段中每个故事的状态）读取
- 状态值：`backlog | ready-for-dev | in-progress | review | done | blocked`
- 如果文件不存在，优雅地回退到 Markdown 扫描

#### `/help` — 上下文感知的下一步

`/help` 读取你当前阶段和进行中的工作，检查哪些产物已完成，并准确告诉你下一步该做什么——一个主要必需步骤，外加可选的附加机会。与 `/start`（仅首次）和 `/project-stage-detect`（全面审计）不同。

#### 全面质量保证和测试框架

九个新的质量保证/测试技能，涵盖完整的测试生命周期：

- **`/test-setup`** — 为你的引擎搭建测试框架和 CI/CD 管线
- **`/test-helpers`** — 生成特定于引擎的测试辅助库（GDUnit4、NUnit 等）
- **`/qa-plan`** — 为冲刺或功能生成质量保证测试计划，按测试类型对故事进行分类
- **`/smoke-check`** — 在质量保证移交前运行关键路径冒烟测试关卡
- **`/soak-test`** — 为长时间游戏会话生成浸泡测试协议（稳定性、内存泄漏）
- **`/regression-suite`** — 将测试覆盖范围映射到 GDD 关键路径，识别缺少回归测试的已修复 bug
- **`/test-evidence-review`** — 对测试文件和手动证据文档进行质量审查
- **`/test-flakiness`** — 通过读取 CI 运行日志检测非确定性测试
- **`/skill-test`** — 验证技能文件的结构合规性和行为正确性（三种模式：lint、spec、catalog）

新功能还包括：**`/bug-triage`** 重新评估所有未解决的 bug 的优先级、严重性和归属。

#### 技能验证器（`/skill-test`）

`/skill-test` 是一个用于验证工具本身的无技能。在编辑任何技能文件后运行它。三种模式：
- `lint` — 验证 YAML 前置元数据和必填字段
- `spec [skill-name]` — 对特定技能运行行为规范测试
- `catalog` — 检查 `.claude/skills/` 中的所有技能是否已在目录中索引

新的 `validate-skill-change.sh` 钩子会在技能文件被修改时自动提醒你运行 `/skill-test`。

#### 团队运维与团队质量保证编排

- **`/team-live-ops`** — 协调 live-ops-designer + economy-designer + community-manager + analytics-engineer 进行发布后内容规划（季节性活动、战斗通行证、留存）
- **`/team-qa`** — 编排 qa-lead + qa-tester + gameplay-programmer + producer 完成完整的质量保证周期：策略、执行、覆盖和签收

#### 模型层级路由

技能现在根据任务复杂度明确分配给 Haiku、Sonnet 或 Opus 层级。只读状态检查使用 Haiku；复杂的多文档综合使用 Opus；其他所有情况默认使用 Sonnet。层级分配记录在 `.claude/docs/coordination-rules.md`。

#### 目录 CLAUDE.md 文件

三个新的目录范围 CLAUDE.md 文件（`design/`、`src/`、`docs/`）为在这些目录中工作的代理提供了特定路径的指令。当 Claude Code 读取该目录中的文件时，这些指令会自动加载。

---

### 升级后

1. **验证新钩子**已在 `.claude/settings.json` 中注册——检查全部四个：`log-agent-stop.sh`、`notify.sh`、`post-compact.sh`、`validate-skill-change.sh`。

2. **测试审计追踪**——生成任何子代理，开始和停止事件都应出现在 `production/session-logs/` 中。

3. **如果你正处于活跃生产阶段**，请生成 `sprint-status.yaml`：
   ```
   /sprint-plan status
   ```

4. **如果你有早于此模板版本的现有 GDD 或 ADR**，请运行 `/adopt`——它将在不覆盖你的内容的情况下识别需要添加哪些章节。

5. **在对技能进行任何编辑后**，使用 `/skill-test` 验证你的技能——新的 `validate-skill-change.sh` 钩子会自动提醒你执行此操作。

---

## v0.2.0 → v0.3.0

**发布：** 2026-03-09
**提交范围：** `e289ce9..HEAD`
**关键主题：** `/design-system` GDD 创作、`/map-systems` 重命名、自定义状态栏

### 破坏性变更

#### `/design-systems` 重命名为 `/map-systems`

`/design-systems` 技能已重命名为 `/map-systems` 以提高清晰度（分解 = *映射*，而非*设计*）。

**需要操作：** 更新所有调用 `/design-systems` 的文档、说明或脚本。新的调用方式为 `/map-systems`。

### 变更内容

| 类别 | 变更 |
|----------|---------|
| **新技能** | `/design-system`（引导式 GDD 创作，逐章节） |
| **已重命名技能** | `/design-systems` → `/map-systems`（破坏性重命名） |
| **新文件** | `.claude/statusline.sh`、`.claude/settings.json` 状态栏配置 |
| **技能更新** | `/gate-check` — 在 PASS 时写入 `production/stage.txt`，新的阶段定义 |
| **技能更新** | `brainstorm`、`start`、`design-review`、`project-stage-detect`、`setup-engine` — 交叉引用修复 |
| **Bug 修复** | `log-agent.sh`、`validate-commit.sh` — 钩子执行已修复 |
| **文档** | 新增 `UPGRADING.md`，`README.md` 已更新，`WORKFLOW-GUIDE.md` 已更新 |

---

### 文件：安全覆盖

**新增文件：**
```
.claude/skills/design-system/SKILL.md
.claude/statusline.sh
```

**要覆盖的现有文件（无用户内容）：**
```
.claude/skills/map-systems/SKILL.md      ← 原为 design-systems/SKILL.md
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

**删除（被重命名替换）：**
```
.claude/skills/design-systems/   ← 整个目录；由 map-systems/ 替换
```

---

### 文件：仔细合并

#### `.claude/settings.json`

新版本添加了一个指向 `.claude/statusline.sh` 的 `statusLine` 配置块。如果你没有自定义 `settings.json`，覆盖是安全的。否则，请手动添加此块：

```json
"statusLine": {
  "script": ".claude/statusline.sh"
}
```

---

### 新功能

#### 自定义状态栏

`.claude/statusline.sh` 在终端状态栏中显示 7 阶段生产管线的面包屑导航：

```
ctx: 42% | claude-sonnet-4-6 | 系统设计
```

在生产/打磨/发布阶段，如果 `production/session-state/active.md` 中存在 `<!-- STATUS -->` 块，它还会显示活跃的史诗/功能/任务：

```
ctx: 42% | claude-sonnet-4-6 | Production | Combat System > Melee Combat > Hitboxes
```

当前阶段从项目产物中自动检测，或通过将阶段名称写入 `production/stage.txt` 来固定。

#### `/gate-check` 阶段推进

当关卡 PASS 判定被确认时，`/gate-check` 现在将新阶段名称写入 `production/stage.txt`。这会立即使所有未来会话的状态栏保持更新，无需手动编辑文件。

---

### 升级后

1. **删除旧的技能目录：**
   ```bash
   rm -rf .claude/skills/design-systems/
   ```

2. **测试状态栏**——启动一个 Claude Code 会话，你应该能在终端底部看到阶段面包屑。

3. **验证钩子执行**仍然正常工作：
   ```bash
   bash .claude/hooks/log-agent.sh '{}' '{}'
   bash .claude/hooks/validate-commit.sh '{}' '{}'
   ```

---

## v0.1.0 → v0.2.0

**发布：** 2026-02-21
**提交范围：** `ad540fe..e289ce9`
**关键主题：** 上下文弹性、AskUserQuestion 集成、`/map-systems` 技能

### 变更内容

| 类别 | 变更 |
|----------|---------|
| **新技能** | `/start`（入门引导）、`/map-systems`（系统分解）、`/design-system`（引导式 GDD 创作） |
| **新钩子** | `session-start.sh`（恢复）、`detect-gaps.sh`（差距检测） |
| **新模板** | `systems-index.md`、3 个协作协议模板 |
| **上下文管理** | 重大重写——新增文件支持的状态策略 |
| **代理更新** | 14 个设计/创意代理——AskUserQuestion 集成 |
| **技能更新** | 所有 7 个 `team-*` 技能 + `brainstorm`——在阶段转换处添加 AskUserQuestion |
| **CLAUDE.md** | 从约 159 行精简到约 60 行；5 个文档导入替换了 10 个 |
| **钩子更新** | 所有 8 个钩子——Windows 兼容性修复、新功能 |
| **已移除文档** | `docs/IMPROVEMENTS-PROPOSAL.md`、`docs/MULTI-STAGE-DOCUMENT-WORKFLOW.md` |

---

### 文件：安全覆盖

这些是纯基础设施——你没有自定义过它们。直接复制新版本，不会对你的项目内容造成风险。

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

**要覆盖的现有文件（无用户内容）：**
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

**要覆盖的代理文件**（如果你没有在其中编写自定义提示词）：
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

如果你*确实*自定义了代理提示词，请参见下面的"仔细合并"。

---

### 文件：仔细合并

这些文件既包含模板结构，也包含你的项目特定内容。**不要**覆盖它们——请手动合并更改。

#### `CLAUDE.md`

模板版本已从约 159 行精简到约 60 行。关键的结构性更改：移除了 5 个文档导入，因为 Claude Code 会自动加载它们（agent-roster、skills-reference、hooks-reference、rules-reference、review-workflow）。

**从你的版本中保留的内容：**
- `## Technology Stack` 章节（你的引擎/语言选择）
- 你做的任何项目特定添加

**从新版本中采纳的内容：**
- 更精简的导入列表（如果存在，删除 5 个冗余的 `@` 导入）
- 更新后的协作协议措辞

#### `.claude/docs/technical-preferences.md`

如果你运行过 `/setup-engine`，此文件包含你的引擎配置、命名约定和性能预算。全部保留。模板版本只是空占位符。

#### `.claude/docs/templates/game-concept.md`

小的结构性更新——新增了一个指向 `/map-systems` 的 `## Next Steps` 章节。如果你想要更新的指导，请将该章节添加到你的副本中，但不是必需的。

#### `.claude/settings.json`

检查新版本是否添加了你想要的任何权限规则。变化很小（模式更新）。如果你没有自定义 `settings.json`，覆盖是安全的。

#### 自定义代理文件

如果你已向任何代理 `.md` 文件添加了项目特定知识或自定义行为，请进行差异比较并手动添加新的 AskUserQuestion 集成章节，而不是覆盖。每个代理的变化是在系统提示词末尾添加一个标准化的协作协议块。

---

### 文件：删除

这些文件已在 v0.2.0 中移除。如果存在于你的仓库中，可以安全删除——它们已被更有组织的替代方案取代。

```
docs/IMPROVEMENTS-PROPOSAL.md      → 已被 WORKFLOW-GUIDE.md 取代
docs/MULTI-STAGE-DOCUMENT-WORKFLOW.md → 内容已合并到 context-management.md
```

---

### 升级后

1. **运行 `/project-stage-detect`** 以验证系统使用新的检测逻辑正确读取你的项目。

2. **如果你还没有用过 `/start`**，请运行一次——它现在能正确识别你的阶段并跳过你已经完成的入门步骤。

3. **检查 `production/session-state/`** 是否存在并已被 gitignore：
   ```bash
   ls production/session-state/
   cat .gitignore | grep session-state
   ```

4. **测试钩子执行**——如果你在 Windows 上，请验证新钩子在 Git Bash 中运行无误：
   ```bash
   bash .claude/hooks/detect-gaps.sh '{}' '{}'
   bash .claude/hooks/session-start.sh '{}' '{}'
   ```

---

*每个未来版本都将在此文件中拥有自己的章节。*
