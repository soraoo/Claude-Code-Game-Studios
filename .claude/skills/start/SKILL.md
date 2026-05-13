---
name: start
description: "首次上手引导——询问你的起点，然后引导到正确的工作流。不做任何假设。"
argument-hint: "[无参数]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, AskUserQuestion
---

# 引导式上手

此 Skill 写入一个文件：`production/review-mode.txt`（审核模式配置在阶段 3b 设置）。

此 Skill 是新用户的入口点。它不会假设你有游戏创意、引擎偏好或任何先前经验。它先询问，然后引导你到正确的工作流。

---

## 阶段 1：检测项目状态

在询问任何问题之前，静默收集上下文以便定制指导。不要主动显示这些结果——它们用于指导你的建议，而非打开对话的内容。

检查：
- **引擎是否已配置？** 读取 `.claude/docs/technical-preferences.md`。如果 Engine 字段包含 `[TO BE CONFIGURED]`，则引擎尚未设置。
- **游戏概念是否存在？** 检查 `design/gdd/game-concept.md`。
- **源代码是否存在？** 用 Glob 检查 `src/` 中的源文件（`*.gd`、`*.cs`、`*.cpp`、`*.h`、`*.rs`、`*.py`、`*.js`、`*.ts`）。
- **原型是否存在？** 检查 `prototypes/` 中的子目录。
- **设计文档是否存在？** 统计 `design/gdd/` 中的 markdown 文件数量。
- **生产产物？** 检查 `production/sprints/` 或 `production/milestones/` 中的文件。

将这些发现存储在内部分析中，用于验证用户的自我评估并定制建议。

---

## 阶段 2：询问用户起点

这是用户看到的第一件事。使用 `AskUserQuestion` 提供以下精确选项，让用户可以点击而非输入：

- **提示**："欢迎来到 Claude Code Game Studios！在给出任何建议之前，我想了解你的起点。你目前的游戏创意处于什么阶段？"
- **选项**：
  - `A) 还没有想法` — 我完全没有游戏概念。我想探索并找到要制作的东西。
  - `B) 模糊的想法` — 我有一个大致的主题、感觉或类型（例如"太空类游戏"或"慵懒的农场游戏"），但还没有具体内容。
  - `C) 清晰的概念` — 我知道核心创意——类型、基本机制，也许有一句话概括——但还没有将其形式化为文档。
  - `D) 已有工作` — 我已有设计文档、原型、代码或重要的规划。我想整理或继续工作。

等待用户选择。在他们回应之前不要继续。

---

## 阶段 3：根据回答进行路由

#### 如果是 A：还没有想法

用户在进行其他任何操作之前需要进行创意探索。

1. 承认从零开始完全没问题
2. 简要说明 `/brainstorm` 的功能（使用专业框架——MDA、玩家心理学、动词优先设计——进行引导式构思）。提到它有两种模式：`/brainstorm open` 用于完全开放的探索，或 `/brainstorm [提示]` 如果他们有任何模糊的主题（例如"太空"、"休闲"、"恐怖"）。
3. 推荐运行 `/brainstorm open` 作为下一步，但如果他们想到什么的话邀请他们使用提示
4. 展示推荐路径：
   **概念阶段：**
   - `/brainstorm open` — 发现你的游戏概念
   - `/setup-engine` — 配置引擎（brainstorm 将推荐一个）
   - `/art-bible` — 定义视觉标识（使用 brainstorm 生成的视觉标识锚点）
   - `/brainstorm` — 将概念分解为系统
   - `/design-system` — 为每个 MVP 系统编写 GDD
   - cross-GDD consistency check — 跨系统一致性检查
   - `/gate-check` — 在架构工作之前验证准备就绪
   **架构阶段：**
   - — 生成主体架构蓝图和必需的 ADR 列表
   - `/architecture-decision (×N)` — 按照必需的 ADR 列表记录关键技术决策
   - — 将决策编译为可操作的规则清单
   - `/architecture-review` — 验证架构覆盖率
   **预生产阶段：**
   - — 为关键界面编写 UX 规格（主菜单、HUD、核心交互）
   - `/prototype` — 构建可抛弃的原型来验证核心机制
   - `/playtest-report (×1+)` — 记录每次垂直切片试玩会话
   - `/create-epics` — 将系统映射为 Epic
   - `/create-stories` — 将 Epic 分解为可实现的 Story
   - `/sprint-plan` — 规划第一个迭代
   **生产阶段：** → 使用 `/dev-story` 拾取 Story

#### 如果是 B：模糊的想法

1. 请他们分享模糊的想法——即使只有几个字也足够
2. 肯定这个想法作为起点（不要评判或改变方向）
3. 推荐运行 `/brainstorm [他们的提示]` 来发展它
4. 展示推荐路径：
   **概念阶段：**
   - `/brainstorm [提示]` — 将想法发展为完整概念
   - `/setup-engine` — 配置引擎
   - `/art-bible` — 定义视觉标识（使用 brainstorm 生成的视觉标识锚点）
   - `/brainstorm` — 将概念分解为系统
   - `/design-system` — 为每个 MVP 系统编写 GDD
   - cross-GDD consistency check — 跨系统一致性检查
   - `/gate-check` — 在架构工作之前验证准备就绪
   **架构阶段：**
   - — 生成主体架构蓝图和必需的 ADR 列表
   - `/architecture-decision (×N)` — 按照必需的 ADR 列表记录关键技术决策
   - — 将决策编译为可操作的规则清单
   - `/architecture-review` — 验证架构覆盖率
   **预生产阶段：**
   - — 为关键界面编写 UX 规格（主菜单、HUD、核心交互）
   - `/prototype` — 构建可抛弃的原型来验证核心机制
   - `/playtest-report (×1+)` — 记录每次垂直切片试玩会话
   - `/create-epics` — 将系统映射为 Epic
   - `/create-stories` — 将 Epic 分解为可实现的 Story
   - `/sprint-plan` — 规划第一个迭代
   **生产阶段：** → 使用 `/dev-story` 拾取 Story

#### 如果是 C：清晰的概念

1. 请他们用一句话描述概念——类型和核心机制。使用纯文本，不用 AskUserQuestion（这是开放式回答）。
2. 确认概念，然后使用 `AskUserQuestion` 提供两条路径：
   - **提示**："你想如何继续？"
   - **选项**：
     - `先形式化` — 运行 `/brainstorm [概念]` 将其结构化为正式的游戏概念文档
     - `直接开始` — 现在去 `/setup-engine`，之后再手动写 GDD
3. 展示推荐路径：
   **概念阶段：**
   - `/brainstorm` — （他们在第 2 步中的选择）
   - `/art-bible` — 定义视觉标识（在 brainstorm 运行后，或概念文档存在后）
   - `/code-review` — 验证概念文档
   - `/brainstorm` — 将概念分解为单个系统
   - `/design-system` — 为每个 MVP 系统编写 GDD
   - cross-GDD consistency check — 跨系统一致性检查
   - `/gate-check` — 在架构工作之前验证准备就绪
   **架构阶段：**
   - — 生成主体架构蓝图和必需的 ADR 列表
   - `/architecture-decision (×N)` — 按照必需的 ADR 列表记录关键技术决策
   - — 将决策编译为可操作的规则清单
   - `/architecture-review` — 验证架构覆盖率
   **预生产阶段：**
   - — 为关键界面编写 UX 规格（主菜单、HUD、核心交互）
   - `/prototype` — 构建可抛弃的原型来验证核心机制
   - `/playtest-report (×1+)` — 记录每次垂直切片试玩会话
   - `/create-epics` — 将系统映射为 Epic
   - `/create-stories` — 将 Epic 分解为可实现的 Story
   - `/sprint-plan` — 规划第一个迭代
   **生产阶段：** → 使用 `/dev-story` 拾取 Story

#### 如果是 D：已有工作

1. 分享你在阶段 1 中发现的内容：
   - "我可以看到你有 [X 个源文件 / Y 个设计文档 / Z 个原型]..."
   - "你的引擎是 [配置为 X / 尚未配置]..."

2. **子情况 D1 — 早期阶段**（引擎未配置或只有游戏概念文档存在）：
   - 如果引擎未配置，推荐先 `/setup-engine`
   - 然后 `/project-stage-detect` 进行缺口盘点

   **子情况 D2 — GDD、ADR 或 Story 已存在：**
   - 说明："有文件不等于模板的 Skill 能够使用它们。GDD 可能缺少必需的章节。`/project-stage-detect` 专门检查这一点。"
   - 推荐：
     1. `/project-stage-detect` — 了解所处阶段以及完全缺失的内容
     2. `/project-stage-detect` — 审计现有产物是否采用正确的内部格式

3. 展示 D2 的推荐路径：
   - `/project-stage-detect` — 阶段检测 + 存在性缺口
   - `/project-stage-detect` — 格式合规审计 + 迁移计划
   - `/setup-engine` — 如果引擎未配置
   - `/design-system retrofit [path]` — 填补缺失的 GDD 章节
   - `/architecture-decision retrofit [path]` — 添加缺失的 ADR 章节
   - `/architecture-review` — 引导 TR 需求注册表
   - `/gate-check` — 验证下一阶段准备就绪

---

## 阶段 3b：设置审核模式

检查 `production/review-mode.txt` 是否已存在。

**如果存在**：读取并显示当前模式——"审核模式设置为 `[当前模式]`。"——然后继续阶段 4。不要再次询问。

**如果不存在**：使用 `AskUserQuestion`：

- **提示**："一个设置选项：在浏览整个工作流时，你希望进行多少设计审核？"
- **选项**：
  - `完整` — 在每个关键工作流步骤由总监专家审核。最适合团队、学习工作流，或当你希望对每个决策都有全面反馈时。
  - `精简（推荐）` — 仅在阶段关卡过渡时使用总监（/gate-check）。跳过每个 Skill 的审核。适合独立开发者和小团队的平衡方式。
  - `独立` — 完全不进行总监审核。最快速度。最适合游戏 jam、原型开发，或当审核感觉像额外负担时。

在用户选择后立即将选择写入 `production/review-mode.txt`——无需单独的"我可以写入吗？"，因为写入是选择的直接结果：
- `完整` → 写入 `full`
- `精简（推荐）` → 写入 `lean`
- `独立` → 写入 `solo`

如果 `production/` 目录不存在则创建。

---

## 阶段 4：继续之前确认

在展示推荐路径后，使用 `AskUserQuestion` 询问用户想先进行哪一步。绝不自动运行下一个 Skill。

- **提示**："是否要从 [推荐的第一步] 开始？"
- **选项**：
  - `是的，从 [推荐的第一步] 开始`
  - `我想先做别的事`

---

## 阶段 5：移交

当用户确认下一步时，用一句简短的话回应："输入 `[skill 命令]` 开始。"不需要其他内容。不要重新解释 Skill 或添加鼓励的话。`/start` Skill 的工作已经完成。

判决：**完成** — 用户已引导并移交给下一步。

---

## 边缘情况

- **用户选择 D 但项目为空**：温和地引导——"看起来项目是一个还没有任何产物的新模板。路径 A 或 B 更适合吗？"
- **用户选择 A 但项目有代码**：提及你发现的内容——"我注意到 `src/` 中已经有代码。你是否想选 D（已有工作）？"
- **用户是回头客（引擎已配置，概念已存在）**：完全跳过上手引导——"看起来你已经设置好了！你的引擎是 [X]，游戏概念在 `design/gdd/game-concept.md`。审核模式：`[读取自 production/review-mode.txt，如果缺失则为 'lean（默认）']`。想从上次停下的地方继续吗？直接告诉我你想做什么。"
- **用户不适合任何选项**：让他们用自己的话描述情况并适应。

---

## 协作协议

1. **先问** — 绝不假设用户的状态或意图
2. **提供选项** — 给出清晰的路径，而非命令
3. **用户决定** — 他们选择方向
4. **不自动执行** — 推荐下一个 Skill，不经过询问不运行
5. **适应** — 如果用户的情况不符合模板，倾听并调整
