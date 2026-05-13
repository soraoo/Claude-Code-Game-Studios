# 技术偏好

<!-- 由 /setup-engine 填充。在开发过程中根据用户决策更新。 -->
<!-- 所有 Agent 引用此文件以获取项目特定的标准和惯例。 -->

## 引擎与语言

- **引擎**：[TO BE CONFIGURED — 运行 /setup-engine]
- **语言**：[TO BE CONFIGURED]
- **渲染**：[TO BE CONFIGURED]
- **物理**：[TO BE CONFIGURED]

## 输入与平台

<!-- 由 /setup-engine 写入。由 /ux-design、/ux-review、/test-setup、/team-ui 和 /dev-story 读取 -->
<!-- 以将交互规格、测试工具和实现限定到正确的输入方法。 -->

- **目标平台**：[TO BE CONFIGURED — 例如 PC、主机、手机、Web]
- **输入方式**：[TO BE CONFIGURED — 例如 键盘/鼠标、手柄、触摸、混合]
- **主要输入**：[TO BE CONFIGURED — 此游戏的主要输入方式]
- **手柄支持**：[TO BE CONFIGURED — 完整 / 部分 / 无]
- **触摸支持**：[TO BE CONFIGURED — 完整 / 部分 / 无]
- **平台说明**：[TO BE CONFIGURED — 任何平台特定的 UX 约束]

## 命名规范

- **类**：[TO BE CONFIGURED]
- **变量**：[TO BE CONFIGURED]
- **信号/事件**：[TO BE CONFIGURED]
- **文件**：[TO BE CONFIGURED]
- **场景/预制体**：[TO BE CONFIGURED]
- **常量**：[TO BE CONFIGURED]

## 性能预算

- **目标帧率**：[TO BE CONFIGURED]
- **帧预算**：[TO BE CONFIGURED]
- **绘制调用**：[TO BE CONFIGURED]
- **内存上限**：[TO BE CONFIGURED]

## 测试

- **框架**：[TO BE CONFIGURED]
- **最低覆盖率**：[TO BE CONFIGURED]
- **必需测试**：平衡公式、玩法系统、网络（如适用）

## 禁止模式

<!-- 添加绝不应出现在此项目代码库中的模式 -->
- [尚未配置——随着架构决策的制定逐步添加]

## 允许的库/插件

<!-- 在此添加已批准的第三方依赖 -->
- [尚未配置——随着依赖获得批准逐步添加]

## 架构决策日志

<!-- 快速参考，链接到 docs/architecture/ 中的完整 ADR -->
- [尚无 ADR——使用 /architecture-decision 创建一个]

## 引擎专家

<!-- 由 /setup-engine 在配置引擎时写入。 -->
<!-- 由 /code-review、/architecture-decision、/architecture-review 和团队 Skill 读取 -->
<!-- 以了解为引擎特定验证派生哪个专家。 -->

- **主要**：[TO BE CONFIGURED — 运行 /setup-engine]
- **语言/代码专家**：[TO BE CONFIGURED]
- **着色器专家**：[TO BE CONFIGURED]
- **UI 专家**：[TO BE CONFIGURED]
- **其他专家**：[TO BE CONFIGURED]
- **路由说明**：[TO BE CONFIGURED]

### 文件扩展名路由

<!-- Skill 使用此表按文件类型选择正确的专家。 -->
<!-- 如果某行显示 [TO BE CONFIGURED]，对该文件类型回退到主要专家。 -->

| 文件扩展名 / 类型 | 应派生的专家 |
|-----------------------|---------------------|
| 游戏代码（主要语言） | [TO BE CONFIGURED] |
| 着色器 / 材质文件 | [TO BE CONFIGURED] |
| UI / 界面文件 | [TO BE CONFIGURED] |
| 场景 / 预制体 / 关卡文件 | [TO BE CONFIGURED] |
| 原生扩展 / 插件文件 | [TO BE CONFIGURED] |
| 通用架构审查 | 主要 |
