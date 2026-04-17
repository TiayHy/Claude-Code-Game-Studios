# Unreal Engine 5.7 — 当前最佳实践

**最后验证日期：** 2026-02-13

本文档收录了 UE5 现代开发模式，其中部分内容可能未出现在 LLM 训练数据中。
以下是截至 UE 5.7 的生产可用推荐方案。

---

## 项目设置

### 新项目使用 UE 5.7
- 最新功能：Megalights、正式生产可用的 Substrate 和 PCG
- 更好的性能和稳定性

### 选择合适的渲染功能
- **Lumen**：实时全局光照（大多数项目推荐）
- **Nanite**：高精度网格的虚拟化几何体（细节丰富环境推荐）
- **Megalights**：数百万动态光源（复杂光照推荐）
- **Substrate**：模块化材质系统（新项目推荐）

---

## C++ 编码

### 使用现代 C++ 特性（UE5.7 支持 C++20）

```cpp
// ✅ 使用 TObjectPtr<T>（UE5 类型安全指针）
UPROPERTY()
TObjectPtr<UStaticMeshComponent> MeshComp;

// ✅ 结构化绑定
if (auto [bSuccess, Value] = TryGetValue(); bSuccess) {
    // Use Value
}

// ✅ Concepts 和约束（C++20）
template<typename T>
concept Damageable = requires(T t, float damage) {
    { t.TakeDamage(damage) } -> std::same_as<void>;
};
```

### 使用 UPROPERTY() 管理垃圾回收

```cpp
// ✅ UPROPERTY 确保 GC 不会删除此对象
UPROPERTY()
TObjectPtr<AActor> MyActor;

// ❌ 裸指针可能变为悬空指针
AActor* MyActor; // 危险！可能被垃圾回收
```

### 使用 UFUNCTION() 暴露给 Blueprint

```cpp
// ✅ 可从 Blueprint 调用
UFUNCTION(BlueprintCallable, Category="Combat")
void TakeDamage(float Damage);

// ✅ 可在 Blueprint 中实现
UFUNCTION(BlueprintImplementableEvent, Category="Combat")
void OnDeath();
```

---

## Blueprint 最佳实践

### 何时使用 Blueprint vs C++

- **C++**：核心游戏系统、性能关键代码、低层引擎交互
- **Blueprint**：快速原型、内容创建、数据驱动逻辑、设计师工作流

### Blueprint 性能提示

```cpp
// ✅ 尽量少用 Event Tick（开销较大）
// 优先使用定时器或事件

// ✅ 使用 Blueprint Nativization（Blueprints → C++）
// Project Settings > Packaging > Blueprint Nativization

// ✅ 缓存频繁访问的组件
// 不要在每帧都调用 GetComponent
```

---

## 渲染（UE 5.7）

### 使用 Lumen 实现全局光照

```cpp
// 启用：Project Settings > Engine > Rendering > Dynamic Global Illumination Method = Lumen
// Real-time GI, no lightmap baking needed (RECOMMENDED)
```

### 使用 Nanite 处理高精度网格

```cpp
// 启用：在 Static Mesh 上，Details > Nanite Settings > Enable Nanite Support
// Automatically LODs millions of triangles (RECOMMENDED for detailed meshes)
```

### 使用 Megalights 处理复杂光照（UE 5.5+）

```cpp
// 启用：Project Settings > Engine > Rendering > Megalights = Enabled
// Supports millions of dynamic lights with minimal cost
```

### 使用 Substrate 材质（5.7 正式生产可用）

```cpp
// 启用：Project Settings > Engine > Substrate > Enable Substrate
// Modular, physically accurate materials (RECOMMENDED for new projects)
```

---

## Enhanced Input System

### 配置 Enhanced Input

```cpp
// 1. 创建 Input Action（IA_Jump）
// 2. 创建 Input Mapping Context（IMC_Default）
// 3. 添加映射：IA_Jump → Space Bar

// C++ 配置：
#include "EnhancedInputComponent.h"
#include "EnhancedInputSubsystems.h"

void AMyCharacter::BeginPlay() {
    Super::BeginPlay();

    if (APlayerController* PC = Cast<APlayerController>(GetController())) {
        if (UEnhancedInputLocalPlayerSubsystem* Subsystem =
            ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(PC->GetLocalPlayer())) {
            Subsystem->AddMappingContext(DefaultMappingContext, 0);
        }
    }
}

void AMyCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent) {
    UEnhancedInputComponent* EIC = Cast<UEnhancedInputComponent>(PlayerInputComponent);
    EIC->BindAction(JumpAction, ETriggerEvent::Started, this, &ACharacter::Jump);
    EIC->BindAction(MoveAction, ETriggerEvent::Triggered, this, &AMyCharacter::Move);
}

void AMyCharacter::Move(const FInputActionValue& Value) {
    FVector2D MoveVector = Value.Get<FVector2D>();
    AddMovementInput(GetActorForwardVector(), MoveVector.Y);
    AddMovementInput(GetActorRightVector(), MoveVector.X);
}
```

---

## Gameplay Ability System（GAS）

### 使用 GAS 处理复杂游戏机制

```cpp
// ✅ 适用场景：技能、buff、伤害计算、冷却
// Modular, scalable, multiplayer-ready

// 安装：启用 "Gameplay Abilities" 插件

// 示例技能：
UCLASS()
class UGA_Fireball : public UGameplayAbility {
    GENERATED_BODY()

public:
    virtual void ActivateAbility(...) override {
        // Ability logic
        SpawnFireball();
        CommitAbility(); // Commit cost/cooldown
    }
};
```

---

## World Partition（大型世界）

### 使用 World Partition 处理开放世界

```cpp
// 启用：World Settings > Enable World Partition
// Automatically streams world cells based on player location

// Data Layers：组织内容（如 "Gameplay"、"Audio"、"Lighting"）
// Runtime Data Layers：运行时加载/卸载
```

---

## Niagara（VFX）

### 使用 Niagara（而非 Cascade）

```cpp
// 创建：Content Browser > Right Click > FX > Niagara System
// GPU-accelerated, node-based particle system (RECOMMENDED)

// 生成粒子：
UNiagaraComponent* NiagaraComp = UNiagaraFunctionLibrary::SpawnSystemAtLocation(
    GetWorld(),
    ExplosionSystem,
    GetActorLocation()
);
```

---

## MetaSounds（音频）

### 使用 MetaSounds 实现程序化音频

```cpp
// 创建：Content Browser > Right Click > Sounds > MetaSound Source
// Node-based audio, replaces Sound Cue for complex logic (RECOMMENDED)

// 播放 MetaSound：
UAudioComponent* AudioComp = UGameplayStatics::SpawnSound2D(
    GetWorld(),
    MetaSoundSource
);
```

---

## 复制（多人游戏）

### 服务端权威模式

```cpp
// ✅ 客户端发送输入，服务端验证并复制
UFUNCTION(Server, Reliable)
void Server_Move(FVector Direction);

void AMyCharacter::Server_Move_Implementation(FVector Direction) {
    // Server validates and applies movement
    AddMovementInput(Direction);
}

// ✅ 复制重要状态
UPROPERTY(Replicated)
int32 Health;

void AMyCharacter::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const {
    Super::GetLifetimeReplicatedProps(OutLifetimeProps);
    DOREPLIFETIME(AMyCharacter, Health);
}
```

---

## 性能优化

### 使用对象池

```cpp
// ✅ 复用对象而非 Spawn/Destroy
TArray<AActor*> ProjectilePool;

AActor* GetPooledProjectile() {
    for (AActor* Proj : ProjectilePool) {
        if (!Proj->IsActive()) {
            Proj->SetActive(true);
            return Proj;
        }
    }
    // Pool exhausted, spawn new
    return SpawnNewProjectile();
}
```

### 使用 Instanced Static Meshes

```cpp
// ✅ 分层实例化静态网格组件（HISM）
// Render thousands of identical meshes in one draw call
UHierarchicalInstancedStaticMeshComponent* HISM = CreateDefaultSubobject<UHierarchicalInstancedStaticMeshComponent>(TEXT("Trees"));
for (int i = 0; i < 1000; i++) {
    HISM->AddInstance(FTransform(RandomLocation));
}
```

---

## 调试

### 使用日志

```cpp
// ✅ 结构化日志
UE_LOG(LogTemp, Warning, TEXT("Player health: %d"), Health);

// 自定义日志类别
DECLARE_LOG_CATEGORY_EXTERN(LogMyGame, Log, All);
DEFINE_LOG_CATEGORY(LogMyGame);
UE_LOG(LogMyGame, Error, TEXT("Critical error!"));
```

### 使用 Visual Logger

```cpp
// ✅ 可视化调试
#include "VisualLogger/VisualLogger.h"

UE_VLOG_SEGMENT(this, LogTemp, Log, StartPos, EndPos, FColor::Red, TEXT("Raycast"));
UE_VLOG_LOCATION(this, LogTemp, Log, TargetLocation, 50.f, FColor::Green, TEXT("Target"));
```

---

## 总结：UE 5.7 推荐技术栈

| 功能 | 推荐方案（2026） | 说明 |
|---------|------------------|-------|
| **光照** | Lumen + Megalights | 实时 GI，数百万光源 |
| **几何体** | Nanite | 高精度网格，自动 LOD |
| **材质** | Substrate | 模块化，物理精确 |
| **输入** | Enhanced Input | 可重绑定，模块化 |
| **VFX** | Niagara | GPU 加速 |
| **音频** | MetaSounds | 程序化音频 |
| **世界流送** | World Partition | 大型开放世界 |
| **游戏机制** | Gameplay Ability System | 复杂技能，buff |

---

**参考来源：**
- https://docs.unrealengine.com/5.7/en-US/
- https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-5-7-release-notes
