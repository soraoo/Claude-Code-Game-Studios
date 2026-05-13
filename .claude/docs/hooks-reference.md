# 活跃 Hook

Hook 配置在 `.claude/settings.json` 中并自动触发：

| Hook | 事件 | 触发条件 | 操作 |
| ---- | ----- | ------- | ------ |
| `session-start.sh` | SessionStart | 会话开始 | 加载迭代上下文、里程碑、Git 活动；检测并预览活跃会话状态文件以进行恢复 |
| `validate-commit.sh` | PreToolUse (Bash) | `git commit` 命令 | 基本提交验证 |
| `validate-push.sh` | PreToolUse (Bash) | `git push` 命令 | 推送到受保护分支时警告 (main) |
| `notify.sh` | Notification | 通知事件 | 通过 PowerShell 显示 Windows Toast 通知 |
| `pre-compact.sh` | PreCompact | 上下文压缩 | 在压缩前转储会话状态（active.md、修改的文件、进行中的文档）到对话中 |
| `post-compact.sh` | PostCompact | 压缩后 | 提醒 Claude 从 `active.md` 检查点恢复会话状态 |
| `session-stop.sh` | Stop | 会话结束 | 总结成果并更新会话日志 |
| `log-agent.sh` | SubagentStart | Agent 被派生 | 审计跟踪开始 |
| `log-agent-stop.sh` | SubagentStop | Agent 停止 | 审计跟踪结束 |
