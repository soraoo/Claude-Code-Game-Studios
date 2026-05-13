# settings.local.json 模板

创建 `.claude/settings.local.json` 用于不应提交到版本控制的个人偏好覆盖。
将其添加到 `.gitignore`。

## 示例 settings.local.json

```json
{
  "permissions": {
    "allow": [
      "Bash(git *)",
      "Bash(npm *)",
      "Read",
      "Glob",
      "Grep"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(git push --force *)"
    ]
  }
}
```

## 权限模式

Claude Code 支持不同的权限模式。游戏开发推荐：

### 开发期间（默认）
使用**普通模式** — Claude 在运行大多数命令之前询问。这对生产代码最安全。

### 原型制作期间
使用**自动接受模式**并限制范围 — 对可抛弃代码实现更快的迭代。仅在 `prototypes/` 目录中工作时使用。

### 代码审查期间
使用**只读**权限 — Claude 可以阅读和搜索，但不能修改文件。

## 本地自定义 Hook

你可以在 `settings.local.json` 中添加个人 Hook，扩展（而非覆盖）项目 Hook。
例如，在构建完成时添加通知：

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'echo Session ended at $(date)'",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```
