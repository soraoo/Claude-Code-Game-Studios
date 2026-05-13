---
name: smoke-check
description: "在 QA 交接前运行关键路径冒烟测试门禁。执行自动化测试套件，验证核心功能，并生成 PASS/FAIL 报告。在 Sprint 的故事实现完成后、手动 QA 开始前运行。冒烟检查失败意味着该版本尚未准备好交给 QA。"
argument-hint: "[sprint | quick | --platform pc|console|mobile|all]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Bash, Write, AskUserQuestion
---

# Smoke Check（冒烟检查）

该 Skill 是"实现完成"与"准备 QA 交接"之间的门禁。它运行自动化测试套件，检查测试覆盖缺口，与开发者批量验证关键路径，并生成 PASS/FAIL 报告。

规则很简单：**冒烟检查失败的版本不能交给 QA。** 将损坏的版本交给 QA 会浪费他们的时间并使团队士气低落。

**输出：** `production/qa/smoke-[date].md`

---

## 参数解析

参数可以组合使用：`/smoke-check sprint --platform console`

**基础模式**（第一个参数，默认为 `sprint`）：
- `sprint` —— 针对当前 Sprint 的故事进行完整冒烟检查
- `quick` —— 跳过覆盖扫描（阶段 3）和批次 3；用于快速重新检查

**平台标志**（`--platform`，默认：无）：
- `--platform pc` —— 添加 PC 专用检查（键盘、鼠标、窗口模式）
- `--platform console` —— 添加主机专用检查（手柄、TV 安全区、平台认证要求）
- `--platform mobile` —— 添加移动端专用检查（触控、竖屏/横屏、电池/发热行为）
- `--platform all` —— 添加所有平台变体；输出各平台判定表

如果提供了 `--platform`，阶段 4 会添加平台专用批次，阶段 5 会在总体判定之外输出各平台判定表。

---

## 阶段 1：检测测试环境

在执行任何操作之前，先了解环境：

1. **测试框架检查**：确认 `tests/` 目录是否存在。
   如果不存在："在 `tests/` 未找到测试目录。请运行  来搭建测试基础设施，或者如果测试位于其他位置，请手动创建该目录。" 然后停止。

2. **CI 检查**：检查 `.github/workflows/` 是否包含引用测试的工作流文件。在报告中注明 CI 是否已配置。

3. **引擎检测**：读取 `.claude/docs/technical-preferences.md` 并提取 `Engine:` 的值。保存该值供阶段 2 选择测试命令使用。

4. **冒烟测试列表**：检查 `production/qa/smoke-tests.md` 或 `tests/smoke/` 是否存在。如果找到冒烟测试列表，加载它供阶段 4 使用。如果两者都不存在，则从当前 QA 计划中提取冒烟测试（阶段 4 回退方案）。

5. **QA 计划检查**：使用 glob 匹配 `production/qa/qa-plan-*.md` 并取最近修改的文件。如果找到，记录路径——它将在阶段 3 和阶段 4 中使用。如果未找到，记录："未找到 QA 计划。为获得最佳效果，请在运行冒烟检查前先运行 `/qa-plan sprint`。"

在继续之前报告发现结果："环境：[引擎]。测试目录：[已找到 / 未找到]。CI 已配置：[是 / 否]。QA 计划：[路径 / 未找到]。"

---

## 阶段 2：运行自动化测试

通过 Bash 尝试运行测试套件。根据阶段 1 检测到的引擎选择命令：

**Godot 4：**
```bash
godot --headless --script tests/gdunit4_runner.gd 2>&1
```
如果 GDUnit4 运行脚本在该路径不存在，尝试：
```bash
godot --headless -s addons/gdunit4/GdUnitRunner.gd 2>&1
```
如果两个路径都不存在，记录："未找到 GDUnit4 运行器——请确认您的测试框架的运行器路径。"

**Unity：**
Unity 测试需要编辑器，在大多数环境中无法通过 shell 无头运行。检查最近的测试结果产物：
```bash
ls -t test-results/ 2>/dev/null | head -5
```
如果测试结果文件存在（XML 或 JSON），读取最新文件并解析 PASS/FAIL 计数。如果没有产物："Unity 测试必须从编辑器或 CI 流水线运行。请在继续之前手动确认测试状态。"

**Unreal Engine：**
```bash
ls -t Saved/Logs/ 2>/dev/null | grep -i "test\|automation" | head -5
```
如果未找到匹配的日志："UE 自动化测试必须通过 Session Frontend 或 CI 流水线运行。请手动确认测试状态。"

**未知引擎 / 未配置：**
"引擎未在 `.claude/docs/technical-preferences.md` 中配置。请运行 `/setup-engine` 指定引擎，然后重新运行 `/smoke-check`。"

**如果测试运行器在当前环境中不可用**（引擎二进制文件不在 PATH 上、运行器脚本未找到等），请清晰报告：

"无法执行自动化测试——引擎二进制文件未在 PATH 上找到。状态将记录为 NOT RUN。请从本地 IDE 或 CI 流水线确认测试结果。未确认的 NOT RUN 视为 PASS WITH WARNINGS，而非 FAIL——开发者必须手动确认结果。"

不要将 NOT RUN 视为自动 FAIL。将其记录为警告。开发者在阶段 4 中的手动确认可以解决此问题。

解析运行器输出并提取：
- 运行的总测试数
- 通过数
- 失败数
- 任何失败测试的名称（最多 10 个；如果更多，注明数量）
- 运行器本身的任何崩溃或错误输出

---

## 阶段 3：检查测试覆盖

按优先级顺序从以下来源提取故事列表：
1. 阶段 1 中找到的 QA 计划（其测试摘要表列出了每个故事的预期测试文件路径）
2. 来自 `production/sprints/` 的当前 Sprint 计划（最近修改的文件）
3. 如果传入了 `quick` 参数，则完全跳过此阶段并注明："覆盖扫描已跳过——运行 `/smoke-check sprint` 进行完整的覆盖分析。"

对于范围内的每个故事：

1. 从故事的文件路径中提取系统段
   （例如，`production/epics/combat/story-001.md` → `combat`）
2. 使用 glob 在 `tests/unit/[system]/` 和 `tests/integration/[system]/` 中查找文件名包含故事段或密切相关术语的文件
3. 检查故事文件本身是否有 `Test file:` 头部字段或"Test Evidence"章节

为每个故事分配覆盖状态：

| 状态 | 含义 |
|--------|---------|
| **COVERED** | 找到与该故事系统和范围匹配的测试文件 |
| **MANUAL** | 故事类型为 Visual/Feel 或 UI；找到了测试证据文档 |
| **MISSING** | Logic 或 Integration 故事没有匹配的测试文件 |
| **EXPECTED** | Config/Data 故事——无需测试文件；抽查即可 |
| **UNKNOWN** | 故事文件缺失或无法读取 |

MISSING 条目是建议性缺口。它们不会导致 FAIL 判定，但必须在报告中显著显示，并且必须在 `/story-done` 可以完全关闭这些故事之前解决。

---

## 阶段 4：运行手动冒烟检查

按优先级顺序从以下来源提取冒烟测试清单：
1. QA 计划的"冒烟测试范围"章节（如果在阶段 1 找到了 QA 计划）
2. `production/qa/smoke-tests.md`（如果存在）
3. `tests/smoke/` 目录内容（如果存在）
4. 下面的标准回退列表（仅在以上都不存在时使用）

将批次 2 和批次 3 调整为从 sprint 或 QA 计划中识别的实际系统。将方括号中的占位符替换为当前 Sprint 故事中的真实机制名称。

使用 `AskUserQuestion` 进行批量验证。最多保持 3 次调用。

**批次 1 —— 核心稳定性（始终运行）：**
```
question: "冒烟检查 —— 批次 1：核心稳定性。请验证以下各项："
options:
  - "游戏启动到主菜单无崩溃 —— PASS"
  - "游戏启动到主菜单无崩溃 —— FAIL"
  - "新游戏 / 新会话成功启动 —— PASS"
  - "新游戏 / 新会话成功启动 —— FAIL"
  - "主菜单响应所有输入 —— PASS"
  - "主菜单响应所有输入 —— FAIL"
```

**批次 2 —— Sprint 机制和回归（始终运行）：**
```
question: "冒烟检查 —— 批次 2：当前 Sprint 的更改和回归检查："
options:
  - "[本 Sprint 的主要机制] —— PASS"
  - "[本 Sprint 的主要机制] —— FAIL：[描述损坏情况]"
  - "[本 Sprint 的第二个显著更改（如有）] —— PASS"
  - "[本 Sprint 的第二个显著更改] —— FAIL"
  - "上一个 Sprint 的功能仍然有效（无回归） —— PASS"
  - "上一个 Sprint 的功能 —— 发现回归：[简要描述]"
```

**批次 3 —— 数据完整性和性能（除非传入了 `quick` 参数，否则运行）：**
```
question: "冒烟检查 —— 批次 3：数据完整性和性能："
options:
  - "存档 / 读档完成无数据丢失 —— PASS"
  - "存档 / 读档 —— FAIL：[描述损坏情况]"
  - "存档 / 读档 —— N/A（存档系统尚未实现）"
  - "未观察到新的帧率下降或卡顿 —— PASS"
  - "发现帧率下降或卡顿 —— FAIL：[位置]"
  - "性能 —— 本次会话未检查"
```

记录每个回答的原文，供阶段 5 报告使用。

**平台批次** *（仅在提供了 `--platform` 参数时运行）*：

**PC 平台**（`--platform pc` 或 `--platform all`）：
```
question: "冒烟检查 —— PC 平台：验证平台特定行为："
options:
  - "键盘控制在所有菜单和游戏过程中正常工作 —— PASS"
  - "键盘控制 —— FAIL：[描述问题]"
  - "鼠标输入和光标可见性在所有状态下正确 —— PASS"
  - "鼠标输入 —— FAIL：[描述问题]"
  - "窗口化和全屏模式切换无误且图形正常 —— PASS"
  - "窗口化/全屏 —— FAIL：[描述问题]"
  - "分辨率更改正确应用 —— PASS"
  - "分辨率更改 —— FAIL：[描述问题]"
```

**主机平台**（`--platform console` 或 `--platform all`）：
```
question: "冒烟检查 —— 主机平台：验证平台特定行为："
options:
  - "手柄输入对所有操作正常工作 —— PASS"
  - "手柄输入 —— FAIL：[描述问题]"
  - "UI 在 TV 安全区边距内正常显示（无文本被裁剪） —— PASS"
  - "TV 安全区 —— FAIL：[描述被裁剪的内容]"
  - "没有向手柄用户显示仅键盘/鼠标的回退方案 —— PASS"
  - "输入提示不一致 —— FAIL：[描述]"
  - "游戏从冷启动正常启动（无先前存档） —— PASS"
  - "冷启动 —— FAIL：[描述问题]"
```

**移动端平台**（`--platform mobile` 或 `--platform all`）：
```
question: "冒烟检查 —— 移动端平台：验证平台特定行为："
options:
  - "触控对所有主要操作正常工作 —— PASS"
  - "触控控制 —— FAIL：[描述问题]"
  - "游戏正确处理屏幕方向变化（竖屏 ↔ 横屏） —— PASS"
  - "方向变化 —— FAIL：[描述损坏情况]"
  - "后台 / 前台切换（Home 键）处理优雅 —— PASS"
  - "后台/前台 —— FAIL：[描述问题]"
  - "在目标设备上无明显性能问题（无过热降频迹象） —— PASS"
  - "移动端性能 —— FAIL：[描述问题]"
```

---

## 阶段 5：生成报告

组装完整的冒烟检查报告：

````markdown
## Smoke Check Report（冒烟检查报告）
**日期**：[date]
**Sprint**：[sprint name / number, or "Not identified"]
**引擎**：[engine]
**QA 计划**：[path, or "Not found — run /qa-plan first"]
**参数**：[sprint | quick | blank]

---

### 自动化测试

**状态**：[PASS（[N] 个测试，[N] 个通过）| FAIL（[N] 个失败）|
NOT RUN（[原因]）]

[如果是 FAIL，列出失败的测试：]
- `[test name]` —— [来自运行器输出的简要失败描述]

[如果是 NOT RUN：]
"需要手动确认：测试在您的本地 IDE 或 CI 中是否通过？这将决定自动化测试行是否导致 FAIL 判定。"

---

### 测试覆盖

| 故事 | 类型 | 测试文件 | 覆盖状态 |
|-------|------|-----------|----------------|
| [标题] | Logic | `tests/unit/[system]/[slug]_test.[ext]` | COVERED |
| [标题] | Visual/Feel | `tests/evidence/[slug]-screenshots.md` | MANUAL |
| [标题] | Logic | —— | MISSING ⚠ |
| [标题] | Config/Data | —— | EXPECTED |

**摘要**：[N] 个已覆盖，[N] 个手动，[N] 个缺失，[N] 个预期可忽略。

---

### 手动冒烟检查

- [x] 游戏启动无崩溃 —— PASS
- [x] 新游戏开始 —— PASS
- [x] [核心机制] —— PASS
- [ ] [其他检查] —— FAIL：[用户描述]
- [x] 存档 / 读档 —— PASS
- [-] 性能 —— 本次会话未检查

---

### 缺失的测试证据

在可以通过 `/story-done` 标记为 COMPLETE 之前，必须提供测试证据的故事：

- **[故事标题]**（`[路径]`）—— Logic 故事没有测试文件。
  预期位置：`tests/unit/[system]/[story-slug]_test.[ext]`

[如果没有：]"所有 Logic 和 Integration 故事都有测试覆盖。"

---

### 平台专用结果 *（仅在提供了 `--platform` 时显示）*

| 平台 | 已运行检查 | 通过 | 失败 | 平台判定 |
|----------|-----------|--------|--------|-----------------|
| PC | [N] | [N] | [N] | PASS / FAIL |
| Console | [N] | [N] | [N] | PASS / FAIL |
| Mobile | [N] | [N] | [N] | PASS / FAIL |

**平台备注**：[未在通过/失败中体现的平台特定观察]

任何有一个或多个 FAIL 检查的平台都会导致总体 FAIL 判定。

---

### 判定：[PASS | PASS WITH WARNINGS | FAIL]

[判定规则 —— 第一条匹配的规则胜出：]

**FAIL** 如果满足**任意**以下条件：
- 自动化测试套件已运行并报告了一个或多个测试失败
- 任何批次 1（核心稳定性）检查返回 FAIL
- 任何批次 2（主要 sprint 机制或回归检查）返回 FAIL

**PASS WITH WARNINGS** 如果满足**所有**以下条件：
- 自动化测试 PASS 或 NOT RUN（开发者尚未确认）
- 所有批次 1 和批次 2 冒烟检查 PASS
- 一个或多个 Logic/Integration 故事有 MISSING 测试证据

**PASS** 如果满足**所有**以下条件：
- 自动化测试 PASS
- 所有批次中的所有冒烟检查 PASS 或 N/A
- 无 MISSING 测试证据条目
````

---

## 阶段 6：写入和门禁

在对话中呈现完整报告，然后询问：

"我是否可以将此冒烟检查报告写入 `production/qa/smoke-[date].md`？"

仅在批准后写入。

写入后，给出门禁判定：

**如果判定为 FAIL：**

"冒烟检查失败。在解决以下失败之前，不要交接给 QA：

[列出每个失败的自动化测试或冒烟检查，附上一行描述]

修复失败项并在 QA 交接前重新运行 `/smoke-check` 以重新通过门禁。"

**如果判定为 PASS WITH WARNINGS：**

"冒烟检查通过但有警告。该版本已准备好进行手动 QA。

在受影响故事上运行 `/story-done` 之前需要解决的建议性事项：
[列出 MISSING 测试证据条目]

QA 交接：将 `production/qa/qa-plan-[sprint].md` 分享给 qa-tester Agent 以开始手动验证。"

**如果判定为 PASS：**

"冒烟检查干净通过。该版本已准备好进行手动 QA。

QA 交接：将 `production/qa/qa-plan-[sprint].md` 分享给 qa-tester Agent 以开始手动验证。"

---

## 协作协议

- **切勿将 NOT RUN 视为自动 FAIL** —— 将其记录为 NOT RUN，让开发者手动确认状态。未确认的 NOT RUN 导致 PASS WITH WARNINGS，而非 FAIL。
- **切勿自动修复失败** —— 报告失败并说明必须解决的内容。不要尝试编辑源代码或测试文件。
- **PASS WITH WARNINGS 不阻止 QA 交接** —— 它记录的是后续跟进的建议性缺口。
- **`quick` 参数**跳过阶段 3（覆盖扫描）和阶段 4 批次 3。用于修复特定失败后的快速重新检查。
- 对所有手动冒烟检查验证使用 `AskUserQuestion`。
- **未经询问切勿写入报告** —— 阶段 6 需要在创建任何文件之前获得明确批准。
