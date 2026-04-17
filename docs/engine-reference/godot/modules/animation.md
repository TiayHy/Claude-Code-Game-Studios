# Godot 动画 —— 速查表

最后验证：2026-02-12 | 引擎：Godot 4.6

## 自 ~4.3（LLM 截止日期）以来的变化

### 4.6 变化
- **IK 系统完全恢复**：3D 骨骼的完整逆运动学支持
  - CCDIK、FABRIK、Jacobian IK、Spline IK、TwoBoneIK
  - 通过 `SkeletonModifier3D` 节点应用（非旧的 IK 方式）
- **动画编辑器体验优化**：Bezier 节点组支持独显/隐藏/锁定/删除；时间轴可拖拽

### 4.5 变化
- **BoneConstraint3D**：将骨骼绑定到其他骨骼，支持修饰符
  - `AimModifier3D`、`CopyTransformModifier3D`、`ConvertTransformModifier3D`

### 4.3 变化（训练数据中已有）
- **AnimationMixer**：AnimationPlayer 和 AnimationTree 的基类
  - `method_call_mode` → `callback_mode_method`
  - `playback_active` → `active`
  - `bone_pose_updated` 信号 → `skeleton_updated`
- **`Skeleton3D.add_bone()`**：现返回 `int32`（原为 `void`）

## 当前 API 模式

### AnimationPlayer（API 未变，基类已改）
```gdscript
@onready var anim_player: AnimationPlayer = %AnimationPlayer

func play_attack() -> void:
    anim_player.play(&"attack")
    await anim_player.animation_finished
```

### IK 设置（4.6 — 新增）
```gdscript
# 将基于 SkeletonModifier3D 的 IK 节点添加为 Skeleton3D 的子节点
# 可用类型：
# - SkeletonModifier3D（基类）
# - TwoBoneIK（手臂、腿部）
# - FABRIK（链条、触手）
# - CCDIK（尾巴、脊柱）
# - Jacobian IK（复杂多关节）
# - Spline IK（沿曲线）

# 在编辑器或代码中配置：
# 1. 将 IK 修饰符节点添加为 Skeleton3D 的子节点
# 2. 设置目标骨骼和末端骨骼
# 3. 添加 Marker3D 作为 IK 目标
# 4. IK 求解器每帧自动运行
```

### BoneConstraint3D（4.5 — 新增）
```gdscript
# 添加为 Skeleton3D 的子节点
# 类型：
# - AimModifier3D：让骨骼指向目标
# - CopyTransformModifier3D：镜像另一骨骼的变换
# - ConvertTransformModifier3D：重新映射变换值
```

### AnimationTree（4.3 起基类已改）
```gdscript
# AnimationTree 现继承自 AnimationMixer（而非直接继承 Node）
# 使用 AnimationMixer 属性：
@onready var anim_tree: AnimationTree = %AnimationTree

func _ready() -> void:
    anim_tree.active = true  # 不要用 playback_active（4.3 起已废弃）
```

## 常见错误
- 使用 `playback_active` 而非 `active`（4.3 起废弃）
- 使用 `bone_pose_updated` 信号而非 `skeleton_updated`（4.3 起更名）
- 使用旧的 IK 方式而非 SkeletonModifier3D 系统（4.6 起恢复）
- 类型检查动画节点时未检查 `is AnimationMixer`