---
name: create-epics
description: "将已批准的 GDD 和架构转化为 Epic——每个架构模块一个 Epic。定义范围、管辖的 ADR、引擎风险和未追踪的需求。不分解为 Story——在每个 Epic 创建后运行 /create-stories [epic-slug]。"
argument-hint: "[system-name | layer: foundation|core|feature|presentation | all] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Task, AskUserQuestion
agent: technical-director
---

# 创建 Epic

Epic 是一个命名的、有边界的工作体，映射到一个架构模块。它定义需要构建**什么**以及**谁**在架构上负责。它不规定实现步骤——那是 Story 的工作。

**对每个层级运行一次此 Skill**，在开发接近该层级时。在核心层接近完成之前不要创建功能层 Epic——设计将会发生变化。

**输出：** `production/epics/[epic-slug]/EPIC.md` + `production/epics/index.md`

**每个 Epic 后的下一步：** `/create-stories [epic-slug]`

**何时运行：** 在 GDD 审查和架构审查通过后。

---

## 1. 解析参数

解析审查模式（一次，存储供本次运行的所有关卡派生使用）：
1. 如果传入了 `--review [full|lean|solo]` → 使用该值
2. 否则读取 `production/review-mode.txt` → 使用该值
3. 否则 → 默认为 `lean`



**模式：**
- `/create-epics all` — 按层级顺序处理所有系统
- `/create-epics layer: foundation` — 仅基础层
- `/create-epics layer: core` — 仅核心层
- `/create-epics layer: feature` — 仅功能层
- `/create-epics layer: presentation` — 仅表现层
- `/create-epics [system-name]` — 一个特定系统
- 无参数 — 询问："你想为哪个层级或系统创建 Epic？"

---

## 2. 加载输入

### 步骤 2a——摘要扫描（快速）

在完整阅读任何内容之前对所有 GDD 的 `## 摘要` 章节进行 Grep：

```
Grep pattern="## 摘要" glob="design/gdd/*.md" output_mode="content" -A 5
```

对于 `layer:` 或 `[system-name]` 模式：根据摘要快速参考仅过滤范围内的 GDD。跳过完整阅读任何范围外的内容。

### 步骤 2b——完整文档加载（仅范围内系统）

使用步骤 2a 的 grep 结果，识别哪些系统在范围内。仅对范围内系统阅读完整文档——不要阅读范围外系统或层级的 GDD 或 ADR。

为范围内系统阅读：

- `design/gdd/systems-index.md`——权威系统列表、层级、优先级
- 仅范围内的 GDD（已批准或已设计状态，由步骤 2a 结果过滤）
- `docs/architecture/architecture.md`——模块所有权和 API 边界
- 仅覆盖范围内系统领域的已接受 ADR——阅读"已解决的 GDD 需求"、"决策"和"引擎兼容性"章节；跳过不相关领域的 ADR
- `docs/architecture/control-manifest.md`——来自头部的清单版本日期
- `docs/architecture/tr-registry.yaml`——用于追踪需求到 ADR 覆盖
- `docs/engine-reference/[engine]/VERSION.md`——引擎名称、版本、风险级别

报告："已加载 [N] 个 GDD，[M] 个 ADR，引擎：[名称 + 版本]。"

---

## 3. 处理顺序

按依赖安全的层级顺序处理：
1. **基础层**（无依赖）
2. **核心层**（依赖基础层）
3. **功能层**（依赖核心层）
4. **表现层**（依赖功能层 + 核心层）

每层内，使用 `systems-index.md` 中的顺序。

---

## 4. 定义每个 Epic

对每个系统，将其映射到 `architecture.md` 中的一个架构模块。

对照 TR 注册表检查 ADR 覆盖：
- **已追踪需求**：有覆盖它们的已接受 ADR 的 TR-ID
- **未追踪需求**：没有 ADR 的 TR-ID——在继续之前警告

在写入任何内容之前向用户展示：

```
## Epic：[系统名称]

**层级**：[基础 / 核心 / 功能 / 表现]
**GDD**：design/gdd/[文件名].md
**架构模块**：[来自 architecture.md 的模块名称]
**管辖的 ADR**：[ADR-NNNN, ADR-MMMM]
**引擎风险**：[低 / 中 / 高——管辖 ADR 中的最高风险]
**ADR 覆盖的 GDD 需求**：[N / 总数]
**未追踪需求**：[列出没有 ADR 的 TR-ID，或"无"]
```

如果存在未追踪需求：
> "⚠️ [系统] 中有 [N] 个需求没有 ADR。Epic 可以创建，但这些需求的 Story 将被标记为 Blocked，直到 ADR 存在。先在 docs/architecture/ 中创建 ADR，或使用占位符继续。"

询问："我应该创建 Epic：[名称] 吗？"
选项："是，创建它"、"跳过"、"暂停——我需要先写 ADR"

---

## 4b. 制作人 Epic 结构关卡

**审查模式检查**——在派生 PR-EPIC 之前应用：
- `solo` → 跳过。注明："PR-EPIC 已跳过——单人模式。"进入步骤 5（写入 Epic 文件）。
- `lean` → 跳过（非 PHASE-GATE）。注明："PR-EPIC 已跳过——精简模式。"进入步骤 5（写入 Epic 文件）。
- `full` → 正常派生。

在当前层级的所有 Epic 定义完成后（步骤 4 对所有范围内系统完成），在写入任何文件之前，审查生产可行性。为用户标记问题。

传递：完整的 Epic 结构摘要（所有 Epic、其范围摘要、管辖的 ADR 数量）、正在处理的层级、里程碑时间线和团队产能。

展示可行性评估。如果 UNREALISTIC，提供在写入前修改 Epic 边界的选项（拆分过度范围的或合并范围不足的 Epic）。如果是 CONCERNS，提出它们并让用户决定。在制作人关卡解决之前不要写入 Epic 文件。

---

## 5. 写入 Epic 文件

批准后，询问："我可以将 Epic 文件写入 `production/epics/[epic-slug]/EPIC.md` 吗？"

用户确认后，写入：

### `production/epics/[epic-slug]/EPIC.md`

```markdown
# Epic：[系统名称]

> **层级**：[基础 / 核心 / 功能 / 表现]
> **GDD**：design/gdd/[文件名].md
> **架构模块**：[模块名称]
> **状态**：就绪
> **Story**：尚未创建——运行 `/create-stories [epic-slug]`

## 概述

[1 段话描述此 Epic 实现什么，源自 GDD 概述和架构模块的职责声明]

## 管辖的 ADR

| ADR | 决策摘要 | 引擎风险 |
|-----|-----------------|-------------|
| ADR-NNNN：[标题] | [1 行摘要] | 低/中/高 |

## GDD 需求

| TR-ID | 需求 | ADR 覆盖 |
|-------|-------------|--------------|
| TR-[系统]-001 | [来自注册表的需求文本] | ADR-NNNN ✅ |
| TR-[系统]-002 | [需求文本] | ❌ 无 ADR |

## 完成定义

此 Epic 完成当：
- 所有 Story 已实现、审查并通过 `/story-done` 关闭
- `design/gdd/[文件名].md` 中的所有验收标准已验证
- 所有 Logic 和 Integration Story 在 `tests/` 中有通过的测试文件
- 所有 Visual/Feel 和 UI Story 在 `production/qa/evidence/` 中有带签字的证据文档

## 下一步

运行 `/create-stories [epic-slug]` 将此 Epic 分解为可实现的 Story。
```

### 更新 `production/epics/index.md`

创建或更新主索引。

---

## 6. 关卡检查提醒

在对请求范围的所有 Epic 写入后：

- **基础层 + 核心层完成**：这些是预生产→生产关卡所必需的。运行 `/gate-check production` 检查就绪状态。
- **提醒**：Epic 定义范围。Story 定义实现步骤。在开发者可以开始工作之前，为每个 Epic 运行 `/create-stories [epic-slug]`。

---

## 协作协议

1. **一次一个 Epic**——在要求创建每个 Epic 之前展示其定义
2. **对空白发出警告**——在继续之前标记未追踪的需求
3. **写入前询问**——写入任何文件之前获取逐 Epic 的批准
4. **不凭空创造**——所有内容来自 GDD、ADR 和架构文档
5. **绝不创建 Story**——此 Skill 停在 Epic 级别

处理完所有请求的 Epic 后：

- **判定：COMPLETE**——[N] 个 Epic 已写入。运行 `/create-stories [epic-slug]` 每个 Epic。
- **判定：BLOCKED**——用户拒绝了所有 Epic，或未找到符合条件的系统。
