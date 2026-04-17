# Godot 渲染 —— 速查表

最后验证：2026-02-12 | 引擎：Godot 4.6

## 自 ~4.3（LLM 截止日期）以来的变化

### 4.6 变化
- **D3D12 是 Windows 默认渲染后端**（此前为 Vulkan）
- **Glow 在色调映射之前处理**（此前在之后）—— 使用屏幕混合模式
- **AgX 色调映射器**：新增白点和对比度控制
- **SSR 全面改进**：更好的真实感、视觉稳定性和性能

### 4.5 变化
- **着色器烘焙器**：预编译着色器以减少启动时间
- **SMAA 1x**：新的抗锯齿选项（比 FXAA 更锐利，比 TAA 更便宜）
- **模板缓冲支持**：支持选择性几何遮罩/传送门效果
- **弯曲法线贴图**：法线贴图中编码方向遮挡
- **镜面遮蔽**：环境光遮蔽现在正确影响反射

### 4.4 变化
- **`RenderingDevice.draw_list_begin`**：移除大量参数；新增可选的 `breadcrumb`
- **着色器纹理类型**：从 `Texture2D` 改为 `Texture` 基类
- **粒子 `.restart()`**：新增可选的 `keep_seed` 参数

### 4.3 变化（训练数据中已有）
- **Compositor 节点**：`Compositor` + `CompositorEffect` 用于后处理链

## 当前 API 模式

### 后处理（4.3+）
```gdscript
# 使用 Compositor 节点 —— 不要用手动视口着色器链
# 将 Compositor 添加为 WorldEnvironment 或 Camera3D 的子节点
# 为每个后处理步骤创建 CompositorEffect 资源
```

### 抗锯齿选项（4.6）
```
项目设置 → 渲染 → 抗锯齿：
- MSAA 2D/3D：硬件 MSAA（质量好但昂贵）
- 屏幕空间 AA：FXAA（快，模糊）或 SMAA（锐利，中等成本）  # SMAA 4.5 新增
- TAA：时域（质量最好，快速运动有鬼影）
```

### 渲染后端选择（4.6）
```
项目设置 → 渲染 → 渲染器：
- Forward+（默认）：功能完整，桌面端为主
- Mobile：针对移动端/低端优化，功能有限
- Compatibility：OpenGL 3.3 / WebGL 2，硬件支持最广

Windows 默认后端：D3D12（4.6 前为 Vulkan）
```

## 常见错误
- 假设 Vulkan 是 Windows 默认后端（4.6 起为 D3D12）
- 使用手动视口链而非 Compositor 做后处理
- 在着色器 uniform 类型中使用 `Texture2D`（4.4 起使用 `Texture`）
- 着色器变体较多的项目未使用着色器烘焙器