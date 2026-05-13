# 独立游戏开发——快速入门指南

## 这是什么？

一个简化版的 Claude Code Agent 架构，用于独立游戏开发。13 个专业 Agent，21 项 Skill，4 个阶段——专为需要快速原型制作和迭代的独立开发者和
小团队设计。

## 4 个阶段

1. **概念** — 确定你的游戏创意，选择引擎，设定核心支柱和视觉风格
2. **原型** — 构建、测试、验证核心机制（最重要的阶段）
3. **生产** — 实现功能，代码审查，迭代
4. **打磨与发布** — QA、Bug 修复、打磨、发布

## 如何开始

### 首次使用

运行 `/start`——它会检测你的项目状态并将你引导到正确的位置。

### 有创意想法？

运行 `/brainstorm` 来探索并记录你的游戏概念。

### 准备开始构建？

运行 `/prototype` 用可抛弃代码验证你的核心机制。
**尽早并经常制作原型。** 在证明有趣之前不要构建系统。

### 原型验证通过？

运行 `/gate-check` 确认你已准备好进入生产阶段，然后：
1. 为每个主要系统运行 `/design-system`
2. 使用 `/dev-story` 实现
3. 合并前运行 `/code-review`

### 有可玩演示了？

运行 `/gate-check` 进入打磨与发布阶段，然后：
1. `/qa-plan` 创建测试计划
2. `/smoke-check` 验证一切正常
3. `/playtest-report` 分析反馈
4. `/changelog` 生成发布说明

## 可用 Agent

| 我需要…… | 使用此 Agent |
|-------------|---------------|
| 设计一个机制 | `game-designer` |
| 编写玩法代码 | `gameplay-programmer` |
| 创建着色器 | `technical-artist` |
| 设计关卡 | `level-designer` |
| 审查代码 | `lead-programmer` |
| 撰写对话 | `writer` |
| 构建 UI | `ui-programmer` |
| 定义视觉风格 | `art-director` |
| Godot 特定帮助 | `godot-specialist` |
| 架构决策 | `technical-director` |
| 测试我的游戏 | `qa-tester`（仅阶段 4） |

## 核心原则

- **原型优先**——在构建系统之前验证趣味性
- **QA 推迟**——在有可玩演示之前不要测试
- **你是导演**——Agent 建议，你决定
- **扁平结构**——没有层级，只需为工作选择正确的 Agent
