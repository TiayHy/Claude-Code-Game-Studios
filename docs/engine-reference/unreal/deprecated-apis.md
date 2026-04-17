# Unreal Engine 5.7 — 已废弃的 API

**最后验证日期：** 2026-02-13

已废弃 API 及替代方案速查表。
格式：**不要使用 X** → **改用 Y**

---

## 输入

| 已废弃 | 替代方案 | 说明 |
|------------|-------------|-------|
| `InputComponent->BindAction()` | Enhanced Input `BindAction()` | 新输入系统 |
| `InputComponent->BindAxis()` | Enhanced Input `BindAxis()` | 新输入系统 |
| `PlayerController->GetInputAxisValue()` | Enhanced Input Action Values | 新输入系统 |

**迁移方式：** 安装 Enhanced Input 插件，创建 Input Actions 和 Input Mapping Contexts。

---

## 渲染

| 已废弃 | 替代方案 | 说明 |
|------------|-------------|-------|
| Legacy material nodes | Substrate material nodes | Substrate 在 5.7 正式生产可用 |
| Forward shading（默认） | Deferred + Lumen | Lumen 在 UE5 中为默认 |
| Old lighting workflow | Lumen Global Illumination | 实时 GI |

---

## 世界构建

| 已废弃 | 替代方案 | 说明 |
|------------|-------------|-------|
| UE4 World Composition | World Partition（UE5） | 大世界流送 |
| Level Streaming Volumes | World Partition Data Layers | 更好的关卡流送 |

---

## 动画

| 已废弃 | 替代方案 | 说明 |
|------------|-------------|-------|
| Old animation retargeting | IK Rig + IK Retargeter | UE5 重定向系统 |
| Legacy control rig | Control Rig 2.0 | 生产级绑骨工具 |

---

## 游戏机制

| 已废弃 | 替代方案 | 说明 |
|------------|-------------|-------|
| `UGameplayStatics::LoadStreamLevel()` | World Partition streaming | 使用 Data Layers |
| Hardcoded input bindings | Enhanced Input system | 可重绑定，模块化输入 |

---

## Niagara（VFX）

| 已废弃 | 替代方案 | 说明 |
|------------|-------------|-------|
| Cascade particle system | Niagara | Cascade 已完全废弃 |

---

## 音频

| 已废弃 | 替代方案 | 说明 |
|------------|-------------|-------|
| Old audio mixer | MetaSounds | 程序化音频系统 |
| Sound Cue（复杂逻辑） | MetaSounds | 更强大，基于节点 |

---

## 网络

| 已废弃 | 替代方案 | 说明 |
|------------|-------------|-------|
| `DOREPLIFETIME()`（基本） | `DOREPLIFETIME_CONDITION()` | 条件复制以优化性能 |

---

## C++ 脚本

| 已废弃 | 替代方案 | 说明 |
|------------|-------------|-------|
| `TSharedPtr<T>` 用于 UObjects | `TObjectPtr<T>` | UE5 类型安全指针 |
| Manual RTTI checks | `Cast<T>()` / `IsA<T>()` | 类型安全转换 |

---

## 快速迁移模式

### 输入示例
```cpp
// ❌ 已废弃
void AMyCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent) {
    PlayerInputComponent->BindAction("Jump", IE_Pressed, this, &ACharacter::Jump);
}

// ✅ Enhanced Input
#include "EnhancedInputComponent.h"

void AMyCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent) {
    UEnhancedInputComponent* EIC = Cast<UEnhancedInputComponent>(PlayerInputComponent);
    if (EIC) {
        EIC->BindAction(JumpAction, ETriggerEvent::Started, this, &ACharacter::Jump);
    }
}
```

### 材质示例
```cpp
// ❌ 已废弃：传统材质
// Use standard material graph (still works but not recommended)

// ✅ Substrate Material
// Enable: Project Settings > Engine > Substrate > Enable Substrate
// Use Substrate nodes in material editor
```

### World Partition 示例
```cpp
// ❌ 已废弃：关卡流送体积
// Load/unload levels manually

// ✅ World Partition
// Enable: World Settings > Enable World Partition
// Use Data Layers for streaming
```

### 粒子系统示例
```cpp
// ❌ 已废弃：Cascade
UParticleSystemComponent* PSC = CreateDefaultSubobject<UParticleSystemComponent>(TEXT("Particles"));

// ✅ Niagara
UNiagaraComponent* NiagaraComp = CreateDefaultSubobject<UNiagaraComponent>(TEXT("Niagara"));
```

### 音频示例
```cpp
// ❌ 已废弃：Sound Cue 用于复杂逻辑
// Use Sound Cue editor nodes

// ✅ MetaSounds
// Create MetaSound Source asset, use node-based audio
```

---

## 总结：UE 5.7 技术栈

| 功能 | 推荐方案（2026） | 避免使用（传统） |
|---------|------------------|----------------------|
| **输入** | Enhanced Input | Legacy Input Bindings |
| **材质** | Substrate | Legacy Material System |
| **光照** | Lumen + Megalights | Lightmaps + Limited Lights |
| **粒子** | Niagara | Cascade |
| **音频** | MetaSounds | Sound Cue（逻辑部分） |
| **世界流送** | World Partition | World Composition |
| **动画重定向** | IK Rig + Retargeter | Old Retargeting |
| **几何体** | Nanite（高精度） | Standard Static Mesh LODs |

---

**参考来源：**
- https://docs.unrealengine.com/5.7/en-US/deprecated-and-removed-features/
- https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-5-7-release-notes
