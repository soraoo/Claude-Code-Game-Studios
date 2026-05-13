---
name: setup-engine
description: "配置项目的游戏引擎和版本。在 CLAUDE.md 中锁定引擎，检测知识缺口，并在引擎版本超出 LLM 训练数据时通过 WebSearch 填充引擎参考文档。"
argument-hint: "[engine] | [engine version] | refresh | upgrade [old-version] [new-version] | 无参数进入引导选择"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, WebSearch, WebFetch, Task, AskUserQuestion
---

当此 Skill 被调用时：

## 1. 解析参数

四种模式：

- **完整指定**：`/setup-engine godot 4.6` — 提供了引擎和版本
- **仅引擎**：`/setup-engine unity` — 提供了引擎，版本将查询获取
- **无参数**：`/setup-engine` — 完全引导模式（引擎推荐 + 版本）
- **刷新**：`/setup-engine refresh` — 更新参考文档（参见第 10 节）
- **升级**：`/setup-engine upgrade [old-version] [new-version]` — 迁移到新引擎版本（参见第 11 节）

---

## 2. 引导模式（无参数）

如果未指定引擎，运行交互式引擎选择流程：

### 检查现有游戏概念
- 如果存在，读取 `design/gdd/game-concept.md` — 提取游戏类型、范围、平台目标、美术风格、团队规模以及来自 `/brainstorm` 的任何引擎推荐
- 如果不存在概念，告知用户：
  > "未找到游戏概念。建议先运行 `/brainstorm` 来发现你想构建什么——它也会推荐引擎。或者告诉我你的游戏情况，我可以帮你选择。"

### 如果用户想在无概念的情况下选择，按以下顺序提问：

**问题 1 — 先前经验**（始终优先询问，通过 `AskUserQuestion`）：
- 提示："你之前使用过以下哪些引擎？"
- 选项：`Godot` / `Unity` / `Unreal Engine 5` / `多个——我来解释` / `都没有`
- 如果用户选择了特定引擎 → 推荐该引擎。先前经验优先于所有其他因素。与用户确认并跳过决策矩阵。
- 如果选择"都没有"或"多个" → 继续回答以下问题。

**问题 2-6 — 决策矩阵输入**（仅在无先前引擎经验时）：

**问题 2 — 目标平台**（第二顺位询问，始终通过 `AskUserQuestion` — 平台会在所有其他因素之前排除或大幅加权引擎）：
- 提示："你的游戏目标平台是什么？"
- 选项：`PC（Steam / Epic）` / `移动端（iOS / Android）` / `主机` / `网页 / 浏览器` / `多平台`
- 直接用于推荐结果的平台规则：
  - 移动端 → Unity 强烈推荐；Unreal 不适合；Godot 可用于简单的移动游戏
  - 主机 → Unity 或 Unreal；Godot 主机支持需要第三方发行商或大量额外工作
  - 网页 → Godot 可干净导出到网页；Unity WebGL 功能可用；Unreal 网页支持较弱
  - 仅 PC → 所有引擎都可行；由其他因素决定
  - 多平台 → Unity 在 PC/移动端/主机之间移植性最强

1. **什么类型的游戏？**（2D、3D，还是两者兼有？）
2. **主要输入方式？**（键盘/鼠标、手柄、触摸，还是混合？）
3. **团队规模和经验？**（单人新手、单人经验丰富、小团队？）
4. **有偏好的语言吗？**（GDScript、C#、C++、可视化脚本？）
5. **引擎授权的预算？**（仅免费，或商业授权也可以？）

### 生成推荐

不要使用简单的评分矩阵来排除引擎。相反，应根据以下诚实权衡对用户画像进行推理，然后呈现 1-2 个带有完整上下文的推荐。始终让用户做最后选择——绝不强制裁决。

**引擎诚实权衡：**

**Godot 4**
- 真正优势：2D（同类最佳）、风格化/独立 3D、快速迭代、永久免费（MIT）、开源、学习曲线最平缓、最适合希望完全掌控的独立开发者
- 实际限制：3D 生态相比 Unity/Unreal 薄弱（教程、资源、针对 3D 问题的社区答案较少）；大型开放世界 3D 在 Godot 中非常困难且未经充分验证；主机导出需要第三方发行商或大量额外工作；专业岗位市场较小
- 授权事实：真正免费，无任何收入门槛。MIT 许可证意味着你拥有一切。
- 最适合：任何规模的 2D 游戏；风格化/氛围型 3D；封闭式 3D 世界（非开放世界）；学习曲线重要的首个游戏项目；预算在任何规模下都是硬约束的项目

**Unity**
- 真正优势：中等规模 3D 和移动端的行业标准；庞大的资源商店和教程生态；C# 是专业语言；对独立开发者最佳的主机认证支持；几乎所有类型的强大社区
- 实际限制：2023 年的授权争议损害了信任（运行时费用曾被提出后又撤回——政策变更的风险仍然存在）；C# 初始曲线比 GDScript 陡峭；对简单项目来说编辑器比 Godot 更重
- 授权事实：收入低于 20 万美元且安装量低于 20 万时免费（Unity Personal/Plus）。只有当游戏真正成功时才会产生费用——大多数独立游戏从未达到这个门槛。2023 年的争议值得了解，但当前实际条款对大多数独立开发者来说是合理的。
- 最适合：移动游戏；中等规模 3D；目标主机的游戏；有 C# 背景的开发者；需要大型资源商店的项目；2-5 人团队

**Unreal Engine 5**
- 真正优势：同类最佳的 3D 视觉效果（Lumen、Nanite、Chaos 物理）；AAA 级和照片级真实感 3D 的行业标准；大型开放世界支持成熟且经生产验证；Blueprint 可视化脚本降低了 C++ 门槛；适合目标高端 PC 或主机的游戏
- 实际限制：学习曲线最陡峭；编辑器最重（编译时间长、项目体积大）；对风格化/2D/小规模游戏来说大材小用；C++ 确实很难；不适合移动端或网页；总营收超过 100 万美元后收取 5% 版税
- 授权事实：5% 版税仅在每个游戏总营收超过 100 万美元后适用。对于首个游戏或任何未达到 100 万美元的游戏，无需任何费用。这个门槛高到大多数独立开发者永远不会支付。
- 最适合：AAA 级 3D；大型开放世界游戏；照片级真实感视觉效果；有 C++ 经验或愿意使用 Blueprint 的开发者；目标高端 PC/主机且视觉保真度是核心卖点的游戏

**按类型的指导建议**（将其纳入推荐）：
- 2D 任意风格 → Godot 强烈推荐
- 3D 风格化 / 氛围型 / 封闭世界 → Godot 可行，Unity 是可靠的替代方案
- 3D 开放世界（大型、无缝） → Unity 或 Unreal；Godot 在此方面未经生产验证
- 3D 照片级真实感 / AAA 级 → Unreal
- 移动端优先 → Unity 强烈推荐
- 主机优先 → Unity 或 Unreal；Godot 主机支持需要额外工作
- 恐怖 / 叙事 / 步行模拟 → 任何引擎；根据美术风格和团队经验匹配
- 动作 RPG / 类魂 → 3D 用 Unity 或 Unreal；社区支持和资源在这里很重要
- 2D 平台跳跃 → Godot
- 策略 / 俯视角 / RTS → Godot 或 Unity，取决于 2D 还是 3D

**推荐格式：**
1. 展示以用户具体因素为行的比较表格
2. 给出主要推荐及诚实推理
3. 指出最佳替代方案及何时选择它
4. 明确说明："这是起点而非结论——你随时可以迁移引擎，许多开发者在项目之间切换。"
5. 使用 `AskUserQuestion` 确认："这个推荐感觉合适吗，还是想探索其他引擎？"
   - 选项：`[主要引擎]（推荐）` / `[替代引擎]` / `[第三个引擎]` / `进一步探索` / `输入其他内容`

**如果用户选择"进一步探索"：**
使用 `AskUserQuestion` 提供基于概念特定深度探讨的主题。始终从用户的实际概念生成这些选项——不要使用通用选项。至少应始终包括：
- 主要引擎对该概念的具体限制（例如："Godot 3D 对 [类型] 到底能做到什么程度？"）
- 替代引擎对该概念的具体权衡
- 语言选择对该概念技术挑战的影响
- 任何概念相关的技术问题（例如：自适应音频、开放世界流式加载、多人网络同步代码）

用户可以选择多个主题。深入回答每个选中的主题，然后返回引擎确认问题。

---

## 3. 查询当前版本

引擎选定后：

- 如果提供了版本，使用该版本
- 如果未提供版本，使用 WebSearch 查找最新稳定版：
  - 搜索：`"[引擎] latest stable version [当前年份]"`
  - 与用户确认："[引擎] 最新的稳定版是 [版本]。使用这个版本？"

---

## 4. 更新 CLAUDE.md 技术栈

### 语言选择（仅 Godot）

如果选择了 Godot，在展示建议的技术栈之前，先询问用户使用哪种语言：

> "Godot 支持两种主要语言：
>
>   **A) GDScript** — 类似 Python，Godot 原生，迭代最快。最适合初学者、独立开发者和来自 Python 或 Lua 背景的团队。
>   **B) C#** — .NET 8+，Unity 开发者熟悉，IDE 工具支持更强（Rider / Visual Studio），在重逻辑处理上有轻微性能优势。
>   **C) 两者都用** — GDScript 用于游戏玩法/UI 脚本，C# 用于性能关键型系统。高级设置——需要同时安装 .NET SDK 和 Godot。
>
> 该项目主要使用哪种语言？"

记录选择。它决定了 CLAUDE.md 模板、命名约定、专家路由以及在整个项目中为代码文件生成的 Agent。

---

读取 `CLAUDE.md` 并向用户展示建议的技术栈变更。
询问："我可以将这些引擎设置写入 `CLAUDE.md` 吗？"

在获得确认后再进行任何编辑。

更新技术栈部分，将 `[CHOOSE]` 占位符替换为实际值：

**对于 Godot** — 使用上面所选语言对应的模板。参见本 Skill 底部的 **附录 A**，包含全部三种变体（GDScript、C#、两者都用）。

**对于 Unity：**
```markdown
- **Engine**: Unity [version]
- **Language**: C#
- **Build System**: Unity Build Pipeline
- **Asset Pipeline**: Unity Asset Import Pipeline + Addressables
```

**对于 Unreal：**
```markdown
- **Engine**: Unreal Engine [version]
- **Language**: C++（主要）, Blueprint（游戏玩法原型）
- **Build System**: Unreal Build Tool (UBT)
- **Asset Pipeline**: Unreal Content Pipeline
```

---

## 5. 填充技术偏好

更新 CLAUDE.md 后，创建或更新 `.claude/docs/technical-preferences.md`，填入引擎适用的默认值。先读取现有模板，然后填入：

### 引擎与语言部分
- 根据第 4 步选择的引擎填写

### 命名约定（引擎默认值）

**对于 Godot** — 参见 **附录 A** 中的 GDScript、C# 和两者都用变体。

**对于 Unity（C#）：**
- 类：PascalCase（例如 `PlayerController`）
- 公共字段/属性：PascalCase（例如 `MoveSpeed`）
- 私有字段：_camelCase（例如 `_moveSpeed`）
- 方法：PascalCase（例如 `TakeDamage()`）
- 文件：PascalCase 匹配类名（例如 `PlayerController.cs`）
- 常量：PascalCase 或 UPPER_SNAKE_CASE

**对于 Unreal（C++）：**
- 类：带前缀的 PascalCase（`A` 表示 Actor，`U` 表示 UObject，`F` 表示结构体）
- 变量：PascalCase（例如 `MoveSpeed`）
- 函数：PascalCase（例如 `TakeDamage()`）
- 布尔值：`b` 前缀（例如 `bIsAlive`）
- 文件：匹配类名但不含前缀（例如 `PlayerController.h`）

### 输入与平台部分

使用第 2 节收集的答案（或从游戏概念中提取的答案）填充 `## Input & Platform`。使用以下映射推导值：

| 目标平台 | 手柄支持 | 触摸支持 |
|---------|---------|---------|
| 仅 PC | 部分（推荐） | 无 |
| 主机 | 完全 | 无 |
| 移动端 | 无 | 完全 |
| PC + 主机 | 完全 | 无 |
| PC + 移动端 | 部分 | 完全 |
| 网页 | 部分 | 部分 |

对于**主要输入方式**，使用游戏类型的主导输入：
- 动作/RPG/平台跳跃以主机为目标 → 手柄
- 策略/点击式/RTS → 键盘/鼠标
- 移动游戏 → 触摸
- 跨平台 → 询问用户

展示推导的值并请用户在写入前确认或调整。

已填充的部分示例：
```markdown
## Input & Platform
- **Target Platforms**: PC, Console
- **Input Methods**: Keyboard/Mouse, Gamepad
- **Primary Input**: Gamepad
- **Gamepad Support**: Full
- **Touch Support**: None
- **Platform Notes**: 所有 UI 必须支持方向键导航。无纯悬停交互。
```

### 其余部分
- **性能预算**：使用 `AskUserQuestion`：
  - 提示："我应该现在设置默认性能预算，还是留到以后？"
  - 选项：`[A] 现在设置默认值（60fps、16.6ms 帧预算、引擎适用的绘制调用限制）` / `[B] 留作 [待配置]——我确定目标硬件后再设置`
  - 如果选择 [A]：填入建议的默认值。如果选择 [B]：保留为占位符。
- **测试**：建议引擎适用的框架（Godot 用 GUT，Unity 用 NUnit 等）——添加前先询问。
- **禁止模式**：保留为占位符——不要预填。
- **允许的库**：保留为占位符——不要预填项目当前不需要的依赖项。只有当某个库正在被积极集成时才添加到此，而不是推测性地添加。

> **护栏**：切勿在允许的库中添加推测性依赖项。例如，除非 Steam 集成正在本会话中积极开始，否则不要添加 GodotSteam。发布后集成应在相关工作开始时添加到允许的库中，而不是在引擎设置期间。

### 引擎专家路由

同时填充 `technical-preferences.md` 中的 `## Engine Specialists` 部分，为所选引擎设置正确的路由：

**对于 Godot** — 参见 **附录 A** 中匹配所选语言的路由表。

**对于 Unity：**
```markdown
## Engine Specialists
- **Routing Notes**: 为架构和通用 C# 代码审查调用 primary Specialist。为任何 ECS/Jobs/Burst 代码调用 DOTS specialist。为渲染和视觉效果调用 shader specialist。为所有界面实现调用 UI specialist。为资源管理系统调用 Addressables specialist。

### File Extension Routing

| File Extension / Type | Specialist to Spawn |
|-----------------------|---------------------|
```

**对于 Unreal：**
```markdown
## Engine Specialists
- **Routing Notes**: 为 C++ 架构和广泛的引擎决策调用 primary Specialist。为 Blueprint 图形架构和 BP/C++ 边界设计调用 Blueprint specialist。为所有能力和属性代码调用 GAS specialist。为任何多人游戏或网络系统调用 replication specialist。为所有 UI 实现调用 UMG specialist。

### File Extension Routing

| File Extension / Type | Specialist to Spawn |
|-----------------------|---------------------|
```

### 协作步骤
向用户展示填充好的偏好设置。对于 Godot，包括所选语言并注明完整命名约定和路由表的位置：
> "这是 [engine]（[如果 Godot 则注明语言]）的默认技术偏好设置。命名约定和专家路由见此 Skill 的附录 A——我将应用 [GDScript/C#/Both] 变体。你想自定义其中的某些内容，还是我直接保存默认值？"

对于所有其他引擎，直接展示默认值，无需引用附录。

等待批准后再写入文件。

---

## 6. 确定知识缺口

检查引擎版本是否可能超出 LLM 的训练数据。

**已知的大致覆盖范围**（随模型更新而更新）：
- LLM 知识截止日期：**2025 年 5 月**
- Godot：训练数据可能覆盖到 ~4.3
- Unity：训练数据可能覆盖到 ~2023.x / 早期 6000.x
- Unreal：训练数据可能覆盖到 ~5.3 / 早期 5.4

将用户选择的版本与这些基线进行比较：

- **在训练数据范围内** → `低风险` — 参考文档可选但推荐
- **接近边界** → `中风险` — 推荐创建参考文档
- **超出训练数据** → `高风险` — 必须创建参考文档

告知用户其所属类别及原因。

---

## 7. 填充引擎参考文档

### 如果在训练数据范围内（低风险）：

创建最小的 `docs/engine-reference/<engine>/VERSION.md`：

```markdown
# [Engine] — Version Reference

| Field | Value |
|-------|-------|
| **Engine Version** | [version] |
| **Project Pinned** | [today's date] |
| **LLM Knowledge Cutoff** | May 2025 |
| **Risk Level** | LOW — version is within LLM training data |

## Note

This engine version is within the LLM's training data. Engine reference
docs are optional but can be added later if agents suggest incorrect APIs.

Run `/setup-engine refresh` to populate full reference docs at any time.
```

不要创建 breaking-changes.md、deprecated-apis.md 等——它们只会增加上下文成本，价值甚微。

### 如果超出训练数据（中或高风险）：

通过搜索网络创建完整的参考文档集：

1. **搜索官方迁移/升级指南**：
   - `"[engine] [old version] to [new version] migration guide"`
   - `"[engine] [version] breaking changes"`
   - `"[engine] [version] changelog"`
   - `"[engine] [version] deprecated API"`

2. **从官方文档获取并提取**：
   - 从训练截止版本到当前版本之间的每个版本的破坏性变更
   - 已弃用的 API 及其替代方案
   - 新特性和最佳实践

询问："我可以在 `docs/engine-reference/<engine>/` 下创建引擎参考文档吗？"

等待确认后再写入任何文件。

3. **创建完整的参考目录**：
   ```
   docs/engine-reference/<engine>/
   ├── VERSION.md              # 版本锁定 + 知识缺口分析
   ├── breaking-changes.md     # 逐版本的破坏性变更
   ├── deprecated-apis.md      # "不要使用 X → 使用 Y" 对照表
   ├── current-best-practices.md  # 训练截止后的新实践
   └── modules/                # 各子系统参考（按需创建）
   ```

4. **使用网络搜索的真实数据填充每个文件**，遵循现有参考文档建立的格式。每个文件必须有"Last verified: [date]"头部。

5. **对于模块文件**：仅在有重大变更的子系统中创建模块。不要创建空的或内容极少的模块文件。

---

## 8. 更新 CLAUDE.md 导入

询问："我可以更新 `CLAUDE.md` 中的 `@` 导入，使其指向新的引擎参考吗？"

等待确认，然后将"Engine Version Reference"下的 `@` 导入更新为指向正确的引擎：

```markdown
## Engine Version Reference

@docs/engine-reference/<engine>/VERSION.md
```

如果之前的导入指向了不同的引擎（例如，从 Godot 切换到 Unity），更新它。

---

## 9. 更新 Agent 指令

在进行任何编辑之前询问："我可以向引擎专家 Agent 文件中添加一个 Version Awareness 部分吗？"

对于所选引擎的专家 Agent，验证它们是否已有"Version Awareness"部分。如果没有，按照现有 Godot 专家 Agent 中的模式添加一个。

该部分应指示 Agent：
1. 读取 `docs/engine-reference/<engine>/VERSION.md`
2. 在建议代码前检查已弃用的 API
3. 检查相关版本过渡的破坏性变更
4. 使用 WebSearch 验证不确定的 API

---

## 10. 刷新子命令

如果以 `/setup-engine refresh` 调用：

1. 读取现有的 `docs/engine-reference/<engine>/VERSION.md` 获取当前引擎和版本
2. 使用 WebSearch 检查：
   - 自上次验证以来的新引擎发布
   - 更新的迁移指南
   - 新弃用的 API
3. 用新发现更新所有参考文档
4. 在所有修改过的文件上更新"Last verified"日期
5. 报告变更内容

---

## 11. 升级子命令

如果以 `/setup-engine upgrade [old-version] [new-version]` 调用：

### 步骤 1 — 读取当前版本状态

读取 `docs/engine-reference/<engine>/VERSION.md` 确认当前锁定的版本、风险级别以及任何已记录在案的迁移指南 URL。如果未提供 `old-version` 作为参数，则使用此文件中的锁定版本。

### 步骤 2 — 获取迁移指南

使用 WebSearch 和 WebFetch 查找 `old-version` 到 `new-version` 之间的官方迁移指南：

- 搜索：`"[engine] [old-version] to [new-version] migration guide"`
- 搜索：`"[engine] [new-version] breaking changes changelog"`
- 如果 VERSION.md 中已记录迁移指南 URL，则使用之，否则使用搜索找到的 URL。

提取：重命名的 API、移除的 API、变更的默认值、行为变更以及任何"必须迁移"的项目。

### 步骤 3 — 升级前审计

扫描 `src/` 中使用已知在目标版本中已弃用或变更的 API 的代码：

- 使用 Grep 搜索从迁移指南中提取的已弃用 API 名称（例如，旧函数名称、移除的节点类型、变更的属性名称）
- 列出每个匹配的文件，附上找到的具体 API 引用

以表格形式呈现审计结果：

```
Pre-Upgrade Audit: [engine] [old-version] → [new-version]
==========================================================

Files requiring changes:
  File                              | Deprecated API Found       | Effort
  --------------------------------- | -------------------------- | ------
  src/gameplay/player_movement.gd   | old_api_name               | Low
  src/ui/hud.gd                     | removed_node_type          | Medium

Breaking changes to watch for:
  - [change description from migration guide]
  - [change description from migration guide]

Recommended migration order (dependency-sorted):
  1. [system/layer with fewest dependencies first]
  2. [next system]
  ...
```

如果在 `src/` 中未找到已弃用的 API，报告："在 src/ 中未发现已弃用的 API 使用——升级风险可能较低。"

### 步骤 4 — 在更新前确认

在进行任何更改前询问用户：

> "升级前审计完成。发现 [N] 个文件使用了已弃用的 API。
> 是否继续将 VERSION.md 升级到 [new-version]？
> （这将更新锁定的版本并添加迁移说明——不会更改任何源文件。源代码迁移通过手动或 Story 完成。）"

等待用户明确确认后再继续。

### 步骤 5 — 更新 VERSION.md

确认后：

1. 更新 `docs/engine-reference/<engine>/VERSION.md`：
   - `Engine Version` → `[new-version]`
   - `Project Pinned` → 今天的日期
   - `Last Docs Verified` → 今天的日期
   - 如果新版本超出 LLM 知识截止日期，重新评估并更新 `Risk Level` 和 `Post-Cutoff Version Timeline` 表格
   - 添加一个 `## Migration Notes — [old-version] → [new-version]` 部分，包含：迁移指南 URL、关键破坏性变更、此项目中发现的已弃用 API 以及审计中的推荐迁移顺序

2. 如果 `breaking-changes.md` 或 `deprecated-apis.md` 存在于引擎参考目录中，将新版本的变更追加到这些文件末尾。

### 步骤 6 — 升级后提醒

更新 VERSION.md 后，输出：

```
VERSION.md updated: [engine] [old-version] → [new-version]

Next steps:
1. 迁移上述 [N] 个文件中已弃用的 API 使用
2. 在实际升级引擎二进制文件后运行 /setup-engine refresh，
   以验证没有遗漏新的弃用项
3. 运行 /architecture-review — 引擎升级可能使引用特定 API
   或引擎能力的 ADR 失效
4. 如果有任何 ADR 失效，运行 /propagate-design-change 更新下游 Story
```

---

## 12. 输出摘要

设置完成后，输出：

```
Engine Setup Complete
=====================
Engine:          [name] [version]
Language:        [GDScript | C# | GDScript + C# | C# | C++ + Blueprint]
Knowledge Risk:  [LOW/MEDIUM/HIGH]
Reference Docs:  [created/skipped]
CLAUDE.md:       [updated]
Tech Prefs:      [created/updated]
Agent Config:    [verified]

Next Steps:
1. 查阅 docs/engine-reference/<engine>/VERSION.md
2. [如果来自 /brainstorm] 运行 /map-systems 将你的概念分解为各个系统
3. [如果来自 /brainstorm] 运行 /design-system 编写每个系统的 GDD（引导式、逐节完成）
4. [如果来自 /brainstorm] 运行 /prototype [core-mechanic] 测试核心循环
5. [如果是全新开始] 运行 /brainstorm 发现你的游戏概念
6. 创建你的第一个里程碑：/sprint-plan new
```

---

判定：**完成** — 引擎已配置，参考文档已填充。

## 护栏

- 绝不要猜测引擎版本——始终通过 WebSearch 或用户确认验证
- 绝不在未询问的情况下覆盖现有参考文档——追加或更新
- 如果已有针对不同引擎的参考文档，在替换前先询问
- 在对 CLAUDE.md 进行编辑之前，始终向用户展示即将更改的内容
- 如果 WebSearch 返回模糊结果，展示给用户并让他们决定

---

## 附录 A — Godot 语言配置

所有与语言依赖配置相关的 Godot 特定变体。从第 4 节和第 5 节引用——仅在 Godot 是所选引擎时相关。使用与第 4 节所选语言匹配的子节。

---

### A1. CLAUDE.md 技术栈模板

**GDScript：**
```markdown
- **Engine**: Godot [version]
- **Language**: GDScript
- **Build System**: SCons（引擎）, Godot Export Templates
- **Asset Pipeline**: Godot Import System + custom resource pipeline
```

> **护栏**：使用此 GDScript 模板时，语言字段请准确写入 "`GDScript`" — 不要添加任何额外内容。不要追加 "C++ via GDExtension" 或任何其他语言。下面的 C# 模板包含 GDExtension，因为 C# 项目通常会包装原生代码；GDScript 项目不会。

**C#：**
```markdown
- **Engine**: Godot [version]
- **Language**: C# (.NET 8+, primary), C++ via GDExtension (仅原生插件)
- **Build System**: .NET SDK + Godot Export Templates
- **Asset Pipeline**: Godot Import System + custom resource pipeline
```

**两者都用 — GDScript + C#：**
```markdown
- **Engine**: Godot [version]
- **Language**: GDScript（游戏玩法/UI 脚本）, C#（性能关键型系统）, C++ via GDExtension（仅原生）
- **Build System**: .NET SDK + Godot Export Templates
- **Asset Pipeline**: Godot Import System + custom resource pipeline
```

---

### A2. 命名约定

**GDScript：**
- 类：PascalCase（例如 `PlayerController`）
- 变量/函数：snake_case（例如 `move_speed`）
- 信号：snake_case 过去式（例如 `health_changed`）
- 文件：snake_case 匹配类名（例如 `player_controller.gd`）
- 场景：PascalCase 匹配根节点（例如 `PlayerController.tscn`）
- 常量：UPPER_SNAKE_CASE（例如 `MAX_HEALTH`）

**C#：**
- 类：PascalCase（`PlayerController`）— 也必须为 `partial`
- 公共属性/字段：PascalCase（`MoveSpeed`, `JumpVelocity`）
- 私有字段：`_camelCase`（`_currentHealth`, `_isGrounded`）
- 方法：PascalCase（`TakeDamage()`, `GetCurrentHealth()`）
- 信号委托：PascalCase + `EventHandler` 后缀（`HealthChangedEventHandler`）
- 文件：PascalCase 匹配类名（`PlayerController.cs`）
- 场景：PascalCase 匹配根节点（`PlayerController.tscn`）
- 常量：PascalCase（`MaxHealth`, `DefaultMoveSpeed`）

**两者都用 — GDScript + C#：**
`.gd` 文件使用 GDScript 约定，`.cs` 文件使用 C# 约定。不存在混合语言文件——边界是按文件划分的。如果不确定新系统应使用哪种语言，询问用户并将决定记录在 `technical-preferences.md` 中。

---

### A3. 引擎专家路由

**GDScript：**
```markdown
## Engine Specialists
- **Primary**: godot-specialist
- **UI Specialist**: godot-specialist（无专用 UI specialist — primary 覆盖所有 UI）
- **Routing Notes**: 为架构决策、ADR 验证和跨领域代码审查调用 primary Specialist。为代码质量、信号架构、静态类型强制检查和 GDScript 惯用模式调用 GDScript specialist。为材质设计和着色器代码调用 shader specialist。仅当涉及原生扩展时才调用 GDExtension specialist。

### File Extension Routing

| File Extension / Type | Specialist to Spawn |
|-----------------------|---------------------|
| UI / screen files (Control nodes, CanvasLayer) | godot-specialist |
| Scene / prefab / level files (.tscn, .tres) | godot-specialist |
| General architecture review | godot-specialist |
```

**C#：**
```markdown
## Engine Specialists
- **Primary**: godot-specialist
- **UI Specialist**: godot-specialist（无专用 UI specialist — primary 覆盖所有 UI）
- **Routing Notes**: 为架构决策、ADR 验证和跨领域代码审查调用 primary Specialist。为代码质量、[Signal] 委托模式、[Export] 属性、.csproj 管理和 C# 特定的 Godot 惯用模式调用 C# specialist。为材质设计和着色器代码调用 shader specialist。仅当涉及原生 C++ 插件时才调用 GDExtension specialist。

### File Extension Routing

| File Extension / Type | Specialist to Spawn |
|-----------------------|---------------------|
| UI / screen files (Control nodes, CanvasLayer) | godot-specialist |
| Scene / prefab / level files (.tscn, .tres) | godot-specialist |
| General architecture review | godot-specialist |
```

**两者都用 — GDScript + C#：**
```markdown
## Engine Specialists
- **Primary**: godot-specialist
- **UI Specialist**: godot-specialist（无专用 UI specialist — primary 覆盖所有 UI）
- **Routing Notes**: 为跨语言架构决策以及哪些系统应属于哪种语言调用 primary Specialist。为 .gd 文件调用 GDScript specialist。为 .cs 文件和 .csproj 管理调用 C# specialist。在边界处优先使用信号而非直接的跨语言方法调用。

### File Extension Routing

| File Extension / Type | Specialist to Spawn |
|-----------------------|---------------------|
| Cross-language boundary decisions | godot-specialist |
| UI / screen files (Control nodes, CanvasLayer) | godot-specialist |
| Scene / prefab / level files (.tscn, .tres) | godot-specialist |
| General architecture review | godot-specialist |
```
