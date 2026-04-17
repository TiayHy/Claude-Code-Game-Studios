# Unity 6.3 LTS — 可选包与系统

**最后验证：** 2026-02-13

本文档索引 Unity 6.3 LTS 中可用的**可选包和系统**。
这些不是核心引擎的一部分，但常用于特定类型的游戏。

---

## 如何使用本指南

**✅ 有详细文档** - 参见 `plugins/` 目录的综合指南
**🟡 仅简要概述** - 链接到官方文档，使用 WebSearch 获取详情
**⚠️ 预览版** - 未来版本可能有破坏性变更
**📦 需要安装包** - 通过 Package Manager 安装

---

## 生产就绪的包（有详细文档）

### ✅ Cinemachine
- **用途：** 虚拟相机系统（动态相机、过场动画、相机混合）
- **使用场景：** 第三人称游戏、过场动画、复杂相机行为
- **知识差距：** Cinemachine 3.0+（Unity 6）与 2.x 相比有重大 API 变更
- **状态：** 生产就绪
- **包：** `com.unity.cinemachine`（Package Manager）
- **详细文档：** [plugins/cinemachine.md](plugins/cinemachine.md)
- **官方文档：** https://docs.unity3d.com/Packages/com.unity.cinemachine@3.0/manual/index.html

---

### ✅ Addressables
- **用途：** 高级资源管理（异步加载、远程内容、内存控制）
- **使用场景：** 大型项目、DLC、远程内容交付
- **知识差距：** Unity 6 改进，性能更好
- **状态：** 生产就绪
- **包：** `com.unity.addressables`（Package Manager）
- **详细文档：** [plugins/addressables.md](plugins/addressables.md)
- **官方文档：** https://docs.unity3d.com/Packages/com.unity.addressables@2.0/manual/index.html

---

### ✅ DOTS / Entities（ECS）
- **用途：** 面向数据技术栈（大规模高性能 ECS）
- **使用场景：** 包含数千实体的游戏、RTS、模拟
- **知识差距：** Entities 1.3+（Unity 6）已生产就绪，与 0.x 相比重大重写
- **状态：** 生产就绪（截至 Unity 6.3 LTS）
- **包：** `com.unity.entities`（Package Manager）
- **详细文档：** [plugins/dots-entities.md](plugins/dots-entities.md)
- **官方文档：** https://docs.unity3d.com/Packages/com.unity.entities@1.3/manual/index.html

---

## 其他生产就绪的包（简要概述）

### 🟡 Input System（已覆盖）
- **用途：** 现代输入处理（可重绑定、跨平台）
- **状态：** 生产就绪（Unity 6 默认）
- **包：** `com.unity.inputsystem`
- **文档：** 参见 [modules/input.md](../modules/input.md)
- **官方文档：** https://docs.unity3d.com/Packages/com.unity.inputsystem@1.11/manual/index.html

---

### 🟡 UI Toolkit（已覆盖）
- **用途：** 现代运行时 UI（类 HTML/CSS，高性能）
- **状态：** 生产就绪（Unity 6）
- **包：** 内置
- **文档：** 参见 [modules/ui.md](../modules/ui.md)
- **官方文档：** https://docs.unity3d.com/Packages/com.unity.ui@2.0/manual/index.html

---

### 🟡 Visual Effect Graph（VFX Graph）
- **用途：** GPU 加速粒子系统（数百万粒子）
- **使用场景：** 大规模 VFX、火焰、烟雾、魔法、爆炸
- **状态：** 生产就绪
- **包：** `com.unity.visualeffectgraph`（仅 URP/HDRP）
- **官方文档：** https://docs.unity3d.com/Packages/com.unity.visualeffectgraph@17.0/manual/index.html

---

### 🟡 Shader Graph
- **用途：** 可视化着色器编辑器（基于节点的着色器创建）
- **使用场景：** 无需 HLSL 编码的自定义着色器
- **状态：** 生产就绪
- **包：** `com.unity.shadergraph`（URP/HDRP）
- **官方文档：** https://docs.unity3d.com/Packages/com.unity.shadergraph@17.0/manual/index.html

---

### 🟡 Timeline
- **用途：** 电影序列（过场动画、脚本事件）
- **使用场景：** 叙事驱动游戏、过场动画、脚本序列
- **状态：** 生产就绪
- **包：** `com.unity.timeline`（内置）
- **官方文档：** https://docs.unity3d.com/Packages/com.unity.timeline@1.8/manual/index.html

---

### 🟡 Animation Rigging
- **用途：** 运行时 IK、程序动画
- **使用场景：** 足部 IK、瞄准偏移、程序化肢体放置
- **状态：** 生产就绪（Unity 6）
- **包：** `com.unity.animation.rigging`
- **官方文档：** https://docs.unity3d.com/Packages/com.unity.animation.rigging@1.3/manual/index.html

---

### 🟡 ProBuilder
- **用途：** 编辑器内 3D 建模（关卡原型、灰盒）
- **使用场景：** 快速原型、关卡块出
- **状态：** 生产就绪
- **包：** `com.unity.probuilder`
- **官方文档：** https://docs.unity3d.com/Packages/com.unity.probuilder@6.0/manual/index.html

---

### 🟡 Netcode for GameObjects
- **用途：** Unity 官方多人游戏网络
- **使用场景：** 多人游戏（客户端-服务器架构）
- **状态：** 生产就绪
- **包：** `com.unity.netcode.gameobjects`
- **官方文档：** https://docs-multiplayer.unity3d.com/netcode/current/about/

---

### 🟡 Burst Compiler
- **用途：** 基于 LLVM 的 C# Jobs 编译器（巨大性能提升）
- **使用场景：** 性能关键代码、DOTS、Jobs 系统
- **状态：** 生产就绪
- **包：** `com.unity.burst`（随 DOTS 自动安装）
- **官方文档：** https://docs.unity3d.com/Packages/com.unity.burst@1.8/manual/index.html

---

### 🟡 Jobs System
- **用途：** 多线程作业调度（CPU 并行）
- **使用场景：** 性能优化、并行处理
- **状态：** 生产就绪
- **包：** 内置
- **官方文档：** https://docs.unity3d.com/Manual/JobSystem.html

---

### 🟡 Mathematics
- **用途：** SIMD 数学库（针对 Burst 优化）
- **使用场景：** DOTS、高性能数学
- **状态：** 生产就绪
- **包：** `com.unity.mathematics`
- **官方文档：** https://docs.unity3d.com/Packages/com.unity.mathematics@1.3/manual/index.html

---

### 🟡 ML-Agents（机器学习）
- **用途：** 通过强化学习训练 AI
- **使用场景：** 高级 AI 训练、程序行为
- **状态：** 生产就绪
- **包：** `com.unity.ml-agents`
- **官方文档：** https://github.com/Unity-Technologies/ml-agents

---

### 🟡 Recorder
- **用途：** 捕获游戏画面、截图、动画剪辑
- **使用场景：** 预告片、回放、调试录制
- **状态：** 生产就绪
- **包：** `com.unity.recorder`
- **官方文档：** https://docs.unity3d.com/Packages/com.unity.recorder@5.0/manual/index.html

---

## 预览/实验性包（谨慎使用）

### ⚠️ Splines
- **用途：** 运行时样条创建和编辑
- **使用场景：** 道路、路径、程序内容
- **状态：** 生产就绪（Unity 6）
- **包：** `com.unity.splines`
- **官方文档：** https://docs.unity3d.com/Packages/com.unity.splines@2.6/manual/index.html

---

### ⚠️ Muse（AI 助手）
- **用途：** AI 驱动的资源创建（纹理、精灵、动画）
- **状态：** 预览版（Unity 6）
- **包：** `com.unity.muse.*`
- **官方文档：** https://unity.com/products/muse

---

### ⚠️ Sentis（神经网络推理）
- **用途：** 在 Unity 中运行神经网络（AI 推理）
- **状态：** 预览版
- **包：** `com.unity.sentis`
- **官方文档：** https://docs.unity3d.com/Packages/com.unity.sentis@2.0/manual/index.html

---

## 已弃用的包（新项目避免使用）

### ❌ UGUI（Canvas UI）
- **已弃用：** 仍支持，但建议使用 UI Toolkit
- **改用：** UI Toolkit

---

### ❌ 旧版 Particle System
- **已弃用：** 使用 Visual Effect Graph（VFX Graph）
- **改用：** VFX Graph

---

### ❌ 旧版动画
- **已弃用：** 使用 Animator（Mecanim）
- **改用：** Animator Controller

---

## 按需 WebSearch 策略

对于上述未列出的包，当用户询问时使用以下方法：

1. **WebSearch** 最新文档：`"Unity 6.3 [包名]"`
2. 验证包是否：
   - 超出截止日期（超过 2025 年 5 月的训练数据）
   - 预览版 vs 生产就绪
   - 仍在 Unity 6.3 LTS 中支持
3. 可选择将发现结果缓存到 `plugins/[包名].md` 以供将来参考

---

## 快速决策指南

**需要虚拟相机** → **Cinemachine**
**需要异步资源加载 / DLC** → **Addressables**
**需要数千实体（RTS、模拟）** → **DOTS/Entities**
**需要现代输入** → **Input System**（参见 modules/input.md）
**需要 GPU 粒子** → **Visual Effect Graph**
**需要可视化着色器** → **Shader Graph**
**需要过场动画** → **Timeline**
**需要运行时 IK** → **Animation Rigging**
**需要关卡原型** → **ProBuilder**
**需要多人游戏** → **Netcode for GameObjects**

---

**最后更新：** 2026-02-13
**引擎版本：** Unity 6.3 LTS
**LLM 知识截止日期：** 2025 年 5 月
