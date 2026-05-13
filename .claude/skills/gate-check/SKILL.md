---
name: gate-check
description: "验证是否准备好进入下一个开发阶段。生成 PASS/CONCERNS/FAIL 评估结果，包含具体阻塞项和所需制品。当用户说'我们准备好进入X了吗'、'可以进入生产阶段吗'、'检查是否能开始下一阶段'、'通过关卡'时使用。"
argument-hint: "[target-phase: systems-design | technical-setup | pre-production | production | polish | release] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Bash, Write, Task, AskUserQuestion
model: opus
---

# 阶段关卡验证

该 Skill 验证项目是否准备好进入下一个开发阶段。它会检查所需的制品、质量标准和阻塞项。

**与 `/project-stage-detect` 的区别**：那个 Skill 是诊断性的（"我们在哪个阶段？"）。这个 Skill 是规定性的（"我们准备好前进了吗？"并给出正式评估结果）。

## 生产阶段（共 7 个）

项目按照以下阶段推进：

1. **概念** — 头脑风暴，游戏概念文档
2. **系统设计** — 系统映射，编写 GDD
3. **技术设置** — 引擎配置，架构决策
4. **预生产** — 原型制作，垂直切片验证
5. **生产** — 功能开发（Epic/Feature/Story 跟踪活跃）
6. **打磨** — 性能优化，试玩，Bug 修复
7. **发布** — 发布准备，认证

**当关卡通过时**，将新阶段名称写入 `production/stage.txt`（单行，例如 `Production`）。这将立即更新状态行。

---

## 1. 解析参数

**目标阶段：** `$ARGUMENTS[0]`（空白 = 自动检测当前阶段，然后验证下一个过渡）

同时确定审查模式（一次性确定，本次运行中所有关卡生成均使用）：
1. 如果传入了 `--review [full|lean|solo]` → 使用该值
2. 否则读取 `production/review-mode.txt` → 使用该值
3. 否则 → 默认为 `lean`

注意：在 `solo` 模式下，总监调用（CD-PHASE-GATE、TD-PHASE-GATE、PR-PHASE-GATE、AD-PHASE-GATE）将被跳过——gate-check 仅进行制品存在性检查。在 `lean` 模式下，所有四位总监仍会运行（阶段关卡是 lean 模式的目的）。

- **带参数**：`/gate-check production` — 验证是否准备好进入该特定阶段
- **不带参数**：使用相同的启发式方法自动检测当前阶段，然后**在运行前与用户确认**：

  使用 `AskUserQuestion`：
  - Prompt: "检测到阶段：**[当前阶段]**。正在运行 [当前] → [下一个] 过渡的关卡。是否正确？"
  - 选项:
    - `[A] 是 — 运行此关卡`
    - `[B] 否 — 选择其他关卡`（如果选择此项，显示第二个小部件列出所有关卡选项：概念 → 系统设计、系统设计 → 技术设置、技术设置 → 预生产、预生产 → 生产、生产 → 打磨、打磨 → 发布）

  未提供参数时不得跳过此确认步骤。

---

## 2. 阶段关卡定义

### 关卡：概念 → 系统设计

**必需制品：**
- [ ] `design/gdd/game-concept.md` 存在且有内容
- [ ] 游戏支柱已定义（在概念文档或 `design/gdd/game-pillars.md` 中）
- [ ] `design/gdd/game-concept.md` 中存在视觉标识锚点章节（来自头脑风暴 Phase 4 art-director 输出）

**质量检查：**
- [ ] 游戏概念已审查（`/code-review` 评估结果不是 MAJOR REVISION NEEDED）
- [ ] 核心循环已描述并达成共识
- [ ] 目标受众已确定
- [ ] 视觉标识锚点包含一行视觉规则和至少 2 个支撑视觉原则

---

### 关卡：系统设计 → 技术设置

**必需制品：**
- [ ] 系统索引存在于 `design/gdd/systems-index.md`，至少列举了 MVP 系统
- [ ] 所有 MVP 级别的 GDD 存在于 `design/gdd/` 中，并分别通过 `/code-review`
- [ ] 跨 GDD 审查报告存在于 `design/gdd/` 中（来自跨 GDD 一致性检查）

**质量检查：**
- [ ] 所有 MVP GDD 通过单独的设计审查（8 个必需章节，评估结果不为 MAJOR REVISION NEEDED）
- [ ] 跨 GDD 一致性检查评估结果不是 FAIL（跨 GDD 一致性和设计理论检查通过）
- [ ] 跨 GDD 一致性检查标记的所有问题已解决或明确接受
- [ ] 系统依赖关系已在系统索引中映射，且双向一致
- [ ] MVP 优先级层级已定义
- [ ] 没有标记过时的 GDD 引用（较早的 GDD 已更新以反映后续 GDD 中做出的决策）

---

### 关卡：技术设置 → 预生产

**必需制品：**
- [ ] 引擎已选定（CLAUDE.md 技术栈不是 `[CHOOSE]`）
- [ ] 技术偏好已配置（`.claude/docs/technical-preferences.md` 已填充）
- [ ] 美术圣经存在于 `design/art/art-bible.md`，至少包含第 1-4 节（视觉标识基础）
- [ ] `docs/architecture/` 中至少有 3 个架构决策记录，涵盖 Foundation 层系统（场景管理、事件架构、存档/读档）
- [ ] 引擎参考文档存在于 `docs/engine-reference/[engine]/` 中
- [ ] 测试框架已初始化：`tests/unit/` 和 `tests/integration/` 目录存在
- [ ] CI/CD 测试工作流存在于 `.github/workflows/tests.yml`（或等效文件）
- [ ] 至少有一个示例测试文件以确认框架可用
- [ ] 主架构文档存在于 `docs/architecture/architecture.md`
- [ ] 架构可追溯性索引存在于 `docs/architecture/architecture-traceability.md`
- [ ] `/architecture-review` 已运行（审查报告文件存在于 `docs/architecture/`）
- [ ] `design/accessibility-requirements.md` 存在，且已确定无障碍层级
- [ ] `design/ux/interaction-patterns.md` 存在（模式库已初始化，即使是最简形式）

**质量检查：**
- [ ] 架构决策涵盖核心系统（渲染、输入、状态管理）
- [ ] 技术偏好已设置命名约定和性能预算
- [ ] 无障碍层级已定义并记录（即使"Basic"也可以接受——未定义则不行）
- [ ] 至少一个屏幕的 UX 规格已启动（通常在技术设置期间设计主菜单或核心 HUD）
- [ ] 所有 ADR 都有**引擎兼容性章节**，并标明了引擎版本
- [ ] 所有 ADR 都有**解决的 GDD 需求章节**，并明确链接到 GDD
- [ ] 没有 ADR 引用 `docs/engine-reference/[engine]/deprecated-apis.md` 中列出的 API
- [ ] 所有 HIGH RISK 引擎领域（按 VERSION.md 定义）已在架构文档中明确处理或标记为开放问题
- [ ] 架构可追溯性矩阵有**零基础层缺口**（在进入预生产之前，所有基础需求必须有 ADR 覆盖）

**ADR 循环依赖检查**：对于 `docs/architecture/` 中的所有 ADR，读取每个 ADR 的"ADR 依赖关系"/"依赖于"章节。构建依赖图（ADR-A → ADR-B 表示 A 依赖于 B）。如果检测到任何循环（例如 A→B→A，或 A→B→C→A）：
- 标记为 **FAIL**："循环 ADR 依赖：[ADR-X] → [ADR-Y] → [ADR-X]。只要循环存在，两者都无法达到 Accepted 状态。移除一个'依赖于'边来打破循环。"

**引擎验证**（首先读取 `docs/engine-reference/[engine]/VERSION.md`）：
- [ ] 涉及截止日期后引擎 API 的 ADR 被标记为 Knowledge Risk: HIGH/MEDIUM
- [ ] `/architecture-review` 引擎审计显示没有使用废弃的 API
- [ ] 所有 ADR 同意使用相同的引擎版本（没有过时的版本引用）

---

### 关卡：预生产 → 生产

**必需制品：**
- [ ] `prototypes/` 中至少有 1 个原型，带有 README
- [ ] 第一个冲刺计划存在于 `production/sprints/`
- [ ] 美术圣经已完成（全部 9 个章节），且 AD-ART-BIBLE 签署评估结果记录在 `design/art/art-bible.md` 中
- [ ] 叙事文档中涉及的关键角色的视觉档案存在
- [ ] 系统索引中所有 MVP 级别的 GDD 已完成
- [ ] 主架构文档存在于 `docs/architecture/architecture.md`
- [ ] `docs/architecture/` 中至少有 3 个覆盖 Foundation 层决策的 ADR
- [ ] 控制清单存在于 `docs/architecture/control-manifest.md`（由已接受的 ADR 生成）
- [ ] Epics 已在 `production/epics/` 中定义，至少包含 Foundation 和 Core 层的 Epic（使用 `/create-epics layer: foundation` 和 `/create-epics layer: core` 创建，然后为每个 Epic 运行 `/create-stories [epic-slug]`）
- [ ] 垂直切片构建存在且可玩（不仅仅是范围定义）
- [ ] 垂直切片已进行至少 3 次试玩（内部即可）
- [ ] 垂直切片试玩报告存在于 `production/playtests/` 或等效位置
- [ ] 关键屏幕的 UX 规格存在：主菜单、核心游戏 HUD（位于 `design/ux/`）、暂停菜单
- [ ] HUD 设计文档存在于 `design/ux/hud.md`（如果游戏有游戏内 HUD）
- [ ] 所有关键屏幕的 UX 规格已通过审查（评估结果为 APPROVED 或已接受的 NEEDS REVISION）

**质量检查：**
- [ ] **核心循环趣味已验证** — 试玩数据确认核心机制有趣，而不仅仅是可运行。明确检查垂直切片试玩报告。
- [ ] UX 规格涵盖来自 MVP 级别 GDD 的所有 UI 需求章节
- [ ] 交互模式库记录了关键屏幕中使用的模式
- [ ] `design/accessibility-requirements.md` 中的无障碍层级已在所有关键屏幕的 UX 规格中得到处理
- [ ] 冲刺计划引用 `production/epics/` 中的真实 Story 文件路径（不仅仅是 GDD —— Story 必须嵌入 GDD 需求 ID + ADR 引用）
- [ ] **垂直切片已完成**，而不仅仅是范围确定 —— 构建展示了端到端的完整核心循环。至少有一个完整的[开始 → 挑战 → 解决]循环可运行。
- [ ] 架构文档在 Foundation 或 Core 层没有未解决的开放问题
- [ ] 所有 ADR 都有标明了引擎版本的引擎兼容性章节
- [ ] 所有 ADR 都有 ADR 依赖关系章节（即使所有字段都是"None"）
- [ ] 手动验证确认 GDD + 架构 + Epics 是连贯一致的（如果最近未运行，则运行跨 GDD 一致性检查）
- [ ] **核心幻想已交付** — 至少有一名试玩者独立描述了与核心系统 GDD 的玩家幻想章节相符的体验（未经提示）。

**垂直切片验证**（如果任何项为 NO，则 FAIL）：
- [ ] 有人在没有开发者指导的情况下完成了核心循环游玩
- [ ] 游戏在开始游玩的前 2 分钟内传达出要做什么
- [ ] 垂直切片构建中不存在关键的"乐趣阻塞" Bug
- [ ] 核心机制交互感觉良好（这是主观检查 —— 询问用户）

> **注意**：如果任何垂直切片验证项为 FAIL，则评估结果自动为 FAIL，无论其他检查结果如何。在没有经验证的垂直切片的情况下推进是游戏开发中生产失败的头号原因（根据来自 155 个项目的 GDC 事后分析数据）。

---

### 关卡：生产 → 打磨

**必需制品：**
- [ ] `src/` 中有组织成子系统的活跃代码
- [ ] 来自 GDD 的所有核心机制已实现（对照 `design/gdd/` 与 `src/`）
- [ ] 主要游戏路径可端到端游玩
- [ ] `tests/unit/` 和 `tests/integration/` 中存在覆盖 Logic 和 Integration Story 的测试文件
- [ ] 本次冲刺的所有 Logic Story 在 `tests/unit/` 中有相应的单元测试文件
- [ ] 冒烟检查已运行，评估结果为 PASS 或 PASS WITH WARNINGS —— 报告存在于 `production/qa/`
- [ ] QA 计划存在于 `production/qa/`（由 `/qa-plan` 生成），覆盖本次冲刺或最终生产冲刺
- [ ] QA 签署报告存在于 `production/qa/`（由 `/qa-plan` 生成），评估结果为 APPROVED 或 APPROVED WITH CONDITIONS
- [ ] `production/playtests/` 中至少有 3 次不同的试玩记录
- [ ] 试玩报告涵盖：新玩家体验、中期游戏系统和难度曲线
- [ ] 游戏概念中的趣味假设已明确验证或修订

**质量检查：**
- [ ] 测试全部通过（通过 Bash 运行测试套件）
- [ ] 任何 Bug 追踪器或已知问题中没有严重/阻塞性 Bug
- [ ] 核心循环按设计运行（与 GDD 验收标准比较）
- [ ] 性能在预算范围内（检查 technical-preferences.md 中的目标值）
- [ ] 试玩发现已审查，关键趣味问题已解决（不仅仅是记录）
- [ ] 没有发现"困惑循环"——游戏中没有超过 50% 的试玩者卡住且不知道原因的地方
- [ ] 难度曲线与难度曲线设计文档相符（如果 `design/difficulty-curve.md` 存在）
- [ ] 所有已实现的屏幕都有相应的 UX 规格（没有"在代码中设计"的屏幕）
- [ ] 交互模式库与实现中使用的所有模式保持同步
- [ ] 无障碍合规性已根据 `design/accessibility-requirements.md` 中承诺的层级验证

---

### 关卡：打磨 → 发布

**必需制品：**
- [ ] 里程碑计划中的所有功能已实现
- [ ] 内容已完成（设计文档中引用的所有关卡、资源、对话都存在）
- [ ] 本地化字符串已外部化（`src/` 中没有硬编码的面向玩家的文本）
- [ ] QA 测试计划存在（`/qa-plan` 输出在 `production/qa/`）
- [ ] QA 签署报告存在（`/qa-plan` 输出 —— APPROVED 或 APPROVED WITH CONDITIONS）
- [ ] 所有 Must Have Story 的测试证据存在（Logic/Integration：测试文件通过；Visual/Feel/UI：签署文档在 `production/qa/evidence/`）
- [ ] 在发布候选构建上冒烟检查干净通过（PASS 评估结果）
- [ ] 自上次冲刺以来没有测试回归（测试套件全部通过）
- [ ] 平衡性数据已审查（运行 `/balance-review`）
- [ ] 发布清单已完成（运行 `/gate-check release`）
- [ ] 商店元数据已准备（如适用）
- [ ] 更新日志 / 补丁说明已起草

**质量检查：**
- [ ] 所有测试通过
- [ ] 在所有目标平台上达到性能目标
- [ ] 没有已知的严重、高或中等级别 Bug
- [ ] 无障碍基础已覆盖（如适用：按键重映射、文本缩放）
- [ ] 所有目标语言的本地化已验证
- [ ] 法律要求已满足（EULA、隐私政策、年龄分级等）
- [ ] 构建干净地编译和打包

---

## 3. 运行关卡检查

**在运行制品检查之前**，如果 `docs/consistency-failures.md` 存在则读取它。提取其 Domain 与目标阶段匹配的条目（例如，如果检查系统设计 → 技术设置，提取经济、战斗或任何 GDD 领域的条目；如果检查技术设置 → 预生产，提取架构、引擎领域的条目）。将这些作为上下文携带——目标领域中反复出现的冲突模式需要对那些特定检查进行更严格的审查。

对于目标关卡中的每个项目：

### 制品检查
- 使用 `Glob` 和 `Read` 验证文件存在且有实质内容
- 不仅要检查存在性，还要验证文件有真实内容（不仅仅是模板标题）
- 对于代码检查，验证目录结构和文件数量

**系统设计 → 技术设置关卡——跨 GDD 审查检查**：
使用 `Glob('design/gdd/gdd-cross-review-*.md')` 查找跨 GDD 一致性检查报告。
如果没有文件匹配，将"跨 GDD 审查报告存在"制品标记为 **FAIL**，并突出显示："在 `design/gdd/` 中未找到跨 GDD 一致性检查报告。在进入技术设置前运行跨 GDD 一致性检查。"
如果找到文件，读取它并检查评估结果行：FAIL 评估结果意味着跨 GDD 一致性检查失败，必须在推进前解决。

### 质量检查
- 对于测试检查：如果配置了测试运行器，通过 `Bash` 运行测试套件
- 对于设计审查检查：`Read` GDD 并检查 8 个必需章节
- 对于性能检查：`Read` technical-preferences.md 并与 `tests/performance/` 中的性能分析数据或最近的性能分析输出进行比较
- 对于本地化检查：`Grep` 搜索 `src/` 中的硬编码字符串

### 交叉引用检查
- 比较 `design/gdd/` 文档与 `src/` 中的实现
- 检查架构文档中引用的每个系统是否有相应的代码
- 验证冲刺计划是否引用真实的工作项

---

## 4. 协作评估

对于无法自动验证的项目，**询问用户**：

- "我无法自动验证核心循环是否好玩。是否已经过试玩测试？"
- "未找到试玩报告。是否做过非正式测试？"
- "性能分析数据不可用。您想运行性能分析吗？"

**对于无法验证的项目，绝不要假设 PASS。** 将它们标记为 MANUAL CHECK NEEDED。

---

## 4b. 总监小组评估

在生成最终评估结果之前，通过 Task 将所有四位总监作为并行的子 Agent 启动，使用并行关卡协议（在独立模式下禁用总监关卡）。同时发出所有四个 Task 调用——不要等待一个完成后再启动下一个。

**并行启动：**

1. **`creative-director`** — 关卡 **CD-PHASE-GATE**
2. **`technical-director`** — 关卡 **TD-PHASE-GATE**
3. **`producer`** — 关卡 **PR-PHASE-GATE**
4. **`art-director`** — 关卡 **AD-PHASE-GATE**

向每个传递：目标阶段名称、存在的制品列表以及该关卡定义中列出的上下文字段。

**收集所有四个响应，然后呈现总监小组摘要：**

```
## 总监小组评估

创意总监：  [READY / CONCERNS / NOT READY]
  [反馈]

技术总监：  [READY / CONCERNS / NOT READY]
  [反馈]

制作人：    [READY / CONCERNS / NOT READY]
  [反馈]

美术总监：  [READY / CONCERNS / NOT READY]
  [反馈]
```

**应用到评估结果：**
- 任何总监返回 NOT READY → 评估结果最低为 FAIL（用户可通过明确确认覆盖）
- 任何总监返回 CONCERNS → 评估结果最低为 CONCERNS
- 全部四位 READY → 有资格获得 PASS（仍需通过第 3 节的制品和质量检查）

---

## 5. 输出评估结果

```
## 关卡检查：[当前阶段] → [目标阶段]

**日期**：[日期]
**检查者**：gate-check skill

### 必需制品：[X/Y 已存在]
- [x] design/gdd/game-concept.md — 存在，2.4KB
- [ ] docs/architecture/ — 缺失（未找到 ADR）
- [x] production/sprints/ — 存在，1 个冲刺计划

### 质量检查：[X/Y 通过]
- [x] GDD 有 8/8 个必需章节
- [ ] 测试 — 失败（tests/unit/ 中有 3 个失败）
- [?] 核心循环试玩 — 需要手动检查

### 阻塞项
1. **没有架构决策记录** — 在进入生产前在 docs/architecture/ 中运行 ADR 创建覆盖核心系统架构的记录。
2. **3 个测试失败** — 在推进前修复 tests/unit/ 中的失败测试。

### 建议
- [解决阻塞项的优先行动]
- [不构成阻塞的可选改进]

### 评估结果：[PASS / CONCERNS / FAIL]
- **PASS**：所有必需制品存在，所有质量检查通过
- **CONCERNS**：存在小缺口但可在下一阶段解决
- **FAIL**：必须在推进前解决关键阻塞项
```

---

## 5a. 验证链

在第 5 阶段起草评估结果后，在最终确定前对其进行质疑。

**第 1 步 — 生成 5 个旨在反驳评估结果的质疑问题：**

对于 **PASS** 草稿：
- "我通过实际读取文件验证了哪些质量检查，又有哪些是推断通过的？"
- "是否有需要手动检查的项在没有用户确认的情况下被我标记为 PASS？"
- "我是否确认所有列出的制品都有真实内容，而不仅仅是空标题？"
- "我否定的任何阻塞项是否实际上可能阻碍阶段成功？"
- "我最没有信心的单项检查是什么，为什么？"

对于 **CONCERNS** 草稿：
- "考虑到项目的当前状态，任何列出的关注点是否可以升级为阻塞项？"
- "该关注点是否能在下一阶段内解决，还是会随着时间推移而加剧？"
- "我是否为了回避更严格的评估结果而将某些 FAIL 条件软化为 CONCERN？"
- "是否有我未检查的制品可能暴露出额外的阻塞项？"
- "即使每个关注点单独看都是小问题，但它们合在一起是否会构成阻塞性问题？"

对于 **FAIL** 草稿：
- "我是否准确区分了硬性阻塞项和强烈建议？"
- "是否有任何 PASS 项我过于宽松？"
- "我是否遗漏了用户应该知道的额外阻塞项？"
- "我能否提供一个通往 PASS 的最小路径——必须改变的特定 3 件事？"
- "失败条件是否可解决，还是它指向更深层次的设计问题？"

**第 2 步 — 独立回答每个问题。**
不要引用草稿评估结果文本——重新检查特定文件或询问用户。

**第 3 步 — 根据需要修订：**
- 如果任何答案揭示了一个遗漏的阻塞项 → 升级评估结果（PASS→CONCERNS 或 CONCERNS→FAIL）
- 如果任何答案揭示了一个被夸大的阻塞项 → 仅在引用具体证据时降级
- 如果答案一致 → 确认评估结果不变

**第 4 步 — 在最终报告输出中注明验证：**
`验证链：已检查 [N] 个问题 — 评估结果 [未改变 / 从 X 修订为 Y]`

---

## 6. PASS 时更新阶段

当评估结果为 **PASS** 且用户确认想要推进时：

1. 将新阶段名称写入 `production/stage.txt`（单行，无尾部换行）
2. 这将立即更新所有未来会话的状态行

示例：如果通过了"预生产 → 生产"关卡：
```bash
echo -n "Production" > production/stage.txt
```

**写入前务必询问**："关卡已通过。我可以将 `production/stage.txt` 更新为 'Production' 吗？"

---

## 7. 结束下一步小部件

在呈现评估结果且任何 stage.txt 更新完成后，使用 `AskUserQuestion` 以结构化的下一步提示结束。

**根据刚运行的关卡定制选项：**

对于 **systems-design PASS**：
```
关卡已通过。接下来您想做什么？
[A] 运行 /create-architecture — 生成您的主架构蓝图和 ADR 工作计划（推荐下一步）
[B] 先设计更多 GDD — 所有 MVP 系统完成后返回此处
[C] 本次会话到此结束
```

> **systems-design PASS 注意**：在编写任何 ADR 之前，下一步是运行 /create-architecture。它会生成主架构文档和优先的 ADR 编写列表。跳過此步骤直接在 docs/architecture/ 中运行 ADR 意味着在没有蓝图的情况下编写 ADR —— 风险自负。

对于 **technical-setup PASS**：
```
关卡已通过。接下来您想做什么？
[A] 开始预生产 — 开始制作垂直切片原型
[B] 先编写更多 ADR — 运行 /architecture-decision [next-system]
[C] 本次会话到此结束
```

对于所有其他关卡，提供该阶段最合理的两个下一步选项加上"到此结束"。

---

## 8. 后续行动

根据评估结果，建议具体的下一步：

- **没有美术圣经？** → `/art-bible` 创建视觉标识规格
- **美术圣经存在但没有资源规格？** → `/asset-spec system:[name]` 从已批准的 GDD 生成每个资源的视觉规格和生成提示词
- **没有游戏概念？** → `/brainstorm` 创建一个
- **没有系统索引？** → `/brainstorm` 将概念分解为系统
- **缺少设计文档？** → 委托给 `game-designer`
- **需要小的设计改动？** → `/quick-design` 用于约 4 小时以内的改动（绕过完整 GDD 流程）
- **没有 UX 规格？** → `/ux-design [screen name]` 编写规格，或 `/team-ui [feature]` 使用完整流程
- **UX 规格未审查？** → `/ux-review [file]` 或 `/ux-review all` 进行验证
- **没有无障碍需求文档？** → 使用 `AskUserQuestion` 提供立即创建的选项：
  - Prompt: "关卡要求 `design/accessibility-requirements.md`。要我根据模板创建吗？"
  - 选项: `立即创建 — 我来选择无障碍层级`, `我自己创建`, `暂时跳过`
  - 如果选择"立即创建"：使用第二个 `AskUserQuestion` 询问层级：
    - Prompt: "哪个无障碍层级适合这个项目？"
    - 选项: `Basic — 仅按键重映射 + 字幕（工作量最低）`, `Standard — Basic + 色盲模式 + 可缩放 UI`, `Comprehensive — Standard + 运动无障碍 + 完整设置菜单`, `Exemplary — Comprehensive + 外部审计 + 完全自定义`
  - 然后使用 `.claude/docs/templates/accessibility-requirements.md` 中的模板，填入所选层级，编写 `design/accessibility-requirements.md`。确认："我可以写入 `design/accessibility-requirements.md` 吗？"
- **没有交互模式库？** → `/ux-design patterns` 初始化
- **GDD 未交叉审查？** → 跨 GDD 一致性检查（在所有 MVP GDD 单独批准后运行）
- **跨 GDD 一致性问题？** → 修复标记的 GDD，然后重新运行跨 GDD 一致性检查
- **没有测试框架？** → 为您的引擎搭建框架
- **当前冲刺没有 QA 计划？** → 在实现开始前运行 `/qa-plan sprint` 生成一个
- **缺少 ADR？** → 对于单个决策在 docs/architecture/ 中运行 ADR
- **没有主架构文档？** → 运行 /create-architecture 获取完整蓝图
- **ADR 缺少引擎兼容性章节？** → 重新运行 ADR 在 docs/architecture/ 中，或手动向现有 ADR 添加引擎兼容性章节
- **缺少控制清单？** → 运行 /create-control-manifest（需要已接受的 ADR）
- **缺少 Epics？** → 运行 `/create-epics layer: foundation` 然后 `/create-epics layer: core`（需要控制清单）
- **Epic 缺少 Stories？** → 运行 `/create-stories [epic-slug]`（每个 Epic 创建后运行）
- **Stories 未准备好实现？** → 运行 `/dev-story` 在开发者接手前验证 Stories
- **测试失败？** → 委托给 `lead-programmer` 或 `qa-tester`
- **没有试玩数据？** → `/playtest-report`
- **试玩次数少于 3 次？** → 在推进前进行更多试玩。使用 `/playtest-report` 结构化发现。
- **没有难度曲线文档？** → 考虑在打磨前在 `design/difficulty-curve.md` 创建一个
- **没有玩家旅程文档？** → 使用玩家旅程模板创建 `design/player-journey.md`
- **需要快速冲刺检查？** → `/sprint-plan` 获取当前冲刺进度快照
- **性能未知？** → 运行 `/profiling`
- **未本地化？** → 运行 `/localization`
- **准备好发布了？** → `/gate-check`

---

## 协作协议

此 Skill 遵循协作设计原则：

1. **先扫描**：检查所有制品和质量关卡
2. **询问未知项**：对于无法验证的事项不要假设 PASS
3. **呈现发现**：显示带状态的完整检查清单
4. **用户决定**：评估结果是建议——用户做最终决定
5. **获取批准**："我可以将此关卡检查报告写入 production/gate-checks/ 吗？"

**绝不要**阻止用户推进——评估结果是建议性的。记录风险，让用户决定是否在存在顾虑的情况下继续。
