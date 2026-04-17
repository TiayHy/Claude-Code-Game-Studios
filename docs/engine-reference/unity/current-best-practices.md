# Unity 6.3 LTS — 当前最佳实践

**最后验证：** 2026-02-13

这些是 Unity 6.3 LTS 的现代模式，可能不在 LLM 的训练数据中。
截至 Unity 6.3 LTS 的生产级推荐建议。

---

## 项目设置

### 生产环境使用 Unity 6.3 LTS
- **技术分支**（6.4+）：最新功能，较不稳定
- **LTS**（6.3）：生产就绪，2 年支持（至 2027 年 12 月）

### 选择正确的渲染管线
- **URP（Universal）**：移动设备、跨平台、性能良好 ✅ 大多数游戏推荐
- **HDRP（High Definition）**：高端 PC/主机、写实级画质
- **内置管线**：已弃用，新项目避免使用

---

## 脚本

### 使用 C# 9+ 功能（Unity 6 支持 C# 9）

```csharp
// ✅ Record 类型用于数据
public record PlayerData(string Name, int Level, float Health);

// ✅ 仅初始化属性
public class Config {
    public string GameMode { get; init; }
}

// ✅ 模式匹配
var result = enemy switch {
    Boss boss => boss.Enrage(),
    Minion minion => minion.Flee(),
    _ => null
};
```

### Async/Await 用于资源加载

```csharp
// ✅ 现代异步模式
public async Task<GameObject> LoadEnemyAsync(string key) {
    var handle = Addressables.LoadAssetAsync<GameObject>(key);
    return await handle.Task;
}
```

### 使用 Source Generator 进行序列化（Unity 6+）

```csharp
// ✅ Source 生成的序列化（更快，减少反射）
[GenerateSerializer]
public partial struct PlayerStats : IComponentData {
    public int Health;
    public int Mana;
}
```

---

## DOTS/ECS（Unity 6.3 LTS 生产就绪）

### 使用 ISystem（而非 ComponentSystem）

```csharp
// ✅ 现代非托管 ISystem（Burst 兼容）
public partial struct MovementSystem : ISystem {
    public void OnCreate(ref SystemState state) { }

    public void OnUpdate(ref SystemState state) {
        foreach (var (transform, speed) in
            SystemAPI.Query<RefRW<LocalTransform>, RefRO<MoveSpeed>>()) {
            transform.ValueRW.Position += speed.ValueRO.Value * SystemAPI.Time.DeltaTime;
        }
    }
}
```

### 使用 IJobEntity 进行并行作业

```csharp
// ✅ IJobEntity（替代 IJobForEach）
[BurstCompile]
public partial struct DamageJob : IJobEntity {
    public float DeltaTime;

    void Execute(ref Health health, in DamageOverTime dot) {
        health.Value -= dot.DamagePerSecond * DeltaTime;
    }
}

// 调度它
var job = new DamageJob { DeltaTime = SystemAPI.Time.DeltaTime };
job.ScheduleParallel();
```

---

## 输入

### 使用 Input System 包（非旧版 Input）

```csharp
// ✅ Input Actions（可重绑定、跨平台）
using UnityEngine.InputSystem;

public class PlayerInput : MonoBehaviour {
    private PlayerControls controls;

    void Awake() {
        controls = new PlayerControls();
        controls.Gameplay.Jump.performed += ctx => Jump();
    }

    void OnEnable() => controls.Enable();
    void OnDisable() => controls.Disable();
}
```

在编辑器中创建 Input Actions 资源，通过检查器生成 C# 类。

---

## UI

### 使用 UI Toolkit 用于运行时 UI（Unity 6 生产就绪）

```csharp
// ✅ UI Toolkit（替代新项目的 UGUI）
using UnityEngine.UIElements;

public class MainMenu : MonoBehaviour {
    void OnEnable() {
        var root = GetComponent<UIDocument>().rootVisualElement;

        var playButton = root.Q<Button>("play-button");
        playButton.clicked += StartGame;

        var scoreLabel = root.Q<Label>("score");
        scoreLabel.text = $"最高分：{PlayerPrefs.GetInt("HighScore")}";
    }
}
```

**UXML**（UI 结构）+ **USS**（样式）= 类似 HTML/CSS 的工作流。

---

## 资源管理

### 使用 Addressables（非 Resources）

```csharp
// ✅ Addressables（异步、内存高效）
using UnityEngine.AddressableAssets;

public async Task SpawnEnemyAsync(string enemyKey) {
    var handle = Addressables.InstantiateAsync(enemyKey);
    var enemy = await handle.Task;

    // 清理：销毁时释放
    Addressables.ReleaseInstance(enemy);
}
```

**优势：** 异步加载、远程内容交付、更好的内存控制。

---

## 渲染

### 使用 RenderGraph API 用于自定义通道（URP/HDRP）

```csharp
// ✅ RenderGraph API（Unity 6+）
public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData) {
    using (var builder = renderGraph.AddRasterRenderPass<PassData>("My Pass", out var passData)) {
        // 设置通道
        builder.SetRenderFunc((PassData data, RasterGraphContext context) => {
            // 执行命令
        });
    }
}
```

**替代：** 旧的 `CommandBuffer.Execute()` 模式。

---

## 性能

### 使用 Burst Compiler + Jobs 系统

```csharp
// ✅ Burst 编译的作业（巨大的性能提升）
[BurstCompile]
struct ParticleUpdateJob : IJobParallelFor {
    public NativeArray<float3> Positions;
    public NativeArray<float3> Velocities;
    public float DeltaTime;

    public void Execute(int index) {
        Positions[index] += Velocities[index] * DeltaTime;
    }
}

// 调度
var job = new ParticleUpdateJob {
    Positions = positions,
    Velocities = velocities,
    DeltaTime = Time.deltaTime
};
job.Schedule(positions.Length, 64).Complete();
```

**比等效 C# 代码快 20-100 倍。**

---

### 使用 GPU Instancing 处理重复对象

```csharp
// ✅ GPU Instancing（数千个对象，最小化 draw call）
Graphics.RenderMeshInstanced(
    new RenderParams(material),
    mesh,
    0,
    matrices // NativeArray<Matrix4x4>
);
```

---

## 内存管理

### 使用 NativeContainer（作业中不用托管数组）

```csharp
// ✅ NativeArray（无 GC，Burst 兼容）
NativeArray<int> data = new NativeArray<int>(1000, Allocator.TempJob);
// ... 在作业中使用
data.Dispose(); // 需要手动清理

// ✅ 或使用 using 语句
using var data = new NativeArray<int>(1000, Allocator.TempJob);
// 自动释放
```

---

## 多人游戏

### 使用 Netcode for GameObjects（官方）

```csharp
// ✅ Unity 官方 netcode
using Unity.Netcode;

public class Player : NetworkBehaviour {
    private NetworkVariable<int> health = new NetworkVariable<int>(100);

    [ServerRpc]
    public void TakeDamageServerRpc(int damage) {
        health.Value -= damage;
    }
}
```

**替代：** UNet（已弃用）、MLAPI（更名为 Netcode for GameObjects）。

---

## 测试

### 使用 Unity Test Framework（基于 NUnit）

```csharp
// ✅ Play Mode 测试
[UnityTest]
public IEnumerator Player_TakesDamage_HealthDecreases() {
    var player = new GameObject().AddComponent<Player>();
    player.Health = 100;

    player.TakeDamage(25);
    yield return null; // 等待一帧

    Assert.AreEqual(75, player.Health);
}
```

---

## 调试

### 使用日志最佳实践

```csharp
// ✅ 结构化日志（Unity 6+）
using UnityEngine;

Debug.Log($"玩家 {playerName} 得分 {score} 分");

// ✅ 条件编译用于调试代码
#if UNITY_EDITOR || DEVELOPMENT_BUILD
    Debug.DrawRay(transform.position, direction, Color.red);
#endif
```

---

## 总结：Unity 6 技术栈

| 功能 | 使用这个（2026） | 避免这个（旧版） |
|------|-------------------|------------------|
| **输入** | Input System 包 | `Input` 类 |
| **UI** | UI Toolkit | UGUI（Canvas） |
| **ECS** | ISystem + IJobEntity | ComponentSystem |
| **渲染** | URP + RenderGraph | 内置管线 |
| **资源** | Addressables | Resources |
| **作业** | Burst + IJobParallelFor | 繁重工作时用协程 |
| **多人** | Netcode for GameObjects | UNet |

---

**来源：**
- https://docs.unity3d.com/6000.0/Documentation/Manual/BestPracticeGuides.html
- https://docs.unity3d.com/Packages/com.unity.entities@1.3/manual/index.html
- https://docs.unity3d.com/Packages/com.unity.inputsystem@1.11/manual/index.html
