---
name: qa-plan
description: "为迭代或功能生成 QA 测试计划。读取 GDD 和 Story 文件，按测试类型（Logic/Integration/Visual/UI）分类 Story，生成涵盖自动化测试要求、手动测试用例、冒烟测试范围和试玩签字要求的结构化测试计划。在迭代开始前或启动主要功能时运行。"
argument-hint: "[sprint | feature: system-name | story: path]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, AskUserQuestion
agent: qa-tester
---

# QA 计划

此 Skill 为迭代、功能或单个 Story 生成结构化 QA 计划。它读取所有范围内的 Story 文件及其引用的 GDD，按测试类型对每个 Story 分类，并生成一份计划，准确告诉开发者要自动化什么、手动验证什么、冒烟测试范围是什么以及何时引入试玩者。

在迭代开始之前运行此 Skill，以便团队提前知道需要什么测试工作。在实现之后写的测试计划是事后总结，而非计划。

**输出：** `production/qa/qa-plan-[迭代简称]-[日期].md`

---

## 阶段 1 + 2：解析范围 + 加载输入

从参数确定范围（`sprint`/`feature: [系统名称]`/`story: [路径]`），然后读取每个范围内的 Story 文件，提取标题、类型、验收标准、实现文件、引擎备注、GDD 引用、ADR 引用和依赖关系。加载支持上下文（系统索引、引用的 GDD 的验收标准和公式章节、控制清单）。

---

## 阶段 3：分类每个 Story

根据验收标准中的指标分配类型：Logic（公式、状态转换）、Integration（跨系统信号、存档/读档）、Visual/Feel（动画、VFX）、UI（菜单、HUD）、Config/Data（仅数据文件）。混合 Story 按最高实现风险分配类型。

---

## 阶段 4 + 5：生成测试计划 + 写入输出

生成完整的 QA 计划文档，包含：测试摘要表、自动化测试要求（含测试路径、测试内容、边缘情况）、手动 QA 检查清单（含验证方法、签字人、证据）、冒烟测试范围、试玩要求和完成定义。展示计划并请求写入批准。

协作协议：不未经询问而写入、保守分类、不超出验收标准和 GDD 公式创造测试用例、试玩要求是建议性的。
