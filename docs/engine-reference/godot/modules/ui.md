# Godot UI —— 速查表

最后验证：2026-02-12 | 引擎：Godot 4.6

## 自 ~4.3（LLM 截止日期）以来的变化

### 4.6 变化
- **双焦点系统**：鼠标/触摸焦点与键盘/手柄焦点现已分离
  - 不同输入方式有不同的视觉反馈
  - 自定义焦点实现可能需要更新
- **TabContainer**：标签属性可直接在检视器中编辑
- **TileMapLayer 场景瓦片旋转**：场景瓦片可以像图集瓦片一样旋转

### 4.5 变化
- **FoldableContainer**：新的手风琴风格 UI 节点，用于可折叠区域
- **递归 Control 行为**：单个属性即可禁用整棵节点层级树的鼠标/焦点
- **屏幕阅读器支持**：Control 节点支持 AccessKit
- **实时翻译预览**：在编辑器中测试不同语言环境
- **`RichTextLabel.push_meta`**：新增可选的 `tooltip` 参数（4.4 起）

### 4.4 变化
- **`GraphEdit.connect_node`**：新增可选的 `keep_alive` 参数

## 当前 API 模式

### 主题与样式（4.6）
```gdscript
# 编辑器默认使用新的"现代"主题
# 游戏 UI 照旧使用自定义主题：
var theme := Theme.new()
theme.set_color(&"font_color", &"Label", Color.WHITE)
theme.set_font_size(&"font_size", &"Label", 24)
```

### 焦点管理（4.6 — 已变更）
```gdscript
# 键盘/手柄焦点（grab_focus 仍然有效）
func _ready() -> void:
    %StartButton.grab_focus()

# 重要：4.6 中鼠标悬停与键盘焦点是分离的
# 两者可同时在不同控件上激活
# 用鼠标和键盘/手柄两种方式测试 UI

# 焦点邻居（未变）
%Button1.focus_neighbor_bottom = %Button2.get_path()
%Button1.focus_neighbor_right = %Button3.get_path()
```

### FoldableContainer（4.5 — 新增）
```gdscript
# 手风琴风格可折叠容器
# 添加为你想让其可折叠的内容的父节点
# 点击标题时子节点显示/隐藏
# 通过编辑器属性或代码配置
```

### 递归禁用（4.5 — 新增）
```gdscript
# 禁用整棵层级的所有鼠标/焦点交互
# 用于禁用整个菜单区域
%SettingsPanel.mouse_filter = Control.MOUSE_FILTER_IGNORE
# 4.5 起可递归传播到子节点
```

### 国际化就绪 UI（最佳实践）
```gdscript
# 所有可见字符串使用 tr()
label.text = tr("MENU_START_GAME")

# 使用自动换行应对标签（文本长度因语言而异）
label.autowrap_mode = TextServer.AUTOWRAP_WORD_SMART

# 在编辑器中用实时翻译预览测试（4.5+）
```

## 常见错误
- 假设 `grab_focus()` 会影响鼠标焦点（4.6 中仅影响键盘/手柄）
- 升级到 4.6 后未用鼠标和手柄同时测试 UI
- 硬编码字符串而非使用 `tr()` 做国际化
- 可折叠 UI 未使用 `FoldableContainer`（4.5 新增，比自定义更简洁）