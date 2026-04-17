# Unreal Engine 5.7 — 可选插件与系统

**最后验证日期：** 2026-02-13

本文档索引了 Unreal Engine 5.7 中可用的**可选插件和系统**。
这些**不是**核心引擎的一部分，但常用于特定类型的游戏。

---

## 如何使用本指南

**✅ 有详细文档** - 见 `plugins/` 目录下的完整指南
**🟡 仅简要概述** - 链接至官方文档，使用 WebSearch 获取详情
**⚠️ 实验性** - 未来版本可能有破坏性变更
**📦 需要插件** - 需在 `Edit > Plugins` 中启用

---

## 生产可用的系统（有详细文档）

### ✅ Gameplay Ability System（GAS）
- **用途：** 模块化技能系统（技能、属性、效果、冷却、资源消耗）
- **适用场景：** RPG、MOBA、带技能的射击游戏、任何基于技能的游戏机制
- **知识差距：** GAS 自 UE4 起已稳定，UE5 改进内容超出截止日期
- **状态：** 生产可用
- **插件：** `GameplayAbilities`（内置，在 Plugins 中启用）
- **详细文档：** [plugins/gameplay-ability-system.md](plugins/gameplay-ability-system.md)
- **官方文档：** https://docs.unrealengine.com/5.7/en-US/gameplay-ability-system-for-unreal-engine/

---

### ✅ CommonUI
- **用途：** 跨平台 UI 框架（自动处理游戏手柄/鼠标/触摸输入路由）
- **适用场景：** 多平台游戏（主机 + PC）、输入无关的 UI
- **知识差距：** UE5+ 生产可用，重大改进超出截止日期
- **状态：** 生产可用
- **插件：** `CommonUI`（内置，在 Plugins 中启用）
- **详细文档：** [plugins/common-ui.md](plugins/common-ui.md)
- **官方文档：** https://docs.unrealengine.com/5.7/en-US/commonui-plugin-for-advanced-user-interfaces-in-unreal-engine/

---

### ✅ Gameplay Camera System
- **用途：** 模块化摄像机管理（摄像机模式、混合、上下文感知摄像机）
- **适用场景：** 需要动态摄像机行为的游戏（第三人称、瞄准、载具）
- **知识差距：** UE 5.5 新增，完全超出截止日期
- **状态：** ⚠️ 实验性（UE 5.5-5.7）
- **插件：** `GameplayCameras`（内置，在 Plugins 中启用）
- **详细文档：** [plugins/gameplay-camera-system.md](plugins/gameplay-camera-system.md)
- **官方文档：** https://docs.unrealengine.com/5.7/en-US/gameplay-cameras-in-unreal-engine/

---

### ✅ PCG（程序化内容生成）
- **用途：** 基于节点的程序化世界生成（植被、道具、地形细节）
- **适用场景：** 开放世界、程序化关卡、大规模环境填充
- **知识差距：** UE 5.0-5.6 实验性，5.7 生产可用
- **状态：** 生产可用（截至 UE 5.7）
- **插件：** `PCG`（内置，在 Plugins 中启用）
- **详细文档：** [plugins/pcg.md](plugins/pcg.md)
- **官方文档：** https://docs.unrealengine.com/5.7/en-US/procedural-content-generation-in-unreal-engine/

---

## 其他生产可用插件（简要概述）

### 🟡 Mass Entity
- **用途：** 高性能 ECS，用于大规模 AI/人群（10,000+ 实体）
- **适用场景：** RTS、城市模拟器、大规模人群、大规模 AI
- **状态：** 生产可用（UE 5.1+）
- **插件：** `MassEntity`、`MassGameplay`、`MassCrowd`
- **官方文档：** https://docs.unrealengine.com/5.7/en-US/mass-entity-in-unreal-engine/

---

### 🟡 Niagara Fluids
- **用途：** GPU 流体模拟（烟雾、火、液体）
- **适用场景：** 逼真的火/烟效果、水模拟
- **状态：** 实验性 → 生产可用（UE 5.4+）
- **插件：** `NiagaraFluids`（内置）
- **官方文档：** https://docs.unrealengine.com/5.7/en-US/niagara-fluids-in-unreal-engine/

---

### 🟡 Water Plugin
- **用途：** 海洋、河流、湖泊渲染及浮力
- **适用场景：** 有水体的游戏、船只、游泳
- **状态：** 生产可用（UE 5.0+）
- **插件：** `Water`（内置）
- **官方文档：** https://docs.unrealengine.com/5.7/en-US/water-system-in-unreal-engine/

---

### 🟡 Landmass Plugin
- **用途：** 地形雕刻和景观编辑
- **适用场景：** 大规模地形修改、程序化景观
- **状态：** 生产可用
- **插件：** `Landmass`（内置）
- **官方文档：** https://docs.unrealengine.com/5.7/en-US/landmass-plugin-in-unreal-engine/

---

### 🟡 Chaos Destruction
- **用途：** 实时破碎和破坏
- **适用场景：** 可破坏环境（墙壁、建筑物、物体）
- **状态：** 生产可用（UE 5.0+）
- **插件：** `ChaosDestruction`（内置）
- **官方文档：** https://docs.unrealengine.com/5.7/en-US/destruction-in-unreal-engine/

---

### 🟡 Chaos Vehicle
- **用途：** 高级载具物理（轮式载具、悬挂）
- **适用场景：** 赛车游戏、载具密集型游戏机制
- **状态：** 生产可用（替代 PhysX Vehicles）
- **插件：** `ChaosVehicles`（内置）
- **官方文档：** https://docs.unrealengine.com/5.7/en-US/chaos-vehicles-overview-in-unreal-engine/

---

### 🟡 Geometry Scripting
- **用途：** 运行时程序化网格生成和编辑
- **适用场景：** 动态网格创建、程序化建模
- **状态：** 生产可用（UE 5.1+）
- **插件：** `GeometryScripting`（内置）
- **官方文档：** https://docs.unrealengine.com/5.7/en-US/geometry-scripting-in-unreal-engine/

---

### 🟡 Motion Design Tools
- **用途：** 动态图形、程序化动画、关键帧动画
- **适用场景：** UI 动画、程序化运动、关键帧序列
- **状态：** 实验性 → 生产可用（UE 5.4+）
- **插件：** `MotionDesign`（内置）
- **官方文档：** https://docs.unrealengine.com/5.7/en-US/motion-design-mode-in-unreal-engine/

---

## 实验性插件（谨慎使用）

### ⚠️ AI Assistant（UE 5.7+）
- **用途：** 编辑器内 AI 指导和帮助
- **状态：** 实验性
- **插件：** 在 UE 5.7 设置中启用
- **官方文档：** UE 5.7 发布公告

---

### ⚠️ OpenXR（VR/AR）
- **用途：** 跨平台 VR/AR 支持
- **适用场景：** VR/AR 游戏
- **状态：** VR 生产可用，AR 实验性
- **插件：** `OpenXR`（内置）
- **官方文档：** https://docs.unrealengine.com/5.7/en-US/openxr-in-unreal-engine/

---

### ⚠️ Online Subsystem（EOS、Steam 等）
- **用途：** 平台无关的在线服务（匹配、好友、成就）
- **适用场景：** 带在线功能的多人游戏
- **状态：** 生产可用
- **插件：** `OnlineSubsystem`、`OnlineSubsystemEOS`、`OnlineSubsystemSteam`
- **官方文档：** https://docs.unrealengine.com/5.7/en-US/online-subsystem-in-unreal-engine/

---

## 已废弃插件（新项目请勿使用）

### ❌ PhysX Vehicles
- **已废弃：** 请改用 Chaos Vehicles
- **状态：** 传统，不推荐

---

### ❌ 旧版 Replication Graph
- **已废弃：** 已由 Iris 替代（UE 5.1+）
- **状态：** 请使用 Iris 实现现代网络

---

## 按需 WebSearch 策略

对于上述未列出的插件，当用户询问时使用以下方法：

1. **WebSearch** 最新文档：`"Unreal Engine 5.7 [插件名称]"`
2. 验证插件是否为：
   - 超出截止日期（超过 2025 年 5 月的训练数据）
   - 实验性 vs 生产可用
   - 仍在 UE 5.7 中受支持
3. 可选择将结果缓存到 `plugins/[插件名称].md` 供将来参考

---

## 快速决策指南

**需要技能/ buffs** → **Gameplay Ability System（GAS）**
**需要跨平台 UI（主机 + PC）** → **CommonUI**
**需要动态摄像机** → **Gameplay Camera System**
**需要程序化世界** → **PCG**
**需要大规模人群（1000+ AI）** → **Mass Entity**
**需要可破坏环境** → **Chaos Destruction**
**需要载具** → **Chaos Vehicles**
**需要水/海洋** → **Water Plugin**
**需要 VR/AR** → **OpenXR**

---

**最后更新：** 2026-02-13
**引擎版本：** Unreal Engine 5.7
**LLM 知识截止日期：** 2025 年 5 月
