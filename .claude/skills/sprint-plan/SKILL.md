---
name: sprint-plan
description: "基于当前里程碑、已完成工作和可用产能生成新的迭代计划或更新现有计划。从生产文档和设计待办列表中提取上下文。"
argument-hint: "[new|update|status] [--review full|lean|solo]"
user-invocable: true
allowed-tools: Read, Glob, Grep, Write, Edit, Task, AskUserQuestion
context: |
  !ls production/sprints/ 2>/dev/null
---

## 阶段 0：解析参数

提取模式参数（`new`、`update` 或 `status`）并解析审查模式。

---

## 阶段 1 + 2：收集上下文 + 生成输出

读取当前里程碑、上一个迭代（如有）、设计文档和风险登记册。对于 `new`，生成包含迭代目标、产能、必须/应该/可有的任务表、遗留项、风险、外部依赖和完成定义的迭代计划。对于 `status`，生成包含进度、已完成/进行中/未开始/被阻塞任务的状态报告。

---

## 阶段 3：写入迭代状态文件 + 阶段 4：制作人可行性关卡 + 阶段 5：QA 计划关卡 + 阶段 6：后续步骤

生成新迭代计划后，同时写入 `production/sprint-status.yaml` 作为机器可读的 Story 状态数据源。在最终确定之前，根据审查模式通过制作人可行性关卡和 QA 计划关卡。写入后建议后续步骤：`/qa-plan sprint`、`/story-readiness [story-file]`、`/dev-story [story-file]`。
