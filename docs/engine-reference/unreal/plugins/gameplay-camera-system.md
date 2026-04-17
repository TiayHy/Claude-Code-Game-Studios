# Unreal Engine 5.7 — Gameplay Camera System

**Last verified:** 2026-02-13
**Status:** ⚠️ Experimental (introduced in UE 5.5)
**Plugin:** `GameplayCameras` (built-in, enable in Plugins)

---

## 概述

**Gameplay Camera System** 是 UE 5.5 中引入的模块化摄像机管理框架。它用灵活基于节点的摄像机模式系统取代了传统摄像机设置，处理摄像机模式切换、混合以及上下文相关的摄像机行为。

**使用 Gameplay Cameras 的场景：**
- 动态摄像机行为（第三人称、瞄准、载具、电影感）
- 上下文感知的摄像机切换（战斗、探索、对话）
- 模式之间平滑的摄像机混合
- 程序化摄像机运动（摄像机抖动、延迟、偏移）

**⚠️ 警告：** 此插件在 UE 5.5-5.7 中为实验性功能。预计未来版本会有 API 变更。

---

## 核心概念

### 1. **Camera Rig**
- 定义摄像机配置（位置、旋转、FOV 等）
- 模块化节点图（类似于 Material Editor）

### 2. **Camera Director**
- 管理当前激活的 Camera Rig
- 处理 Camera Rig 之间的混合

### 3. **Camera Nodes**
- 摄像机行为的构建块：
  - **Position Nodes**：Orbit、Follow、Fixed Position
  - **Rotation Nodes**：Look At、Match Actor Rotation
  - **Modifiers**：Camera Shake、Lag、Offset

---

## 设置

### 1. 启用 Plugin

`Edit > Plugins > Gameplay Cameras > Enabled > Restart`

### 2. 添加 Camera Component

```cpp
#include "GameplayCameras/Public/GameplayCameraComponent.h"

UCLASS()
class AMyCharacter : public ACharacter {
    GENERATED_BODY()

public:
    AMyCharacter() {
        // 创建摄像机组件
        CameraComponent = CreateDefaultSubobject<UGameplayCameraComponent>(TEXT("GameplayCamera"));
        CameraComponent->SetupAttachment(RootComponent);
    }

protected:
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Camera")
    TObjectPtr<UGameplayCameraComponent> CameraComponent;
};
```

---

## 创建 Camera Rig

### 1. 创建 Camera Rig Asset

1. Content Browser > Gameplay > Gameplay Camera Rig
2. 打开 Camera Rig Editor（基于节点的图）

### 2. 构建 Camera Rig（示例：第三人称）

**节点配置：**
```
Actor Position (Character)
  ↓
Orbit Node (Orbit around character)
  ↓
Offset Node (Shoulder offset)
  ↓
Look At Node (Look at character)
  ↓
Camera Output
```

---

## Camera Nodes

### Position Nodes

#### Orbit Node（第三人称）
- 围绕目标 Actor 旋转
- 可配置：
  - **Orbit Distance**：距目标的距离（例如 300 单位）
  - **Pitch Range**：最小/最大俯仰角
  - **Yaw Range**：最小/最大偏航角

#### Follow Node（平滑跟随）
- 带延迟跟随目标
- 可配置：
  - **Lag Speed**：摄像机追赶的速度
  - **Offset**：距目标的固定偏移

#### Fixed Position Node
- 世界空间中的静态摄像机位置

---

### Rotation Nodes

#### Look At Node
- 将摄像机指向目标
- 可配置：
  - **Target**：要注视的 Actor 或组件
  - **Offset**：注视偏移（例如瞄准头部而非脚部）

#### Match Actor Rotation
- 匹配目标 Actor 的旋转
- 适用于第一人称或载具摄像机

---

### Modifier Nodes

#### Camera Shake
- 添加程序化抖动（例如脚步声、爆炸）
- 可配置：
  - **Shake Pattern**：Perlin 噪声、正弦波、自定义
  - **Amplitude**：抖动强度

#### Camera Lag
- 摄像机移动的平滑阻尼
- 可配置：
  - **Lag Speed**：阻尼因子（0 = 即时，越高 = 延迟越大）

#### Offset Node
- 从计算位置出发的静态偏移
- 适用于肩部摄像机偏移

---

## Camera Director（切换 Rig）

### 分配 Camera Rig

```cpp
#include "GameplayCameras/Public/GameplayCameraComponent.h"

void AMyCharacter::SetCameraMode(UGameplayCameraRig* NewRig) {
    if (CameraComponent) {
        CameraComponent->SetCameraRig(NewRig);
    }
}
```

### 在 Camera Rig 之间混合

```cpp
// 在 0.5 秒内混合到瞄准摄像机
CameraComponent->BlendToCameraRig(AimingCameraRig, 0.5f);
```

---

## 示例：第三人称 + 瞄准

### 1. 创建两个 Camera Rigs

**第三人称 Rig：**
```
Actor Position → Orbit (distance: 300) → Look At → Output
```

**瞄准 Rig：**
```
Actor Position → Orbit (distance: 150) → Offset (shoulder) → Look At → Output
```

### 2. 瞄准时切换

```cpp
UPROPERTY(EditAnywhere, Category = "Camera")
TObjectPtr<UGameplayCameraRig> ThirdPersonRig;

UPROPERTY(EditAnywhere, Category = "Camera")
TObjectPtr<UGameplayCameraRig> AimingRig;

void StartAiming() {
    CameraComponent->BlendToCameraRig(AimingRig, 0.3f); // 0.3秒内混合
}

void StopAiming() {
    CameraComponent->BlendToCameraRig(ThirdPersonRig, 0.3f);
}
```

---

## 常见模式

### 过肩摄像机

```
Actor Position
  ↓
Orbit Node (distance: 250, yaw offset: 30°)
  ↓
Offset Node (X: 0, Y: 50, Z: 50) // Shoulder offset
  ↓
Look At Node (target: Character head)
  ↓
Output
```

---

### 载具摄像机

```
Vehicle Position
  ↓
Follow Node (lag: 0.2)
  ↓
Offset Node (behind vehicle: X: -400, Z: 150)
  ↓
Look At Node (target: Vehicle)
  ↓
Output
```

---

### 第一人称摄像机

```
Character Head Socket
  ↓
Match Actor Rotation
  ↓
Output
```

---

## Camera Shake

### 触发 Camera Shake

```cpp
#include "GameplayCameras/Public/GameplayCameraShake.h"

void TriggerExplosionShake() {
    if (APlayerController* PC = GetWorld()->GetFirstPlayerController()) {
        if (UGameplayCameraComponent* CameraComp = PC->FindComponentByClass<UGameplayCameraComponent>()) {
            CameraComp->PlayCameraShake(ExplosionShakeClass, 1.0f);
        }
    }
}
```

---

## 性能提示

- 限制摄像机抖动频率（不要每帧都触发）
- 谨慎使用摄像机延迟（高延迟值开销较大）
- 缓存摄像机 rig 引用（不要每帧都搜索）

---

## 调试

### 摄像机调试可视化

```cpp
// 控制台命令：
// GameplayCameras.Debug 1 - 显示激活的摄像机 rig 信息
// showdebug camera - 显示摄像机调试信息
```

---

## 从传统摄像机迁移

### 旧的 Spring Arm + Camera Component

```cpp
// ❌ 旧版：Spring Arm Component
USpringArmComponent* SpringArm;
UCameraComponent* Camera;

// ✅ 新版：Gameplay Camera Component
UGameplayCameraComponent* CameraComponent;
// 在 Camera Rig asset 中构建 orbit + look-at rig
```

---

## 局限性（实验性状态）

- **API 不稳定**：预计 UE 5.8+ 会有破坏性变更
- **文档有限**：官方文档仍在完善中
- **Blueprint 支持**：主要面向 C++（Blueprint 支持正在改进）
- **生产风险**：发布前充分测试

---

## Sources
- https://docs.unrealengine.com/5.7/en-US/gameplay-cameras-in-unreal-engine/
- UE 5.5+ Release Notes
- **注意：** 此系统为实验性功能。请务必查看最新官方文档以了解 API 变更。
