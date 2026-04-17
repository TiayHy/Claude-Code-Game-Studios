# Unreal Engine — 版本参考

| 字段 | 值 |
|-------|-------|
| **引擎版本** | Unreal Engine 5.7 |
| **发布日期** | 2025 年 11 月 |
| **项目锁定日期** | 2026-02-13 |
| **最后验证文档日期** | 2026-02-13 |
| **LLM 知识截止日期** | 2025 年 5 月 |

## 知识差距警告

LLM 的训练数据可能覆盖 Unreal Engine 约至 5.3 版本。5.4、5.5、5.6 和 5.7 版本引入了模型**不知道**的重大变更。
在建议 Unreal API 调用之前，务必先参考此目录。

## 截止日期后版本时间线

| 版本 | 发布日期 | 风险等级 | 核心主题 |
|---------|---------|------------|-----------|
| 5.4 | ~2025 年中 | HIGH | Motion Design 工具、动画改进、PCG 增强 |
| 5.5 | ~2025 年 9 月 | HIGH | Megalights（百万光源）、动画创作、MegaCity 演示 |
| 5.6 | ~2025 年 10 月 | MEDIUM | 性能优化、bug 修复 |
| 5.7 | 2025 年 11 月 | HIGH | PCG 生产可用、Substrate 生产可用、AI 助手 |

## UE 5.3 到 UE 5.7 的主要变更

### 破坏性变更
- **Substrate Material System**：新材质框架（取代传统材质）
- **PCG（程序化内容生成）**：生产可用，API 重大变更
- **Megalights**：新光照系统（百万动态光源）
- **Animation Authoring**：新绑骨和动画工具
- **AI Assistant**：编辑器内 AI 指导（实验性）

### 新功能（截止日期后）
- **Megalights**：大规模动态光照（百万光源）
- **Substrate Materials**：生产可用模块化材质系统
- **PCG Framework**：程序化世界生成（5.7 生产可用）
- **Enhanced Virtual Production**：MetaHuman 集成、更深入的 VP 工作流
- **Animation Improvements**：更好的绑骨、混合、程序化动画
- **AI Assistant**：编辑器内 AI 帮助（实验性）

### 已废弃系统
- **Legacy Material System**：新项目请迁移到 Substrate
- **Old PCG API**：使用新的生产级 PCG API（5.7+）

## 已验证来源

- 官方文档：https://docs.unrealengine.com/5.7/
- UE 5.7 发布说明：https://dev.epicgames.com/documentation/en-us/unreal-engine/unreal-engine-5-7-release-notes
- 5.7 新功能：https://dev.epicgames.com/documentation/en-us/unreal-engine/whats-new
- UE 5.7 公告：https://www.unrealengine.com/en-US/news/unreal-engine-5-7-is-now-available
- UE 5.5 博客：https://www.unrealengine.com/en-US/blog/unreal-engine-5-5-is-now-available
- 迁移指南：https://docs.unrealengine.com/5.7/en-US/upgrading-projects/
