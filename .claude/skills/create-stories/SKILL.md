---
name: create-stories
description: "将单个 Epic 分解为可实现的 Story 文件。读取 Epic、其 GDD、管辖的 ADR 和控制清单。每个 Story 嵌入其 GDD 需求 TR-ID、ADR 指南、验收标准、Story 类型和测试证据路径。在每个 Epic 的 /create-epics 之后运行。"
argument-hint: "[epic-slug | epic-path] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Task, AskUserQuestion
agent: lead-programmer
---

# 创建 Story

Story 是单个可实现的行为——足够小以在一次专注会话中完成，自包含，且完全可追溯到 GDD 需求和 ADR 决策。Story 是开发者可开始工作的单元。Epic 是架构师定义的单元。

**对每个 Epic 运行此 Skill**，而非每个层级。首先为基础层 Epic 运行，然后是核心层，依此类推——匹配依赖顺序。

**输出：** `production/epics/[epic-slug]/story-NNN-[slug].md` 文件

**上一步：** `/create-epics [system]`
**Story 存在后的下一步：** `/story-readiness [story-path]` 然后 `/dev-story [story-path]`

---

## 1. 解析参数

提取 `--review [full|lean|solo]`（如果存在）并存储为本次运行的审查模式覆盖。如果未提供，读取 `production/review-mode.txt`（如果缺失默认 `full`）。此解析的模式应用于此 Skill 中的所有关卡派生——在每次关卡调用之前应用 `(director gates disabled in indie mode)` 中的检查模式。

- `/create-stories [epic-slug]`——例如 `/create-stories combat`
- `/create-stories production/epics/combat/EPIC.md`——也接受完整路径
- 无参数——询问："你想将哪个 Epic 分解为 Story？" Glob `production/epics/*/EPIC.md` 并列出可用 Epic 及其状态。

---

## 2. 为此 Epic 加载一切

完整阅读：

- `production/epics/[epic-slug]/EPIC.md`——Epic 概述、管辖的 ADR、GDD 需求表
- Epic 的 GDD（`design/gdd/[文件名].md`）——阅读全部 8 个章节，特别是验收标准、公式和边缘情况
- Epic 中列出的所有管辖 ADR——阅读决策、实现指南、引擎兼容性和引擎备注章节
- `docs/architecture/control-manifest.md`——提取此 Epic 层级的规则；记录头部的清单版本日期
- `docs/architecture/tr-registry.yaml`——加载此系统的所有 TR-ID

**ADR 存在验证**：从 Epic 读取管辖 ADR 列表后，确认每个 ADR 文件在磁盘上存在。如果任何 ADR 文件找不到，在分解任何 Story 之前**立即停止**：

> "Epic 引用了 [ADR-NNNN: title] 但 `docs/architecture/[adr-file].md` 未找到。检查 Epic 的管辖 ADR 列表中的文件名，或在 docs/architecture/ 中创建 ADR。在所有引用的 ADR 文件都确认存在之前无法创建 Story。"

在确认所有引用的 ADR 文件存在之前不要进入步骤 3。

报告："已加载 Epic [名称]，GDD [文件名]，[N] 个管辖 ADR（全部确认存在），控制清单 v[日期]。"

---

## 3. 按类型分类 Story

**Story 类型分类**——根据其验收标准为每个 Story 分配类型：

| Story 类型 | 当标准引用以下内容时分配... |
|---|---|
| **Logic** | 公式、数值阈值、状态转换、AI 决策、计算 |
| **Integration** | 两个或更多系统交互、信号跨越边界、存档/读档往返 |
| **Visual/Feel** | 动画行为、VFX、"感觉响应灵敏"、时机、屏幕震动、音频同步 |
| **UI** | 菜单、HUD 元素、按钮、界面、对话框、工具提示 |
| **Config/Data** | 仅平衡调优数值、数据文件更改——无新代码逻辑 |

混合 Story：分配实现风险最高的类型。类型决定在 `/story-done` 可以关闭 Story 之前需要什么测试证据。

---

## 4. 将 GDD 分解为 Story

对每个 GDD 验收标准：

1. 将需要相同核心实现的相关标准分组
2. 每组 = 一个 Story
3. 排序：基础行为优先、边缘情况最后、UI 最后

**Story 大小规则：** 一个 Story = 一次专注会话（约 2-4 小时）。如果一组标准需要更长时间，拆分为两个 Story。

对每个 Story，确定：
- **GDD 需求**：这满足哪个验收标准？
- **TR-ID**：在 `tr-registry.yaml` 中查找。使用稳定的 ID。如果不匹配，使用 `TR-[系统]-???` 并警告。
- **管辖 ADR**：哪个 ADR 管辖如何实现它？
  - `状态：已接受` → 正常嵌入
  - `状态：建议` → 设置 Story `Status: Blocked` 并注明："BLOCKED：ADR-NNNN 是建议状态——在 docs/architecture/ 中创建 ADR 以推进它"
- **Story 类型**：来自步骤 3 的分类
- **引擎风险**：来自 ADR 的知识风险字段

---

## 4b. QA 主管 Story 就绪关卡

**审查模式检查**——在派生 QL-STORY-READY 之前应用：
- `solo` → 跳过。注明："QL-STORY-READY 已跳过——单人模式。"进入步骤 5（展示 Story 供审查）。
- `lean` → 跳过（非 PHASE-GATE）。注明："QL-STORY-READY 已跳过——精简模式。"进入步骤 5（展示 Story 供审查）。
- `full` → 正常派生。

在分解所有 Story 后（步骤 4 完成）但在展示写入批准之前，审查测试覆盖和就绪状态。为用户标记空白。

传递：完整的 Story 列表，包含验收标准、Story 类型和 TR-ID；Epic 的 GDD 验收标准供参考。

展示 QA 主管的评估。对于被标记为 GAPS 或 INADEQUATE 的每个 Story，在继续之前修改验收标准——具有不可测试标准的 Story 无法正确实现。一旦所有 Story 达到 ADEQUATE，继续。

**ADEQUATE 之后**：对每个 Logic 和 Integration Story，要求 qa-tester 生成具体的测试用例规格——每个验收标准一个——按此格式：

```
Test: [标准文本]
  Given: [前置条件]
  When: [动作]
  Then: [预期结果 / 断言]
  Edge cases: [要测试的边界值或失败状态]
```

对 Visual/Feel 和 UI Story，生成手动验证步骤：
```
Manual check: [标准文本]
  Setup: [如何达到该状态]
  Verify: [要查看什么]
  Pass condition: [无歧义的通过描述]
```

这些测试用例规格直接嵌入每个 Story 的 `## QA 测试用例` 章节。开发者按照这些用例实现。程序员不是从头编写测试——QA 已经定义了"完成"的样子。

---

## 5. 展示 Story 供审查

在写入任何文件之前，展示完整的 Story 列表：

```
## Epic：[名称] 的 Story

Story 001：[标题]——Logic——ADR-NNNN
  覆盖：TR-[系统]-001（[1 行需求摘要]）
  所需测试：tests/unit/[系统]/[slug]_test.[扩展名]

Story 002：[标题]——Integration——ADR-MMMM
  覆盖：TR-[系统]-002, TR-[系统]-003
  所需测试：tests/integration/[系统]/[slug]_test.[扩展名]

```

使用 `AskUserQuestion` 获取批准后写入 Story 文件。

---

## 6. 写入 Story 文件 + 更新 EPIC.md

对每个 Story 写入 `production/epics/[epic-slug]/story-[NNN]-[slug].md`，包含：上下文、验收标准、实现笔记、范围外、QA 测试用例、测试证据、依赖关系等字段。同时更新 Epic 的 EPIC.md 中的 Story 表。

---

## 7. 写入后

使用 `AskUserQuestion` 以上下文感知的后续步骤结束。检查是否还有其他没有 Story 的 Epic，提供相应选项。

---

## 协作协议

1. **展示前阅读**——在展示 Story 列表之前静默加载所有输入
2. **一次询问**——在摘要中一次展示 Epic 的所有 Story，而非逐个
3. **对被阻塞的 Story 发出警告**——在写入前标记任何具有建议 ADR 的 Story
4. **写入前询问**——写入文件前获取全部 Story 集的批准
5. **不凭空创造**——验收标准来自 GDD，实现笔记来自 ADR，规则来自清单
6. **绝不开始实现**——此 Skill 停在 Story 文件级别

写入后（或拒绝后）：

- **判定：COMPLETE**——[N] 个 Story 已写入到 `production/epics/[epic-slug]/`。运行 `/dev-story` → `/story-readiness`。
- **判定：BLOCKED**——用户拒绝。未写入 Story 文件。
