# Unreal Engine 5.7 — Gameplay Ability System (GAS)

**Last verified:** 2026-02-13
**Status:** Production-Ready
**Plugin:** `GameplayAbilities` (built-in, enable in Plugins)

---

## 概述

**Gameplay Ability System (GAS)** 是一个模块化框架，用于构建能力、属性、效果和游戏机制。它是 RPG、MOBAs、带技能的第一人称射击游戏以及任何具有复杂技能系统的游戏的标准选择。

**使用 GAS 的场景：**
- 角色能力（法术、技能、攻击）
- 属性（生命值、法力值、耐力、属性点）
- Buff/Debuff（临时效果）
- 冷却和消耗
- 伤害计算
- 支持多人的能力复制

---

## 核心概念

### 1. **Ability System Component (ASC)**
- 管理能力、属性和效果的主要组件
- 添加到 Character 或 PlayerState 上

### 2. **Gameplay Abilities**
- 独立的技能/动作（火球、治疗、冲刺等）
- 可激活、执行（消耗/冷却）、也可取消

### 3. **Attributes & Attribute Sets**
- 可被修改的属性（生命值、法力值、耐力、力量等）
- 存储在 Attribute Sets 中

### 4. **Gameplay Effects**
- 修改属性（伤害、治疗、buff、debuff）
- 可以是立即生效、持续时间或永久生效

### 5. **Gameplay Tags**
- 用于能力逻辑的分层标签（例如 `Ability.Attack.Melee`、`Status.Stunned`）

---

## 设置

### 1. 启用 Plugin

`Edit > Plugins > Gameplay Abilities > Enabled > Restart`

### 2. 添加 Ability System Component

```cpp
#include "AbilitySystemComponent.h"
#include "AttributeSet.h"

UCLASS()
class AMyCharacter : public ACharacter {
    GENERATED_BODY()

public:
    AMyCharacter() {
        // 创建 ASC
        AbilitySystemComponent = CreateDefaultSubobject<UAbilitySystemComponent>(TEXT("AbilitySystem"));
        AbilitySystemComponent->SetIsReplicated(true);
        AbilitySystemComponent->SetReplicationMode(EGameplayEffectReplicationMode::Mixed);

        // 创建 Attribute Set
        AttributeSet = CreateDefaultSubobject<UMyAttributeSet>(TEXT("AttributeSet"));
    }

protected:
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Abilities")
    TObjectPtr<UAbilitySystemComponent> AbilitySystemComponent;

    UPROPERTY()
    TObjectPtr<const UAttributeSet> AttributeSet;
};
```

### 3. 初始化 ASC（多人模式下尤为重要）

```cpp
void AMyCharacter::PossessedBy(AController* NewController) {
    Super::PossessedBy(NewController);

    // Server: 初始化 ASC
    if (AbilitySystemComponent) {
        AbilitySystemComponent->InitAbilityActorInfo(this, this);
        GiveDefaultAbilities();
    }
}

void AMyCharacter::OnRep_PlayerState() {
    Super::OnRep_PlayerState();

    // Client: 初始化 ASC
    if (AbilitySystemComponent) {
        AbilitySystemComponent->InitAbilityActorInfo(this, this);
    }
}
```

---

## Attributes & Attribute Sets

### 创建 Attribute Set

```cpp
#include "AttributeSet.h"
#include "AbilitySystemComponent.h"

UCLASS()
class UMyAttributeSet : public UAttributeSet {
    GENERATED_BODY()

public:
    UMyAttributeSet();

    // Health
    UPROPERTY(BlueprintReadOnly, Category = "Attributes", ReplicatedUsing = OnRep_Health)
    FGameplayAttributeData Health;
    ATTRIBUTE_ACCESSORS(UMyAttributeSet, Health)

    UPROPERTY(BlueprintReadOnly, Category = "Attributes", ReplicatedUsing = OnRep_MaxHealth)
    FGameplayAttributeData MaxHealth;
    ATTRIBUTE_ACCESSORS(UMyAttributeSet, MaxHealth)

    // Mana
    UPROPERTY(BlueprintReadOnly, Category = "Attributes", ReplicatedUsing = OnRep_Mana)
    FGameplayAttributeData Mana;
    ATTRIBUTE_ACCESSORS(UMyAttributeSet, Mana)

    virtual void GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const override;

protected:
    UFUNCTION()
    virtual void OnRep_Health(const FGameplayAttributeData& OldHealth);

    UFUNCTION()
    virtual void OnRep_MaxHealth(const FGameplayAttributeData& OldMaxHealth);

    UFUNCTION()
    virtual void OnRep_Mana(const FGameplayAttributeData& OldMana);
};
```

### 实现 Attribute Set

```cpp
#include "Net/UnrealNetwork.h"

UMyAttributeSet::UMyAttributeSet() {
    // 默认值
    Health = 100.0f;
    MaxHealth = 100.0f;
    Mana = 50.0f;
}

void UMyAttributeSet::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const {
    Super::GetLifetimeReplicatedProps(OutLifetimeProps);

    DOREPLIFETIME_CONDITION_NOTIFY(UMyAttributeSet, Health, COND_None, REPNOTIFY_Always);
    DOREPLIFETIME_CONDITION_NOTIFY(UMyAttributeSet, MaxHealth, COND_None, REPNOTIFY_Always);
    DOREPLIFETIME_CONDITION_NOTIFY(UMyAttributeSet, Mana, COND_None, REPNOTIFY_Always);
}

void UMyAttributeSet::OnRep_Health(const FGameplayAttributeData& OldHealth) {
    GAMEPLAYATTRIBUTE_REPNOTIFY(UMyAttributeSet, Health, OldHealth);
}

// 其他 OnRep 函数实现类似...
```

---

## Gameplay Abilities

### 创建 Gameplay Ability

```cpp
#include "Abilities/GameplayAbility.h"

UCLASS()
class UGA_Fireball : public UGameplayAbility {
    GENERATED_BODY()

public:
    UGA_Fireball() {
        // 能力配置
        InstancingPolicy = EGameplayAbilityInstancingPolicy::InstancedPerActor;
        NetExecutionPolicy = EGameplayAbilityNetExecutionPolicy::ServerInitiated;

        // 标签
        AbilityTags.AddTag(FGameplayTag::RequestGameplayTag(FName("Ability.Attack.Fireball")));
    }

    virtual void ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo,
        const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData) override {

        if (!CommitAbility(Handle, ActorInfo, ActivationInfo)) {
            // 提交失败（法力不足、冷却中等等）
            EndAbility(Handle, ActorInfo, ActivationInfo, true, true);
            return;
        }

        // 生成火球投射物
        SpawnFireball();

        // 结束能力
        EndAbility(Handle, ActorInfo, ActivationInfo, true, false);
    }

    void SpawnFireball() {
        // 生成火球逻辑
    }
};
```

### 授予角色能力

```cpp
void AMyCharacter::GiveDefaultAbilities() {
    if (!HasAuthority() || !AbilitySystemComponent) return;

    // 授予能力
    AbilitySystemComponent->GiveAbility(FGameplayAbilitySpec(UGA_Fireball::StaticClass(), 1, INDEX_NONE, this));
    AbilitySystemComponent->GiveAbility(FGameplayAbilitySpec(UGA_Heal::StaticClass(), 1, INDEX_NONE, this));
}
```

### 激活能力

```cpp
// 通过类激活
AbilitySystemComponent->TryActivateAbilityByClass(UGA_Fireball::StaticClass());

// 通过标签激活
FGameplayTagContainer TagContainer;
TagContainer.AddTag(FGameplayTag::RequestGameplayTag(FName("Ability.Attack.Fireball")));
AbilitySystemComponent->TryActivateAbilitiesByTag(TagContainer);
```

---

## Gameplay Effects

### 创建 Gameplay Effect（伤害）

```cpp
// 在 Blueprint 中创建：Content Browser > Gameplay > Gameplay Effect

// 或在 C++ 中：
UCLASS()
class UGE_Damage : public UGameplayEffect {
    GENERATED_BODY()

public:
    UGE_Damage() {
        // 立即伤害
        DurationPolicy = EGameplayEffectDurationType::Instant;

        // 修改器：减少生命值
        FGameplayModifierInfo ModifierInfo;
        ModifierInfo.Attribute = UMyAttributeSet::GetHealthAttribute();
        ModifierInfo.ModifierOp = EGameplayModOp::Additive;
        ModifierInfo.ModifierMagnitude = FScalableFloat(-25.0f); // -25 生命值

        Modifiers.Add(ModifierInfo);
    }
};
```

### 应用 Gameplay Effect

```cpp
// 对目标应用伤害
if (UAbilitySystemComponent* TargetASC = UAbilitySystemBlueprintLibrary::GetAbilitySystemComponent(Target)) {
    FGameplayEffectContextHandle EffectContext = AbilitySystemComponent->MakeEffectContext();
    EffectContext.AddSourceObject(this);

    FGameplayEffectSpecHandle SpecHandle = AbilitySystemComponent->MakeOutgoingSpec(
        UGE_Damage::StaticClass(), 1, EffectContext);

    if (SpecHandle.IsValid()) {
        AbilitySystemComponent->ApplyGameplayEffectSpecToTarget(*SpecHandle.Data.Get(), TargetASC);
    }
}
```

---

## Gameplay Tags

### 定义标签

`Project Settings > Project > Gameplay Tags > Gameplay Tag List`

示例层级结构：
```
Ability
  ├─ Ability.Attack
  │   ├─ Ability.Attack.Melee
  │   └─ Ability.Attack.Ranged
  ├─ Ability.Defend
  └─ Ability.Utility

Status
  ├─ Status.Stunned
  ├─ Status.Invulnerable
  └─ Status.Silenced
```

### 在能力中使用标签

```cpp
UCLASS()
class UGA_MeleeAttack : public UGameplayAbility {
    GENERATED_BODY()

public:
    UGA_MeleeAttack() {
        // 此能力拥有这些标签
        AbilityTags.AddTag(FGameplayTag::RequestGameplayTag(FName("Ability.Attack.Melee")));

        // 激活时阻止这些标签的能力
        BlockAbilitiesWithTag.AddTag(FGameplayTag::RequestGameplayTag(FName("Ability.Attack")));

        // 激活时取消这些能力
        CancelAbilitiesWithTag.AddTag(FGameplayTag::RequestGameplayTag(FName("Ability.Defend")));

        // 如果目标拥有这些标签则无法激活
        ActivationBlockedTags.AddTag(FGameplayTag::RequestGameplayTag(FName("Status.Stunned")));
    }
};
```

---

## 冷却与消耗

### 添加冷却

```cpp
// 在 Ability Blueprint 或 C++ 中：
// 创建一个 Duration = 冷却时间的 Gameplay Effect
// 赋值给 Ability > Cooldown Gameplay Effect Class
```

### 添加消耗（法力）

```cpp
// 创建一个减少法力的 Gameplay Effect
// 赋值给 Ability > Cost Gameplay Effect Class
```

---

## 常见模式

### 获取当前属性值

```cpp
float CurrentHealth = AbilitySystemComponent->GetNumericAttribute(UMyAttributeSet::GetHealthAttribute());
```

### 监听属性变化

```cpp
AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(UMyAttributeSet::GetHealthAttribute())
    .AddUObject(this, &AMyCharacter::OnHealthChanged);

void AMyCharacter::OnHealthChanged(const FOnAttributeChangeData& Data) {
    UE_LOG(LogTemp, Warning, TEXT("Health: %f"), Data.NewValue);
}
```

---

## Sources
- https://docs.unrealengine.com/5.7/en-US/gameplay-ability-system-for-unreal-engine/
- https://github.com/tranek/GASDocumentation (community guide)
