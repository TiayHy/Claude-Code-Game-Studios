# Unreal Engine 5.7 — 动画模块参考

**最后验证日期：** 2026-02-13
**知识差距：** UE 5.7 动画创作改进、Control Rig 2.0

---

## 概述

UE 5.7 动画系统：
- **Animation Blueprint**：基于状态机的动画逻辑
- **Control Rig**：运行时程序化动画（UE5 生产可用）
- **IK Rig + Retargeter**：现代重定向系统
- **Sequencer**：电影级动画

---

## Animation Blueprint

### 创建 Animation Blueprint

1. Content Browser > Right Click > Animation > Animation Blueprint
2. 选择父类：`AnimInstance`
3. 选择骨架

### 动画状态机

```cpp
// In Animation Blueprint Event Graph:
// - State Machine drives animation states (Idle, Walk, Run, Jump)
// - Blend Spaces for directional movement

// 在 C++ 中访问：
UAnimInstance* AnimInstance = Mesh->GetAnimInstance();
AnimInstance->Montage_Play(AttackMontage);
```

---

## 播放动画蒙太奇

### Animation Montage

```cpp
// 播放蒙太奇
UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();
AnimInstance->Montage_Play(AttackMontage, 1.0f);

// 停止蒙太奇
AnimInstance->Montage_Stop(0.2f, AttackMontage);

// 检查蒙太奇是否正在播放
bool bIsPlaying = AnimInstance->Montage_IsPlaying(AttackMontage);
```

### 蒙太奇通知事件

```cpp
// 在 Animation Montage 中添加通知事件（右键时间轴 > Add Notify > New Notify）
// 在 C++ 中实现：

UCLASS()
class UMyAnimInstance : public UAnimInstance {
    GENERATED_BODY()

public:
    UFUNCTION()
    void AnimNotify_AttackHit() {
        // 到达通知时调用
        DealDamage();
    }
};
```

---

## Blend Spaces

### 1D Blend Space（速度混合）

```cpp
// 创建：Content Browser > Animation > Blend Space 1D
// 水平轴：Speed（0 = Idle，1 = Walk，2 = Run）
// 在关键点添加动画

// 在 Anim Blueprint 中使用：
// - Get speed from character
// - Feed into Blend Space
```

### 2D Blend Space（方向运动）

```cpp
// 创建：Content Browser > Animation > Blend Space
// 水平轴：Direction X（-1 到 1）
// 垂直轴：Direction Y（-1 到 1）
// 放置动画（Fwd、Back、Left、Right、对角线）
```

---

## Control Rig（程序化动画）

### 创建 Control Rig

1. Content Browser > Animation > Control Rig
2. 选择骨架
3. 构建绑骨层级（骨骼、控制点、IK）

### 在 Animation Blueprint 中使用 Control Rig

```cpp
// 在 Anim Blueprint 中添加 "Control Rig" 节点
// Assign Control Rig asset
// 运行时程序化修改骨骼
```

### C++ 中使用 Control Rig

```cpp
// 获取 control rig 组件
UControlRig* ControlRig = /* Get from animation instance */;

// 设置控制值
ControlRig->SetControlValue<FVector>(TEXT("IK_Hand_R"), TargetLocation);
```

---

## IK Rig & Retargeting（UE5）

### 创建 IK Rig

1. Content Browser > Animation > IK Rig
2. 选择骨架
3. 添加 IK goals（手、脚）
4. 设置求解器链

### 重定向动画

1. 为源骨架创建 IK Rig
2. 为目标骨架创建 IK Rig
3. 创建 IK Retargeter 资源
4. 分配源和目标 IK Rigs
5. 批量重定目标动画

### C++ 中重定向

```cpp
// 重定向主要在编辑器中进行
// Animations are retargeted once, then used normally
```

---

## Animation Notify States

### 自定义 Notify State（基于持续时间的事件）

```cpp
UCLASS()
class UAnimNotifyState_Invulnerable : public UAnimNotifyState {
    GENERATED_BODY()

public:
    virtual void NotifyBegin(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation, float TotalDuration, const FAnimNotifyEventReference& EventReference) override {
        // 开始无敌
        AMyCharacter* Character = Cast<AMyCharacter>(MeshComp->GetOwner());
        Character->bIsInvulnerable = true;
    }

    virtual void NotifyEnd(USkeletalMeshComponent* MeshComp, UAnimSequenceBase* Animation, const FAnimNotifyEventReference& EventReference) override {
        // 结束无敌
        AMyCharacter* Character = Cast<AMyCharacter>(MeshComp->GetOwner());
        Character->bIsInvulnerable = false;
    }
};
```

---

## Skeletal Mesh & Sockets

### 附加物体到 Socket

```cpp
// 在 Skeletal Mesh Editor 中创建 socket（Skeleton Tree > Add Socket）

// 将组件附加到 socket
UStaticMeshComponent* Weapon = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Weapon"));
Weapon->SetupAttachment(GetMesh(), TEXT("hand_r_socket"));
```

---

## Animation Curves

### 使用 Animation Curves

```cpp
// 添加曲线到动画：
// Animation Editor > Curves > Add Curve

// 在 Anim Blueprint 或 C++ 中读取曲线值：
UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();
float CurveValue = AnimInstance->GetCurveValue(TEXT("MyCurve"));
```

---

## Root Motion

### 启用 Root Motion

```cpp
// 在 Animation Sequence 中：Asset Details > Root Motion > Enable Root Motion

// 在 Character 类中：
GetCharacterMovement()->bAllowPhysicsRotationDuringAnimRootMotion = true;
```

---

## Animation Layers（链接 Anim Graphs）

### 使用链接的 Anim Layers

```cpp
// 为各层创建单独的 Anim Blueprints（如上半身、下半身）
// 在主 Anim Blueprint 中链接：Add "Linked Anim Graph" 节点

// 动态切换层：
UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();
AnimInstance->LinkAnimClassLayers(NewLayerClass);
```

---

## Sequencer（电影动画）

### 创建 Sequence

1. Content Browser > Cinematics > Level Sequence
2. 添加轨道：Camera、Character、Animation 等

### 从 C++ 播放 Sequence

```cpp
#include "LevelSequenceActor.h"
#include "LevelSequencePlayer.h"

ALevelSequenceActor* SequenceActor = /* Spawn or find in level */;
SequenceActor->GetSequencePlayer()->Play();
```

---

## 性能提示

### 动画优化

```cpp
// LOD（细节级别）用于骨骼网格体
// Reduce bone count for distant characters

// Anim Blueprint 优化：
// - Use "Anim Node Relevancy"（不可见时跳过更新）
// - Disable updates when off-screen：
GetMesh()->VisibilityBasedAnimTickOption = EVisibilityBasedAnimTickOption::OnlyTickPoseWhenRendered;
```

---

## 调试

### 动画调试可视化

```cpp
// 控制台命令：
// showdebug animation - 显示动画状态信息
// a.VisualizeSkeletalMeshBones 1 - 显示骨架骨骼

// 绘制调试骨骼：
DrawDebugCoordinateSystem(GetWorld(), BoneLocation, BoneRotation, 50.0f, false, -1.0f, 0, 2.0f);
```

---

## 参考来源
- https://docs.unrealengine.com/5.7/en-US/animation-in-unreal-engine/
- https://docs.unrealengine.com/5.7/en-US/control-rig-in-unreal-engine/
- https://docs.unrealengine.com/5.7/en-US/ik-rig-in-unreal-engine/
