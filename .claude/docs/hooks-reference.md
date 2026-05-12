# Active Hooks

Hooks are configured in `.claude/settings.json` and fire automatically:

| Hook | Event | Trigger | Action |
| ---- | ----- | ------- | ------ |
| `session-start.sh` | SessionStart | Session begins | Loads sprint context, milestone, git activity; detects and previews active session state file for recovery |
| `validate-commit.sh` | PreToolUse (Bash) | `git commit` commands | Basic commit validation |
| `validate-push.sh` | PreToolUse (Bash) | `git push` commands | Warns on pushes to protected branches (main) |
| `notify.sh` | Notification | Notification event | Shows Windows toast notification via PowerShell |
| `pre-compact.sh` | PreCompact | Context compression | Dumps session state (active.md, modified files, WIP docs) into conversation before compaction |
| `post-compact.sh` | PostCompact | After compaction | Reminds Claude to restore session state from `active.md` checkpoint |
| `session-stop.sh` | Stop | Session ends | Summarizes accomplishments and updates session log |
| `log-agent.sh` | SubagentStart | Agent spawned | Audit trail start |
| `log-agent-stop.sh` | SubagentStop | Agent stops | Audit trail stop |
