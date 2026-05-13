# 主程序员 — 代理记忆

## 技能编写约定

### 前置元数据
- 字段：`name`、`description`、`argument-hint`、`user-invocable`、`allowed-tools`
- 在隔离环境中运行的只读分析技能还带有 `context: fork` 和 `agent:`
- 交互式技能（写入文件、提问）不使用 `context: fork`
- `AskUserQuestion` 是一种在技能正文中描述的用法模式——它不在 `allowed-tools` 前置元数据中列出（没有现有技能这样做）

### 文件布局
- 技能位于 `.claude/skills/<name>/SKILL.md`（每个技能一个子目录，绝不使用扁平 .md）
- 章节标题对阶段使用 `##`，对子章节使用 `###`
- 阶段名称遵循"阶段 N：动词 名词"模式（例如，"阶段 1：找到故事"）
- 输出格式模板放在围栏代码块中

### 已知规范路径（在新技能中引用前请验证）
- 技术债务登记册：`docs/tech-debt-register.md`（而非 `production/tech-debt.md`）
- 冲刺文件：`production/sprints/`
- 史诗故事文件：`production/epics/[epic-slug]/story-[NNN]-[slug].md`
- 控制清单：`docs/architecture/control-manifest.md`
- 会话状态：`production/session-state/active.md`
- 系统索引：`design/gdd/systems-index.md`
- 引擎参考：`docs/engine-reference/[engine]/VERSION.md`

### 已完成的技能
- `story-done` — 故事结束的完成握手（阶段 1-8，写入故事文件）
