# 路径特定规则

`.claude/rules/` 中的规则在编辑匹配路径的文件时自动执行：

| 规则文件 | 路径模式 | 执行内容 |
| ---- | ---- | ---- |
| `gameplay-code.md` | `src/gameplay/**` | 数据驱动数值、delta time、禁止 UI 引用 |
| `engine-code.md` | `src/core/**` | 热路径零分配、线程安全、API 稳定性 |
| `ui-code.md` | `src/ui/**` | 禁止拥有游戏状态、本地化就绪 |
| `test-standards.md` | `tests/**` | 测试命名、覆盖率要求、Fixture 模式 |
| `prototype-code.md` | `prototypes/**` | 放宽标准、需要 README、记录假设 |
