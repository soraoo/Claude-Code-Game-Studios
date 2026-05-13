---
name: story-done
description: "Story 完成审查。读取 Story 文件，对照实现验证每个验收标准，检查 GDD/ADR 偏差，提示代码审查，将 Story 状态更新为完成，并展示 Sprint 中下一个就绪的 Story。"
argument-hint: "[story-file-path] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Bash, Write, Edit, AskUserQuestion, Task
---

# Story Done

此 Skill 在设计实现之间闭环。在实现任何 Story 结束时运行。它确保在将 Story 标记为完成之前验证每个验收标准，GDD 和 ADR 偏差被显式记录而非悄然引入，代码审查被提示而非遗忘，且 Story 文件反映实际的完成状态。

**输出：** 更新后的 Story 文件（状态：Complete）+ 展示下一个 Story。

---

## 阶段 1：查找 Story

解析审查模式（一次性执行，存储供本次运行的所有关卡使用）：
1. 如果传入了 `--review [full|lean|solo]` → 使用该值
2. 否则读取 `production/review-mode.txt` → 使用该值
3. 否则 → 默认为 `lean`

**如果提供了文件路径**（例如 `/story-done production/epics/core/story-damage-calculator.md`）：
直接读取该文件。

**如果未提供参数：**

1. 检查 `production/session-state/active.md` 中当前活跃的 Story。
2. 如果未找到，读取 `production/sprints/` 中最新的文件，查找标记为 IN PROGRESS 的 Story。
3. 如果找到多个进行中的 Story，使用 `AskUserQuestion`：
   - "我们要完成哪个 Story？"
   - 选项：列出进行中的 Story 文件名。
4. 如果找不到任何 Story，要求用户提供路径。

---

## 阶段 2：读取 Story

读取完整的 Story 文件。提取并在上下文中保存：

- **Story 名称和 ID**
- **引用的 GDD 需求 TR-ID**（例如 `TR-combat-001`）
- **Story 头部中的 Manifest 版本**（例如 `2026-03-10`）
- **引用的 ADR**
- **验收标准** — 完整列表（每个复选框项）
- **实现文件** — "要创建/修改的文件"下列出的文件
- **Story 类型** — Story 头部的 `Type:` 字段（Logic / Integration / Visual/Feel / UI / Config/Data）
- **引擎说明** — 任何记录的引擎特定约束
- **完成定义** — 如果存在，Story 级别的 DoD
- **预估与实际范围** — 如果记录了预估

同时读取：
- `docs/architecture/tr-registry.yaml` — 查找 Story 中的每个 TR-ID。从注册表条目读取*当前*的 `requirement` 文本。这是 GDD 要求的真实来源——不要使用 Story 中可能内联引用的任何需求文本（可能已过时）。
- 引用的 GDD 章节 — 仅验收标准和关键规则，而非完整文档。用于交叉验证注册表文本仍然准确。
- 引用的 ADR — 仅决策和结论部分
- `docs/architecture/control-manifest.md` 头部 — 提取当前的 `Manifest Version:` 日期（在阶段 4 的过时检查中使用）

---

## 阶段 3：验证验收标准

对于 Story 中的每个验收标准，尝试使用以下三种方法之一进行验证：

### 自动验证（无需询问直接运行）

- **文件存在检查**：使用 `Glob` 检查 Story 声明要创建的文件。
- **测试通过检查**：如果提到了测试文件路径，通过 `Bash` 运行它。
- **无硬编码值检查**：使用 `Grep` 搜索应位于配置文件中的游戏玩法代码路径中的数字字面量。
- **无硬编码字符串检查**：使用 `Grep` 搜索 `src/` 中应位于本地化文件中的面向玩家的字符串。
- **依赖检查**：如果某个标准说"依赖于 X"，检查 X 是否存在。

### 带确认的手动验证（使用 `AskUserQuestion`）

- 关于主观品质的标准（"感觉响应灵敏"、"动画播放正确"）
- 关于游戏行为的标准（"玩家在...时受到伤害"、"敌人对...做出响应"）
- 性能标准（"在 Xms 内完成"）——询问是否已分析或假定通过

将最多 4 个手动验证问题批量合并到单个 `AskUserQuestion` 调用中：

```
question: "是否满足 [标准]？"
options: "是 — 通过", "否 — 未通过", "尚未测试"
```

### 无法验证（标记但不阻塞）

- 需要完整游戏构建才能测试的标准（端到端游戏场景）
- 标记为：`DEFERRED — 需要试玩会话`

### 测试与标准的可追溯性

在完成上述通过/失败/延迟检查后，将每个验收标准映射到覆盖它的测试：

对于 Story 中的每个验收标准：

1. 询问：是否有测试——单元测试、集成测试或确认的手动试玩——直接验证此标准？
   - **单元测试**：检查 `tests/unit/` 中是否有与标准主题匹配的测试文件或函数名（使用 `Glob` 和 `Grep`）
   - **集成测试**：类似地检查 `tests/integration/`
   - **手动确认**：如果上述通过 `AskUserQuestion` 验证了该标准且答案为"是 — 通过"，则将其计为手动测试

2. 生成可追溯性表格：

```
| 标准 | 测试 | 状态 |
|-----------|------|--------|
| AC-1：[标准文本] | tests/unit/test_foo.gd::test_bar | COVERED |
| AC-2：[标准文本] | 手动试玩确认 | COVERED |
| AC-3：[标准文本] | — | UNTESTED |
```

3. 应用以下升级规则：

   - 如果 **超过 50% 的标准是 UNTESTED**：升级为 **BLOCKING** —— 测试覆盖率不足以确认 Story 实际完成。在覆盖率改善之前，阶段 6 中的判定不能为 COMPLETE。
   - 如果 **部分（不超过 50%）标准是 UNTESTED**：保持 **ADVISORY** —— 不阻塞完成，但必须在完成说明中出现。
   - 如果 **所有标准都是 COVERED**：除在报告中包含表格外，无需其他操作。

4. 对于任何 ADVISORY 级别的未测试标准，添加到阶段 7 的完成说明中：
   `"未测试的标准：[AC-N 列表]。建议在后续 Story 中添加测试。"`

### 测试证据要求

根据阶段 2 中提取的 Story 类型，检查所需的证据：

| Story 类型 | 所需证据 | 关卡级别 |
|---|---|---|
| **Logic** | `tests/unit/[system]/` 中的自动化单元测试 —— 必须存在且通过 | BLOCKING |
| **Integration** | `tests/integration/[system]/` 中的集成测试或试玩文档 | BLOCKING |
| **Visual/Feel** | `production/qa/evidence/` 中的截图 + 签字确认 | ADVISORY |
| **UI** | `production/qa/evidence/` 中的手动演练文档或交互测试 | ADVISORY |
| **Config/Data** | `production/qa/smoke-*.md` 中的冒烟检查通过报告 | ADVISORY |

**对于 Logic Story**：首先读取 Story 的**测试证据**章节，提取确切的所需文件路径。使用 `Glob` 检查该确切路径。如果未找到确切路径，也在 `tests/unit/[system]/` 中广泛搜索（文件可能放置在略有不同的位置）。如果在任一位置都未找到测试文件：
- 标记为 **BLOCKING**："Logic Story 没有单元测试文件。Story 要求在 `[测试证据章节中的确切路径]` 处有此文件。在将此 Story 标记为完成之前，创建并运行测试。"

**对于 Integration Story**：读取 Story 的**测试证据**章节，获取确切所需路径。先使用 `Glob` 检查该确切路径，然后在 `tests/integration/[system]/` 中广泛搜索，再检查 `production/session-logs/` 中是否有引用此 Story 的试玩记录。如果都未找到：标记为 **BLOCKING**（与 Logic 相同规则）。

**对于 Visual/Feel 和 UI Story**：在 `production/qa/evidence/` 中搜索引用此 Story 的文件。如果未找到：标记为 **ADVISORY** —— "未找到手动测试证据。在最终关闭前，使用测试证据模板创建 `production/qa/evidence/[story-slug]-evidence.md` 并获取签字确认。"

**对于 Config/Data Story**：检查是否存在任何 `production/qa/smoke-*.md` 文件。如果未找到：标记为 **ADVISORY** —— "未找到冒烟检查报告。运行 `/smoke-check`。"

**如果未设置 Story 类型**：标记为 **ADVISORY** —— "Story 类型未声明。在 Story 头部添加 `Type: [Logic|Integration|Visual/Feel|UI|Config/Data]` 以在未来的 Story 中启用测试证据关卡强制执行。"

任何 BLOCKING 级别的测试证据缺失会阻止阶段 6 中的 COMPLETE 判定。

---

## 阶段 4：检查偏差

比较实现与设计文档。

自动运行以下检查：

1. **GDD 规则检查**：使用来自 `tr-registry.yaml` 的当前需求文本（通过 Story 的 TR-ID 查找），检查实现是否反映了 GDD 当前的实际要求——而非 Story 编写时的要求。使用 `Grep` 搜索实现文件中当前 GDD 章节提到的关键函数名、数据结构或类名。

2. **Manifest 版本过时检查**：比较 Story 头部嵌入的 `Manifest Version:` 日期与当前 `docs/architecture/control-manifest.md` 头部的 `Manifest Version:` 日期。
   - 如果匹配 → 静默通过。
   - 如果 Story 的版本较旧 → 标记为 ADVISORY：
     `ADVISORY：Story 是基于 manifest v[story-date] 编写的；当前 manifest 是 v[current-date]。可能适用新规则。运行 /story-readiness 进行检查。`
   - 如果 control-manifest.md 不存在 → 跳过此检查。

3. **ADR 约束检查**：读取引用的 ADR 的决策章节。检查 `docs/architecture/control-manifest.md`（如果存在）中的禁止模式。使用 `Grep` 搜索 ADR 中明确禁止的模式。

4. **硬编码值检查**：使用 `Grep` 搜索实现文件中应位于数据文件的游戏逻辑数字字面量。

5. **范围检查**：实现是否触及了 Story 声明范围之外的文件？（未列入"要创建/修改的文件"的文件）

对于发现的每个偏差，分类：

- **BLOCKING** — 实现与 GDD 或 ADR 相矛盾（在标记完成前必须修复）
- **ADVISORY** — 实现与规范略有偏离但在功能上等价（记录，由用户决定）
- **超出范围** — 在 Story 声明的边界之外触及了额外文件（标记以便知晓——可能是有效的，也可能是范围蔓延）

---

## 阶段 4b：QA 覆盖关卡

**审查模式检查** — 在生成 QL-TEST-COVERAGE 之前应用：
- `solo` → 跳过。注明："QL-TEST-COVERAGE 已跳过 — Solo 模式。"继续阶段 5。
- `lean` → 跳过（非 PHASE-GATE）。注明："QL-TEST-COVERAGE 已跳过 — Lean 模式。"继续阶段 5。
- `full` → 正常生成。

在完成阶段 4 的偏差检查后，审查测试覆盖率和就绪程度。向用户标记缺口。

传递：
- Story 文件路径和 Story 类型
- 阶段 3 中发现的测试文件路径（确切路径，或"未找到"）
- Story 的 `## QA Test Cases` 章节（Story 创建时预先编写的测试规格）
- Story 的 `## Acceptance Criteria` 列表

qa-tester 审查测试是否实际覆盖了指定的内容——而不仅仅是文件是否存在。

应用判定：
- **ADEQUATE（充分）** → 继续阶段 5
- **GAPS（有缺口）** → 标记为 **ADVISORY**："QA 负责人识别出覆盖缺口：[列表]。Story 可以完成，但缺口应在后续 Story 中处理。"
- **INADEQUATE（不充分）** → 标记为 **BLOCKING**："QA 负责人：关键逻辑未经测试。在覆盖率改善之前判定不能为 COMPLETE。具体缺口：[列表]。"

Config/Data Story 跳过此阶段（不需要代码测试）。

---

## 阶段 5：首席程序员代码审查关卡

**审查模式检查** — 在生成 LP-CODE-REVIEW 之前应用：
- `solo` → 跳过。注明："LP-CODE-REVIEW 已跳过 — Solo 模式。"继续阶段 6（完成报告）。
- `lean` → 跳过（非 PHASE-GATE）。注明："LP-CODE-REVIEW 已跳过 — Lean 模式。"继续阶段 6（完成报告）。
- `full` → 正常生成。

通过 Task 使用关卡 **LP-CODE-REVIEW** 生成 `lead-programmer`。

传递：实现文件路径、Story 文件路径、相关 GDD 章节、管辖 ADR。

向用户展示判定。如果有 CONCERNS（担忧），通过 `AskUserQuestion` 展示：
- 选项：`修正标记的问题` / `接受并继续` / `进一步讨论`
如果是 REJECT（拒绝），在问题解决之前不继续到阶段 6 判定。

如果 Story 还没有实现文件（判定在编码完成之前运行），跳过此阶段并注明："LP-CODE-REVIEW 已跳过 — 未找到实现文件。在实现完成后运行。"

---

## 阶段 6：展示完成报告

在更新任何文件之前，展示完整报告：

```markdown
## Story Done：[Story 名称]
**Story**：[文件路径]
**日期**：[今天]

### 验收标准：[X/Y 通过]
- [x] [标准 1] — 自动验证（测试通过）
- [x] [标准 2] — 已确认
- [ ] [标准 3] — 失败：[原因]
- [?] [标准 4] — 延迟：需要试玩

### 测试与标准可追溯性
| 标准 | 测试 | 状态 |
|-----------|------|--------|
| AC-1：[文本] | [测试文件::测试名称] | COVERED |
| AC-2：[文本] | 手动确认 | COVERED |
| AC-3：[文本] | — | UNTESTED |

### 测试证据
**Story 类型**：[Logic | Integration | Visual/Feel | UI | Config/Data | 未声明]
**所需证据**：[单元测试文件 | 集成测试或试玩 | 截图 + 签字确认 | 演练文档 | 冒烟检查通过]
**证据发现**：[是 — `[路径]` | 否 — BLOCKING | 否 — ADVISORY]

### 偏差
[无] 或者：
- BLOCKING：[描述] — [GDD/ADR 引用]
- ADVISORY：[描述] — 用户已接受 / 标记为技术债务

### 范围
[所有更改均在声明范围内] 或者：
- 触及的额外文件：[列表] — [注明是有效的还是范围蔓延]

### 判定：COMPLETE / COMPLETE WITH NOTES / BLOCKED
```

**判定定义：**
- **COMPLETE**：所有标准通过，无阻塞性偏差
- **COMPLETE WITH NOTES**：所有标准通过，记录了建议性偏差
- **BLOCKED**：必须首先解决失败的标准或阻塞性偏差

如果判定为 **BLOCKED**：不继续到阶段 7。列出必须修复的内容。主动提供帮助修复阻塞项。

---

## 阶段 7：更新 Story 状态

在写入前询问："我可以更新 Story 文件以标记为完成并记录完成说明吗？"

如果同意，编辑 Story 文件：

1. 更新状态字段：`Status: Complete`
2. 在底部添加 `## Completion Notes` 章节：

```markdown
## Completion Notes
**完成日期**：[日期]
**标准**：[X/Y 通过]（[列出的任何延迟项]）
**偏差**：[无] 或 [建议性偏差列表]
**测试证据**：[Logic：路径下的测试文件 | Visual/Feel：路径下的证据文档 | Config/Data：无需]
**代码审查**：[待处理 / 已完成 / 已跳过]
```

3. 如果存在建议性偏差，询问："我是否需要将这些记录为 `docs/tech-debt-register.md` 中的技术债务？"

4. **更新 `production/sprint-status.yaml`**（如果存在）：
   - 找到与此 Story 文件路径或 ID 匹配的条目
   - 设置 `status: done` 和 `completed: [今天的日期]`
   - 更新顶层的 `updated` 字段
   - 这是静默更新——无需额外批准（已在上述步骤中批准）

### 会话状态更新

在更新 Story 文件后，静默追加到 `production/session-state/active.md`：

```
## 会话摘录 — /story-done [日期]
- 判定：[COMPLETE / COMPLETE WITH NOTES / BLOCKED]
- Story：[Story 文件路径] — [Story 标题]
- 记录的技术债务：[N 项，或"无"]
- 下一步推荐：[下一个就绪的 Story 标题和路径，或"未识别"]
```

如果 `active.md` 不存在，使用此块作为初始内容创建它。在对话中确认："会话状态已更新。"

---

## 阶段 8：展示下一个 Story

完成后，帮助开发者保持势头：

1. 从 `production/sprints/` 读取当前 Sprint 计划。
2. 查找满足以下条件的 Story：
   - 状态为 READY 或 NOT STARTED
   - 未被其他未完成的 Story 阻塞
   - 属于必须拥有（Must Have）或应该拥有（Should Have）层级

展示：

```
### 下一个任务
以下 Story 已就绪可供领取：
1. [Story 名称] — [一行描述] — 预估：[X 小时]
2. [Story 名称] — [一行描述] — 预估：[X 小时]

在开始之前运行 `/story-readiness [路径]` 以确认 Story 已准备好实现。
```

如果此 Sprint 中没有剩余的 Must Have Story（全部已完成或已阻塞）：

```
### Sprint 收尾序列

所有 Must Have Story 已完成。在推进之前需要 QA 签字确认。按顺序运行以下步骤：

1. `/smoke-check sprint` — 验证关键路径端到端仍然可用
2. `/qa-plan` — 完整 QA 周期：测试用例执行、Bug 分类、签字确认报告
3. `/gate-check` — 一旦 QA 批准，推进到下一阶段

在 `/qa-plan` 返回 APPROVED 或 APPROVED WITH CONDITIONS 之前，不要运行 `/gate-check`。
```

如果仍有 Should Have Story 未开始，将它们与收尾序列一起展示，以便用户选择：立即关闭 Sprint，还是先拉入更多工作。

如果没有更多就绪的 Story，但 Must Have Story 仍在进行中（未完成）：
"没有更多可开始的 Story — [N] 个 Must Have Story 仍在进行中。在 Sprint 收尾之前继续实现这些 Story。"

---

## 协作协议

- **未经用户批准，绝不要标记 Story 完成** — 阶段 7 需要明确的"是"才能编辑任何文件。
- **绝不要自动修复失败的标准** — 报告它们并询问该怎么做。
- **偏差是事实，而非判断** — 中立地展示它们；由用户决定是否可接受。
- **BLOCKED 判定是建议性的** — 用户仍可以覆盖并标记为完成；如果他们这样做，请明确记录风险。
- 使用 `AskUserQuestion` 进行代码审查提示以及批量手动标准确认。

---

## 推荐后续步骤

- 运行 `/story-readiness [下一个 Story 路径]` 以在开始实现之前验证下一个 Story
- 如果所有 Must Have Story 都已完成：运行 `/smoke-check sprint` → `/qa-plan` → `/gate-check`
- 如果记录了技术债务：通过跟踪它来保持注册表的最新状态
