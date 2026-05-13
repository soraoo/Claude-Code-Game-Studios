---
paths:
  - "src/gameplay/**"
---

# 玩法代码规则

- 所有玩法数值必须来自外部配置/数据文件，绝不可硬编码
- 所有时间依赖的计算使用 delta time（帧率无关）
- 不得直接引用 UI 代码——使用事件/信号进行跨系统通信
- 每个玩法系统必须实现一个清晰的接口
- 状态机必须有显式的转换表和记录在案的状态
- 为所有玩法逻辑编写单元测试——将逻辑与表现分离
- 在代码注释中记录每个功能实现的设计文档
- 不使用静态单例存储游戏状态——使用依赖注入

## 示例

**正确**（数据驱动）：

```gdscript
var damage: float = config.get_value("combat", "base_damage", 10.0)
var speed: float = stats_resource.movement_speed * delta
```

**错误**（硬编码）：

```gdscript
var damage: float = 25.0   # 违规：硬编码的玩法数值
var speed: float = 5.0      # 违规：未从配置读取，未使用 delta
```
