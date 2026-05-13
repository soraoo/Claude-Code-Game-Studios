# 目录结构

```
/
├── CLAUDE.md                    # 主配置
├── .claude/                     # Agent 定义、Skills、Hooks、Rules、文档
├── src/                         # 游戏源码
├── assets/                      # 游戏资源（美术、音频、VFX、着色器、数据）
├── design/                      # 游戏设计文档（GDD、快速规格）
├── docs/                        # 技术文档
│   └── engine-reference/        # 精选引擎 API 快照（版本锁定）
├── tests/                       # 测试套件（推迟到阶段 4）
├── prototypes/                  # 可抛弃原型（与 src/ 隔离）
└── production/                  # 生产管理
    ├── session-state/           # 临时会话状态 (active.md)
    └── session-logs/            # 会话审计跟踪
```
