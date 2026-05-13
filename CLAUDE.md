# Claude Code Game Studios — 独立游戏模板

单人/小团队游戏开发，化繁为简。13 个代理，21 项技能，4 个阶段。

## 技术栈

- **引擎**：Godot 4.6
- **语言**：GDScript
- **版本控制**：Git
- **构建系统**：Godot CLI 无头构建
- **资源管线**：Godot 导入系统

## 项目结构

@.claude/docs/directory-structure.md

## 开发阶段

1. **概念** — 确定游戏创意，选择引擎，设定核心支柱，视觉风格
2. **原型** — 构建、测试、验证核心机制（快速迭代）
3. **生产** — 实现功能，代码审查，迭代
4. **打磨与发布** — 质量保证，Bug 修复，发布

## 工作流程

**原型优先。快速迭代。在拥有可玩游戏之前，推迟质量保证。**

- 运行 `/start` 进行首次设置
- 随时运行 `/help` 获取上下文感知指导
- 尽早并经常运行 `/prototype` — 在构建之前进行验证
- 质量保证仅在阶段 4（打磨与发布）中激活

## 引擎版本参考

@docs/engine-reference/godot/VERSION.md

## 核心技能

/start, /help, /brainstorm, /setup-engine, /project-stage-detect,
/quick-design, /art-bible, /design-system, /prototype, /gate-check,
/dev-story, /code-review, /story-done, /create-epics, /create-stories,
/sprint-plan, /bug-report, /qa-plan, /smoke-check, /playtest-report, /changelog

## 可用代理

@.claude/docs/agent-roster.md

## 编码标准

@.claude/docs/coding-standards.md

## 上下文管理

@.claude/docs/context-management.md

## 协作协议

**用户驱动的协作，而非自主执行。**
每项任务遵循：**提问 -> 方案 -> 决策 -> 草稿 -> 批准**

- 代理在使用 Write/Edit 工具之前必须询问
- 代理在请求批准之前必须展示草稿或摘要
- 未经用户指示，不得提交
