---
name: help
description: "分析已完成的工作和用户查询，提供下一步该做什么的建议。当用户说'接下来该做什么'或'我现在做什么'或'我卡住了'或'我不知道该做什么'时使用。"
argument-hint: "[optional: what you just finished, e.g. 'finished design-review' or 'stuck on ADRs']"
user-invocable: true
allowed-tools: Read, Glob, Grep
context: |
  !echo "=== Live Project State ===" && echo "Stage: $(cat production/stage.txt 2>/dev/null | tr -d '[:space:]' || echo 'not set')" && echo "Latest sprint: $(ls -t production/sprints/*.md 2>/dev/null | head -1 || echo 'none')" && echo "Session state: $(head -5 production/session-state/active.md 2>/dev/null || echo 'none')"
model: haiku
---

# 工作室帮助——我下一步该做什么？

此 Skill 是只读的——它报告发现但不写入任何文件。

此 Skill 精确判断你在游戏开发管线中的位置，并告诉你接下来该做什么。它是**轻量级**的——不是完整的审计。如需完整的差距分析，使用 `/project-stage-detect`。

---

## 步骤 1：读取目录

读取 `.claude/docs/workflow-catalog.yaml`。这是所有阶段、它们的步骤（按顺序）、每个步骤是必需还是可选以及指示完成的产物 glob 的权威列表。

---

## 步骤 1b：查找不在目录中的 Skill

读取目录后，Glob `.claude/skills/*/SKILL.md` 获取已安装 Skill 的完整列表。对每个文件，从其 frontmatter 中提取 `name:` 字段。

与目录中的 `command:` 值进行比较。任何名称未作为目录命令出现的 Skill 都是**未编目 Skill**——仍可使用但不是阶段关卡工作流的一部分。

收集这些用于步骤 7 的输出——将它们作为页脚块展示：

```
### 同时已安装（不在工作流中）
- `/skill-name`——[来自 SKILL.md frontmatter 的描述]
- `/skill-name`——[描述]
```

只有在至少存在一个未编目 Skill 时才显示此块。根据用户当前阶段限制最多 10 个最相关的（生产阶段的 QA Skill、生产/打磨阶段的团队 Skill 等）。

---

## 步骤 2：确定当前阶段

按此顺序检查：

1. **读取 `production/stage.txt`**——如果存在且有内容，这是权威的阶段名称。将其映射到目录阶段键：
   - "Concept" → `concept`
   - "Systems Design" → `systems-design`
   - "Technical Setup" → `technical-setup`
   - "Pre-Production" → `pre-production`
   - "Production" → `production`
   - "Polish" → `polish`
   - "Release" → `release`

2. **如果 stage.txt 缺失**，从产物推断阶段（最高级匹配胜出）：
   - `src/` 有 10+ 个源文件 → `production`
   - `production/stories/*.md` 存在 → `pre-production`
   - `docs/architecture/adr-*.md` 存在 → `technical-setup`
   - `design/gdd/systems-index.md` 存在 → `systems-design`
   - `design/gdd/game-concept.md` 存在 → `concept`
   - 什么都没有 → `concept`（新项目）

---

## 步骤 3：读取会话上下文

如果存在，读取 `production/session-state/active.md`。提取：
- 最近在处理什么
- 任何进行中的任务或待解决的问题
- 来自 STATUS 块（如果存在）的当前 Epic/功能/任务

这告诉你用户刚刚完成了什么或卡在哪里——用它来个性化输出。

---

## 步骤 4：检查当前阶段的步骤完成情况

对当前阶段的每个步骤（来自目录）：

### 基于产物的检查

如果步骤有 `artifact.glob`：
- 使用 Glob 检查匹配模式的文件是否存在
- 如果指定了 `min_count`，验证至少匹配那么多文件
- 如果指定了 `artifact.pattern`，使用 Grep 验证模式存在于匹配的文件中
- **完成** = 产物条件满足
- **未完成** = 产物缺失或未找到模式

如果步骤有 `artifact.note`（无 glob）：
- 标记为 **手动**——无法自动检测，将询问用户

如果步骤没有 `artifact` 字段：
- 标记为 **未知**——完成情况不可跟踪（例如可重复的实现工作）

### 特殊情况：生产阶段——读取 `sprint-status.yaml`

当当前阶段是 `production` 时，在进行任何基于 glob 的 Story 检查之前检查 `production/sprint-status.yaml`。如果存在，直接读取：

- `status: in-progress` 的 Story → 显示为"当前活跃"
- `status: ready-for-dev` 的 Story → 显示为"下一个要做的"
- `status: done` 的 Story → 计入完成
- `status: blocked` 的 Story → 显示为阻碍项，附带 `blocker` 字段

这提供了精确的逐个 Story 状态，无需 Markdown 扫描。跳过 `implement` 和 `story-done` 步骤的 glob 产物检查——YAML 是权威的。

### 特殊情况：`repeatable: true`（非生产阶段）

对于生产阶段之外的可重复步骤（例如"系统 GDD"），产物检查告诉你是否有任何工作已完成，而非是否已全部完成。用不同的方式标记这些——显示已检测到的内容，然后注明可能仍在进行中。

---

## 步骤 5：找到位置并识别后续步骤

从完成数据中确定：

1. **最后确认完成的步骤**——最远的已完成的必需步骤
2. **当前阻碍项**——第一个未完成的*必需*步骤（这是用户接下来必须做的）
3. **可选机会**——未完成的*可选*步骤，可以在阻碍项之前或同时进行
4. **即将到来的必需步骤**——当前阻碍项之后的必需步骤（显示为"即将到来"以便用户提前规划）

如果用户提供了参数（例如"刚完成设计审查"），使用它来推进他们提到的步骤，即使产物检查不明确。

---

## 步骤 6：检查进行中的工作

如果 `active.md` 显示活跃任务或 Epic：
- 在顶部突出显示："看起来你正在处理 [X]"
- 建议继续或确认是否已完成

---

## 步骤 7：呈现输出

保持**简短直接**。这是快速定位，而非报告。

```
## 你所在的位置：[阶段标签]

**进行中：** [来自 active.md，如果有的话]

### ✓ 已完成
- [已完成的步骤名称]
- [已完成的步骤名称]

### → 接下来（必需）
**[步骤名称]**——[描述]
命令：`[/command]`

### ~ 也可用（可选）
- **[步骤名称]**——[描述] → `/command`
- **[步骤名称]**——[描述] → `/command`

### 之后即将到来
- [下一个必需步骤名称] (`/command`)
- [下一个必需步骤名称] (`/command`)

---
即将接近 **[当前阶段] → [下一阶段]** 关卡 → 准备好时运行 `/gate-check`。
```

**格式化规则：**
- `✓` 表示已确认完成
- `→` 表示当前必需的下一步（只有一个——第一个阻碍项）
- `~` 表示当前可用的可选步骤
- 命令以内联反引号显示
- 如果步骤没有命令（例如"实现 Story"），解释该做什么而不是显示斜杠命令
- 对于手动步骤，询问用户："我无法判断 [步骤] 是否完成——它已经完成了吗？"

判定：**COMPLETE**——已识别后续步骤。

---

## 步骤 8：关卡警告（如果接近）

在当前阶段的步骤之后，检查用户是否可能接近关卡：
- 如果当前阶段的所有必需步骤都已完成（或接近完成），添加："你已接近 **[当前] → [下一]** 关卡。准备好时运行 `/gate-check`。"
- 如果仍有多个必需步骤未完成，跳过关卡警告——还不相关。

---

## 步骤 9：升级路径

在推荐之后，如果用户似乎卡住或困惑，添加：

```
---
需要更多详细信息？
- `/project-stage-detect`——完整的差距分析，列出所有缺失的产物
- `/gate-check`——下一阶段的正式就绪检查
- `/start`——从头重新定位
```

仅在用户的输入暗示困惑时显示（例如"我不知道"、"卡住了"、"迷失"、"不确定"）。不要对简单的"接下来是什么？"查询显示。

---

## 协作协议

- **绝不自动运行下一个 Skill。** 推荐它，让用户自己调用。
- **询问手动步骤**而不是假设完成或未完成。
- **匹配用户的语气**——如果他们听起来有压力（"我完全迷失了"），要令人安心，给出一个行动，而非六个的列表。
- **一个主要推荐**——用户离开时应该确切知道接下来要做的一件事。可选步骤和"即将到来"是次要上下文。
