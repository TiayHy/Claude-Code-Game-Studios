# Godot — 破坏性变更

最后验证日期：2026-02-12

本文档记录 Godot 版本之间的变更，侧重于 LLM 训练截止日期后（4.4+）的变化。

## 4.5 → 4.6（2026 年 1 月 — 截止日期后，高风险）

| 子系统 | 变更 | 详情 |
|-----------|--------|---------|
| 物理 | Jolt 现为默认 3D 物理引擎 | 新项目自动使用 Jolt。现有项目保持原有设置。部分 `HingeJoint3D` 属性（如 `damp`）仅在 GodotPhysics 下有效。 |
| 渲染 | 发光在色调映射之前处理 | 之前在色调映射之后。带有发光效果的场景将呈现不同外观。调整 `WorldEnvironment` 中的强度/混合模式。 |
| 渲染 | Windows 默认使用 D3D12 | 之前为 Vulkan。为了更好的驱动兼容性。 |
| 渲染 | AgX 色调映射器新增控制项 | 新增白点和对比度参数。 |
| 核心 | 四元数初始化为单位向量 | 之前为零向量。对大多数代码影响不大，但技术上属于破坏性变更。 |
| UI | 双焦点系统 | 鼠标/触摸焦点与键盘/手柄焦点现已分离。不同输入方式下的视觉反馈不同。 |
| 动画 | IK 系统完全恢复 | CCDIK、FABRIK、Jacobian IK、Spline IK、TwoBoneIK 通过 `SkeletonModifier3D` 节点实现。 |
| 编辑器 | 新"现代"（Modern）主题为默认 | 灰阶取代蓝色调。恢复方式：编辑器设置 → 界面 → 主题 → 样式：经典 |
| 编辑器 | "选择模式"快捷键变更 | 新的"选择模式"（v 键）防止意外变换。旧模式改名为"变换模式"（q 键）。 |
| 2D | TileMapLayer 场景瓦片旋转 | 场景瓦片现在可以像图集瓦片一样旋转。 |
| 本地化 | CSV 复数形式支持 | 不再需要 Gettext 即可处理复数。新增上下文列。 |
| C# | 自动字符串提取 | 翻译字符串从 C# 代码中自动提取。 |
| 插件 | 新增 `EditorDock` 类 | 专用于插件停靠栏的容器，带布局控制功能。 |

## 4.4 → 4.5（2025 年末 — 截止日期后，高风险）

| 子系统 | 变更 | 详情 |
|-----------|--------|---------|
| GDScript | 新增可变参数 | 函数可接受 `...` 任意数量参数 — 新语言特性 |
| GDScript | `@abstract` 装饰器 | 抽象类和抽象方法现在可强制执行 |
| GDScript | 脚本回溯 | 即使在 Release 构建中也可获得详细调用栈 |
| 渲染 | 模板缓冲区支持 | 新增高级视觉效果能力 |
| 渲染 | SMAA 1x 抗锯齿 | 新增后处理抗锯齿选项 |
| 渲染 | 着色器烘焙器 | 预编译着色器 — 据报告某些演示场景启动速度提升 20 倍 |
| 渲染 | 弯曲法线贴图、镜面遮蔽 | 新增材质特性 |
| 无障碍 | 屏幕阅读器支持 | Control 节点通过 AccessKit 与无障碍工具配合工作 |
| 编辑器 | 实时翻译预览 | 在编辑器内以不同语言测试 GUI 布局 |
| 物理 | 3D 插值重构 | 从 RenderingServer 移至 SceneTree。API 未变但内部实现不同。 |
| 动画 | `BoneConstraint3D` | 新增：AimModifier3D、CopyTransformModifier3D、ConvertTransformModifier3D |
| 资源 | 新增 `duplicate_deep()` | 新增用于嵌套资源深度复制的显式方法 |
| 导航 | 独立的 2D 导航服务器 | 不再作为 3D NavigationServer 的代理；减小 2D 游戏的导出体积 |
| UI | `FoldableContainer` 节点 | 新增手风琴式容器，用于可折叠 UI 区域 |
| UI | 递归 Control 行为 | 可跨整个节点层级禁用鼠标/焦点交互 |
| 平台 | visionOS 导出支持 | 新增平台目标 |
| 平台 | SDL3 手柄驱动 | 手柄处理委托给 SDL 库 |
| 平台 | Android 16KB 分页支持 | Google Play 面向 Android 15+ 所需 |

## 4.3 → 4.4（2025 年中 — 接近截止日期，需验证）

| 子系统 | 变更 | 详情 |
|-----------|--------|---------|
| 核心 | `FileAccess.store_*` 返回 `bool` | 之前为 `void`。涉及方法：`store_8`、`store_16`、`store_32`、`store_64`、`store_buffer`、`store_csv_line`、`store_double`、`store_float`、`store_half`、`store_line`、`store_pascal_string`、`store_real`、`store_string`、`store_var` |
| 核心 | `OS.execute_with_pipe` | 新增可选 `blocking` 参数 |
| 核心 | `RegEx.compile/create_from_string` | 新增可选 `show_error` 参数 |
| 渲染 | `RenderingDevice.draw_list_begin` | 移除多个参数；新增 `breadcrumb` 参数 |
| 渲染 | 着色器纹理类型 | 参数/返回类型从 `Texture2D` 改为 `Texture` |
| 粒子 | `.restart()` 方法 | 新增可选 `keep_seed` 参数（CPU/GPU 2D/3D） |
| GUI | `RichTextLabel.push_meta` | 新增可选 `tooltip` 参数 |
| GUI | `GraphEdit.connect_node` | 新增可选 `keep_alive` 参数 |

## 4.2 → 4.3（训练数据中 — 低风险）

| 子系统 | 变更 | 详情 |
|-----------|--------|---------|
| 动画 | `Skeleton3D.add_bone` 返回 `int32` | 之前为 `void` |
| 动画 | `bone_pose_updated` 信号 | 被 `skeleton_updated` 取代 |
| TileMap | `TileMapLayer` 取代 `TileMap` | 每个图层一个节点，而非多层单节点 |
| 导航 | `NavigationRegion2D` | 移除 `avoidance_layers`、`constrain_avoidance` 属性 |
| 编辑器 | `EditorSceneFormatImporterFBX` | 更名为 `EditorSceneFormatImporterFBX2GLTF` |
| 动画 | `AnimationMixer` 基类 | `AnimationPlayer` 和 `AnimationTree` 现在继承 `AnimationMixer` |
