---
paths:
  - "src/core/**"
---

# 引擎代码规则

- 热路径中零分配（更新循环、渲染、物理）——预分配、池化、复用
- 所有引擎 API 必须是线程安全的，或明确记录为仅单线程
- 每次优化前后都进行性能分析——记录实测数据
- 引擎代码绝不能依赖玩法代码（严格的依赖方向：引擎 <- 玩法）
- 每个公共 API 必须在其文档注释中有使用示例
- 对公共接口的更改需要弃用期和迁移指南
- 所有资源使用 RAII / 确定性清理
- 所有引擎系统必须支持优雅降级
- 在编写引擎 API 代码之前，查阅 `docs/engine-reference/` 获取当前引擎版本，并对照参考文档验证 API

## 示例

**正确**（热路径零分配）：

```gdscript
# 预先分配的数组，每帧复用
var _nearby_cache: Array[Node3D] = []

func _physics_process(delta: float) -> void:
    _nearby_cache.clear()  # 复用，不重新分配
    _spatial_grid.query_radius(position, radius, _nearby_cache)
```

**错误**（在热路径中分配）：

```gdscript
func _physics_process(delta: float) -> void:
    var nearby: Array[Node3D] = []  # 违规：每帧分配
    nearby = get_tree().get_nodes_in_group("enemies")  # 违规：每帧查询场景树
```
