# Unreal Engine 5.7 — 破坏性变更

**最后验证日期：** 2026-02-13

本文档追踪 Unreal Engine 5.3（模型训练截止版本）与 Unreal Engine 5.7（当前版本）之间的破坏性 API 变更和行为差异。按风险等级组织。

## 高风险 — 会破坏现有代码

### Substrate 材质系统（5.7 正式生产可用）
**版本：** UE 5.5+（实验性），5.7（正式生产可用）

Substrate 以模块化、物理精确的框架取代了传统材质系统。

```cpp
// ❌ 旧版：传统材质节点（仍可用但已废弃）
// Standard material graph with Base Color, Metallic, Roughness, etc.

// ✅ 新版：Substrate 材质层
// Use Substrate nodes: Substrate Slab, Substrate Blend, etc.
// Modular material authoring with true physical accuracy
```

**迁移方式：** 在 `Project Settings > Engine > Substrate` 中启用 Substrate，并使用 Substrate 节点重建材质。

---

### PCG（程序化内容生成）API 大改
**版本：** UE 5.7（正式生产可用）

PCG 框架在达到正式生产可用状态的同时，也带来了重大 API 变更。

```cpp
// ❌ 旧版：实验性 PCG API（5.7 之前）
// Old node types, unstable API

// ✅ 新版：生产级 PCG API（5.7+）
// Use FPCGContext, IPCGElement, new node types
// Stable API, production-ready workflow
```

**迁移方式：** 按照 5.7 文档中的 PCG 迁移指南操作。使用实验性 PCG 的代码需进行大量重构。

---

### Megalights 渲染系统
**版本：** UE 5.5+

全新光照系统支持数百万个动态光源。

```cpp
// ❌ 旧版：有限动态光源（聚类前向着色）
// Max ~100-200 dynamic lights before performance degrades

// ✅ 新版：Megalights（5.5+）
// Millions of dynamic lights with minimal performance cost
// Enable: Project Settings > Engine > Rendering > Megalights
```

**迁移方式：** 无需代码更改，但光照行为可能有所不同。启用后需测试场景。

---

## 中风险 — 行为变更

### Enhanced Input System（现为默认）
**版本：** UE 5.1+（推荐），5.7（默认）

Enhanced Input 现为默认输入系统。

```cpp
// ❌ 旧版：传统输入绑定（已废弃）
InputComponent->BindAction("Jump", IE_Pressed, this, &ACharacter::Jump);

// ✅ 新版：Enhanced Input
SetupPlayerInputComponent(UInputComponent* PlayerInputComponent) {
    UEnhancedInputComponent* EIC = Cast<UEnhancedInputComponent>(PlayerInputComponent);
    EIC->BindAction(JumpAction, ETriggerEvent::Started, this, &ACharacter::Jump);
}
```

**迁移方式：** 将传统输入绑定替换为 Enhanced Input 操作。

---

### Nanite 默认启用
**版本：** UE 5.0+（可选），5.7（推荐）

Nanite 虚拟化几何体现在是静态网格的推荐工作流程。

```cpp
// Enable Nanite on static mesh:
// Static Mesh Editor > Details > Nanite Settings > Enable Nanite Support
```

**迁移方式：** 将高精度网格转换为 Nanite，并在目标平台上测试性能。

---

## 低风险 — 废弃功能（仍可用）

### 传统材质系统
**状态：** 已废弃但仍支持
**替代方案：** Substrate 材质系统

传统材质仍可使用，但建议新项目使用 Substrate。

---

### 旧版 World Partition（UE4 风格）
**状态：** 已废弃
**替代方案：** World Partition（UE5+）

大型世界请使用 UE5 的 World Partition 系统。

---

## 平台特定的破坏性变更

### Windows
- **UE 5.7**：DirectX 12 现为默认（旧版为 DX11）
- 更新着色器以兼容 DX12

### macOS
- **UE 5.5+**：要求 Metal 3（最低 macOS 13）

### 移动端
- **UE 5.7**：最低 Android API 级别提升至 26（Android 8.0）
- 最低 iOS 部署目标提升至 iOS 14

---

## 迁移检查清单

从 UE 5.3 升级到 UE 5.7 时：

- [ ] 审视 Substrate 材质（如准备就绪则转换）
- [ ] 审查 PCG 使用情况（若使用实验性版本则更新到生产 API）
- [ ] 测试 Megalights 性能（启用并基准测试）
- [ ] 将传统输入迁移到 Enhanced Input
- [ ] 将高精度网格转换为 Nanite
- [ ] 更新着色器以兼容 DX12（Windows）或 Metal 3（macOS）
- [ ] 验证最低平台版本（Android 8.0，iOS 14）
- [ ] 在目标硬件上测试 Lumen 和 Nanite 性能

---

**参考来源：**
- https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-5-7-release-notes
- https://dev.epicgames.com/documentation/en-us/unreal-engine/upgrading-projects-to-newer-versions-of-unreal-engine
