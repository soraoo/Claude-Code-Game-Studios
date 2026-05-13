---
name: dev-story
description: "读取 Story 文件并实现它。加载完整上下文（Story、GDD 需求、ADR 指南、控制清单），路由到对应系统和引擎的正确程序员 Agent，实现代码和测试，并确认每个验收标准。核心实现 Skill — 在 /story-readiness 之后、/code-review 和 /story-done 之前运行。"
argument-hint: "[story-path]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Bash, Task, AskUserQuestion
---

# Dev Story

本 Skill 连接了规划与编码。它完整读取一个 Story 文件，汇集程序员所需的所有上下文，
路由到正确的 Specialist Agent，并驱动实现直至完成——包括编写测试。

**每个 Story 的循环：**
```
/qa-plan sprint           ← 在 Sprint 开始前定义测试需求
/story-readiness [path]   ← 在开始前验证
/dev-story [path]         ← 实现它（本 Skill）
/code-review [files]      ← 审查它
/story-done [path]        ← 验证并关闭它
```

**所有 Sprint Story 完成后：** 运行 `/qa-plan` 执行完整的 QA 流程，在推进项目阶段前获得签核意见。

**输出：** 项目 `src/` 和 `tests/` 目录中的源代码 + 测试文件。

---

## 阶段 1：查找 Story

**如果提供了路径**：直接读取该文件。

**如果没有参数**：检查 `production/session-state/active.md` 中的当前活动
Story。如果找到，确认："正在继续处理 [story title] —— 是否正确？"
如果未找到，询问："我们要实现哪个 Story？" 使用 Glob
搜索 `production/epics/**/*.md` 并列出状态为 Ready 的 Story。

---

## 阶段 2：加载完整上下文

**在加载任何上下文之前，验证必需文件是否存在。** 从 Story 的 `ADR Governing Implementation` 字段中提取 ADR 路径，然后检查：

| 文件 | 路径 | 如果缺失 |
|------|------|------------|
| TR registry | `docs/architecture/tr-registry.yaml` | **停止** —— "未找到 TR registry。请运行 `/create-epics` 生成它。" |
| 管辖 ADR | Story 的 ADR 字段中的路径 | **停止** —— "未找到 ADR 文件 [path]。请在 docs/architecture/ 中创建 ADR，或修正 Story 的 ADR 字段中的文件名。" |
| Control manifest | `docs/architecture/control-manifest.md` | **警告并继续** —— "未找到控制清单 —— 无法检查层级规则。请运行。" |

如果 TR registry 或管辖 ADR 缺失，在会话状态中将 Story 状态设置为 **BLOCKED**，且不生成任何程序员 Agent。

同时读取以下所有内容 —— 这些是独立的读取操作。在所有上下文加载完成之前，不要开始实现：

### Story 文件
提取并保留：
- **Story 标题、ID、层级、类型**（Logic / Integration / Visual/Feel / UI / Config/Data）
- **TR-ID** —— GDD 需求标识符
- **管辖 ADR** 引用
- **Manifest 版本** —— 嵌入在 Story 头部
- **验收标准** —— 每个复选框项，逐字保留
- **实现说明** —— Story 中的 ADR 指南部分
- **范围外** 边界
- **测试证据** —— 必需的测试文件路径
- **依赖项** —— 在此 Story 之前必须完成的内容

### TR registry
读取 `docs/architecture/tr-registry.yaml`。查找该 Story 的 TR-ID。
读取当前 `requirement` 文本 —— 这是 GDD 当前需求的
真实来源。不要依赖 Story 文件中的任何内联文本（可能已过时）。

### 管辖 ADR
读取 `docs/architecture/[adr-file].md`。提取：
- 完整的决策（Decision）部分
- 实现指南（Implementation Guidelines）部分（这是程序员要遵循的）
- 引擎兼容性（Engine Compatibility）部分（截止日期后的 API、已知风险）
- ADR 依赖项（ADR Dependencies）部分

### 控制清单
读取 `docs/architecture/control-manifest.md`。提取适用于此 Story 层级的规则：
- 必需模式
- 禁止模式
- 性能护栏

检查：Story 中嵌入的 Manifest 版本是否与当前清单头部日期匹配？
如果不匹配，在继续之前使用 `AskUserQuestion`：
- 提示："Story 是基于 manifest v[story-date] 编写的。当前 manifest 是 v[current-date]。可能有新规则适用。你希望如何处理？"
- 选项：
  - `[A] 更新 Story manifest 版本并用当前规则实现（推荐）`
  - `[B] 使用旧规则实现 —— 我接受不符合规范的风险`
  - `[C] 在此停止 —— 我想先审查 manifest 差异`

如果选择 [A]：在生成程序员之前，将 Story 文件的 `Manifest Version:` 字段编辑为当前 manifest 日期。然后仔细阅读 manifest 中的新规则。
如果选择 [B]：无论如何也要仔细阅读 manifest 中的新规则，并在阶段 6 总结的"偏差"部分记录版本不匹配。
如果选择 [C]：停止。不生成任何 Agent。让用户审查并重新运行 `/dev-story`。

### 依赖项验证

从 Story 文件中提取 **依赖项** 列表后，验证每一项：

1. 使用 Glob 搜索 `production/epics/**/*.md` 查找每个依赖项 Story 文件。
2. 读取其 `Status:` 字段。
3. 如果任何依赖项的状态不是 `Complete` 或 `Done`：
   - 使用 `AskUserQuestion`：
     - 提示："Story '[current story]' 依赖于 '[dependency title]'，该 Story 当前状态为 [status]，而非 Complete。你希望如何处理？"
     - 选项：
       - `[A] 仍然继续 —— 我接受依赖项风险`
       - `[B] 停止 —— 我先完成依赖项`
       - `[C] 依赖项已完成但状态未更新 —— 将其标记为 Complete 并继续`
   - 如果选择 [B]：将会话状态中的 Story 状态设置为 **BLOCKED** 并停止。不生成任何程序员 Agent。
   - 如果选择 [C]：在继续之前询问"我可以将 [dependency path] 的 Status 更新为 Complete 吗？"
   - 如果选择 [A]：在阶段 6 总结的"偏差"部分记录："在依赖项未完成的情况下实现：[dependency title] —— [status]。"

如果找不到依赖项文件：警告"未找到依赖项 Story：[path]。请验证路径或创建 Story 文件。"

---

### 引擎参考
读取 `.claude/docs/technical-preferences.md`：
- `Engine:` 值 —— 决定使用哪些程序员 Agent
- 命名规范（类名、文件名、信号/事件名）
- 性能预算（帧预算、内存上限）
- 禁止模式

---

## 阶段 3：路由到正确的程序员

根据 Story 的**层级**、**类型**和**系统名称**，决定通过 Task
生成哪个 Specialist。

**Config/Data Story —— 完全跳过 Agent 生成：**
如果 Story 的类型是 `Config/Data`，则不需要程序员 Agent 或引擎 Specialist。直接跳转到阶段 4（Config/Data 说明）。实现方式是编辑数据文件 —— 无需路由表评估，无需引擎 Specialist。

### 主 Agent 路由表

| Story 上下文 | 主 Agent |
|---|---|
| Foundation 层级 —— 任意类型 | `engine-programmer` |
| 任意层级 —— 类型：UI | `ui-programmer` |
| 任意层级 —— 类型：Visual/Feel | `gameplay-programmer`（实现） |
| Core 或 Feature —— 游戏玩法机制 | `gameplay-programmer` |
| Config/Data —— 无需代码 | 无需 Agent（参见阶段 4 Config 说明） |

### 引擎 Specialist —— 编码类 Story 始终作为辅助生成

读取 `.claude/docs/technical-preferences.md` 中的 `Engine Specialists` 部分
以获取配置的主 Specialist。当 Story 涉及引擎特定的 API、模式或 ADR 具有 HIGH
引擎风险时，将其与主 Agent 一起生成。

| 引擎 | 可用的 Specialist Agent |
|--------|----------------------------|

**当引擎风险为 HIGH 时**（来自 ADR 或 VERSION.md）：即使是非引擎面向的 Story，也要始终生成引擎
Specialist。高风险意味着 ADR 记录了关于截止日期后引擎 API 的假设，需要专家验证。

---

## 阶段 4：实现

通过 Task 生成所选程序员 Agent，携带完整的上下文包：

向 Agent 提供：
1. 完整的 Story 文件内容
2. 当前的 GDD 需求文本（来自 TR registry）
3. ADR 决策 + 实现指南（逐字提供 —— 不要概括）
4. 适用于此层级的控制清单规则
5. 引擎命名规范和性能预算
6. ADR 引擎兼容性部分的任何引擎特定说明
7. 必须创建的测试文件路径
8. 明确指示：**实现此 Story 并编写测试**

Agent 应：
- 按照 ADR 指南在 `src/` 中创建或修改文件
- 遵循控制清单中所有必需和禁止模式
- 保持在 Story 的"范围外"边界内（不要触碰无关文件）
- 编写干净、带文档注释的公共 API

### Config/Data Story（无需 Agent）

对于类型为 Config/Data 的 Story，不需要程序员 Agent。实现
方式是编辑数据文件。读取 Story 的验收标准并对数据文件进行指定的更改。
记录哪些值被更改以及从/到什么值。

### Visual/Feel Story

生成 `gameplay-programmer` 来实现代码/动画调用。注意
Visual/Feel 的验收标准无法自动验证 —— "感觉对吗？"
检查将在 `/story-done` 中通过手动确认完成。

---

## 阶段 5：编写测试

对于 **Logic** 和 **Integration** Story，测试必须在本次
实现中编写 —— 不可推迟到以后。

提醒程序员 Agent：

> "此 Story 的测试文件必须在以下位置创建：`[Test Evidence 部分的路径]`。
> 如果没有该文件，Story 无法通过 `/story-done` 关闭。请将测试
> 与实现一起编写，而不是之后。"

测试要求（来自 coding-standards.md）：
- 文件名：`[system]_[feature]_test.[ext]`
- 函数名：`test_[scenario]_[expected_outcome]`
- 每个验收标准必须至少有一个覆盖它的测试函数
- 无随机种子、无时间相关断言、无外部 I/O
- 测试 GDD 公式部分中的公式边界

对于 **Visual/Feel** 和 **UI** Story：无需自动化测试。提醒 Agent 在
实现总结中注明需要哪些手动证据：
"证据文档需创建在 `production/qa/evidence/[slug]-evidence.md`。"

对于 **Config/Data** Story：无需测试文件。冒烟检查将作为证据。

---

## 阶段 6：收集与总结

程序员 Agent 完成后，收集：

- 创建或修改的文件（含路径）
- 创建的测试文件（路径和编写的测试函数数量）
- 任何偏离 Story 范围外边界的情况（标记这些）
- Agent 提出的任何问题或阻塞项
- Specialist 标记的任何引擎特定风险

呈现一个简洁的实现总结：

```
## 实现完成：[Story Title]

**更改的文件**：
- `src/[path]` —— 已创建 / 已修改（[简要描述]）
- `tests/[path]` —— 测试文件（[N] 个测试函数）

**已覆盖的验收标准**：
- [x] [criterion] —— 在 [file:function] 中实现
- [x] [criterion] —— 由测试 [test_name] 覆盖
- [ ] [criterion] —— 已延期：需要试玩测试（Visual/Feel）

**范围偏差**：[无] 或 [列出在 Story 边界外触碰的文件]
**标记的引擎风险**：[无] 或 [Specialist 发现]
**阻塞项**：[无] 或 [描述]

准备就绪：`/code-review [file1] [file2]` 然后 `/story-done [story-path]`
```

---

## 阶段 7：更新会话状态

静默追加到 `production/session-state/active.md`：

```
## 会话摘要 —— /dev-story [date]
- Story：[story-path] —— [story title]
- 更改的文件：[逗号分隔的列表]
- 已编写测试：[path，或"无 —— Visual/Feel/Config Story"]
- 阻塞项：[无，或描述]
- 下一步：/code-review [files] 然后 /story-done [story-path]
```

如果 `active.md` 不存在则创建它。确认："会话状态已更新。"

---

## 错误恢复协议

如果任何生成的 Agent（通过 Task）返回 BLOCKED、出错或无法完成：

1. **立即上报**：在进入依赖阶段之前向用户报告"[AgentName]：被阻塞 —— [原因]"
2. **评估依赖关系**：检查被阻塞 Agent 的输出是否为后续阶段所必需。如果是，在没有用户输入的情况下，不要越过该依赖点。
3. **通过 AskUserQuestion 提供选项**：
   - 跳过此 Agent 并在最终报告中记录缺口
   - 以更窄的范围重试
   - 在此停止并先解决阻塞项
4. **始终生成部分报告** —— 输出已完成的内容。切勿因为一个 Agent 被阻塞而丢弃工作。

常见阻塞项：
- 输入文件缺失（未找到 Story、缺少 GDD）→ 重定向到创建它的 Skill
- ADR 状态为 Proposed → 不要实现；先在 docs/architecture/ 中运行 ADR
- 范围过大 → 通过 `/create-stories` 拆分为两个 Story
- ADR 和 Story 之间的指令冲突 → 报告冲突，不要猜测
- Manifest 版本不匹配 → 向用户显示差异，询问是按旧规则继续还是先更新 Story

## 协作协议

- **文件写入是委派的** —— 所有源代码、测试文件和证据文档由通过 Task 生成的子 Agent 编写。每个子 Agent 单独执行"我可以写入 [path] 吗？"协议。此编排器不直接写入文件。
- **先加载再实现** —— 在所有上下文加载完成之前不要开始编码
  （Story、TR-ID、ADR、manifest、引擎偏好）。不完整的上下文会产生
  偏离设计的代码。
- **ADR 是法则** —— 实现必须遵循 ADR 的实现
  指南。如果指南与看似"更好"的做法冲突，在总结中标记
  而非默默偏离。
- **保持在范围内** —— "范围外"部分是一个约定。如果实现
  Story 需要触碰范围外的文件，停下来上报：
  "实现 [criterion] 需要修改 [file]，这超出了范围。
  我是否继续，还是创建一个单独的 Story？"
- **Logic/Integration 的测试不可省略** —— 在测试文件存在之前，
  不要标记实现完成
- **Visual/Feel 标准是延期而非跳过** —— 在总结中将其标记为 DEFERRED
  （已延期）；它们将在 `/story-done` 中手动验证
- **在做出大型结构性决策前先询问** —— 如果 Story 需要
  ADR 未涵盖的架构模式，在实现前上报：
  "ADR 没有说明如何处理 [case]。我的计划是 [X]。是否继续？"

---

## 推荐后续步骤

- 运行 `/code-review [file1] [file2]` 在关闭 Story 之前审查实现
- 运行 `/story-done [story-path]` 验证验收标准并将 Story 标记为完成
- 所有 Sprint Story 完成后：在推进项目阶段之前运行 `/qa-plan` 执行完整 QA 流程
