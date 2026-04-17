# Unity 6.3 — UI 模块参考

**最后验证：** 2026-02-13
**知识差距：** Unity 6 UI Toolkit 运行时 UI 生产就绪

---

## 概述

Unity 6 UI 系统：
- **UI Toolkit**（推荐）：现代、高性能、类 HTML/CSS（Unity 6 生产就绪）
- **UGUI（Canvas）**：旧版系统，仍支持但不推荐新项目使用
- **IMGUI**：仅编辑器用，已弃用用于运行时 UI

---

## UI Toolkit（现代 UI）

### 设置 UI Document

1. 创建 UXML（UI 结构）：
   - `Assets > Create > UI Toolkit > UI Document`
2. 创建 USS（样式）：
   - `Assets > Create > UI Toolkit > StyleSheet`
3. 添加到场景：
   - `GameObject > UI Toolkit > UI Document`
   - 赋值 UXML 到 `UIDocument > Source Asset`

---

### UXML（UI 结构）

```xml
<!-- MainMenu.uxml -->
<ui:UXML xmlns:ui="UnityEngine.UIElements">
    <ui:VisualElement class="container">
        <ui:Label text="主菜单" class="title" />
        <ui:Button name="play-button" text="开始游戏" />
        <ui:Button name="settings-button" text="设置" />
        <ui:Button name="quit-button" text="退出" />
    </ui:VisualElement>
</ui:UXML>
```

---

### USS（样式）

```css
/* MainMenu.uss */
.container {
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 100%;
    background-color: rgb(30, 30, 30);
}

.title {
    font-size: 48px;
    color: white;
    margin-bottom: 20px;
}

Button {
    width: 200px;
    height: 50px;
    margin: 10px;
    font-size: 24px;
}

Button:hover {
    background-color: rgb(100, 150, 200);
}
```

---

### C# 脚本（UI Toolkit）

```csharp
using UnityEngine;
using UnityEngine.UIElements;

public class MainMenu : MonoBehaviour {
    void OnEnable() {
        var root = GetComponent<UIDocument>().rootVisualElement;

        // 按名称查询元素
        var playButton = root.Q<Button>("play-button");
        var settingsButton = root.Q<Button>("settings-button");
        var quitButton = root.Q<Button>("quit-button");

        // 注册回调
        playButton.clicked += OnPlayClicked;
        settingsButton.clicked += OnSettingsClicked;
        quitButton.clicked += Application.Quit;
    }

    void OnPlayClicked() {
        Debug.Log("开始游戏点击");
        // 加载游戏场景
    }

    void OnSettingsClicked() {
        Debug.Log("设置点击");
        // 打开设置菜单
    }
}
```

---

### 常用 UI 元素

```csharp
// Label（文本显示）
var label = root.Q<Label>("score-label");
label.text = "分数：100";

// Button
var button = root.Q<Button>("submit-button");
button.clicked += OnSubmit;

// TextField（文本输入）
var textField = root.Q<TextField>("name-input");
string playerName = textField.value;

// Toggle（复选框）
var toggle = root.Q<Toggle>("music-toggle");
bool isMusicEnabled = toggle.value;

// Slider
var slider = root.Q<Slider>("volume-slider");
float volume = slider.value; // 0-1

// DropdownField（下拉菜单）
var dropdown = root.Q<DropdownField>("difficulty-dropdown");
dropdown.choices = new List<string> { "简单", "普通", "困难" };
dropdown.value = "普通";
```

---

### 动态创建 UI（无需 UXML）

```csharp
void CreateUI() {
    var root = GetComponent<UIDocument>().rootVisualElement;

    // 创建元素
    var container = new VisualElement();
    container.AddToClassList("container");

    var label = new Label("你好，UI Toolkit！");
    var button = new Button(() => Debug.Log("点击")) { text = "点我" };

    container.Add(label);
    container.Add(button);
    root.Add(container);
}
```

---

### USS Flexbox 布局

```css
/* 水平布局 */
.horizontal {
    flex-direction: row;
}

/* 垂直布局（默认） */
.vertical {
    flex-direction: column;
}

/* 居中子元素 */
.centered {
    align-items: center;
    justify-content: center;
}

/* 间距 */
.spaced {
    justify-content: space-between;
}
```

---

## UGUI（旧版 Canvas UI）

### 基本设置（Unity 6 中仍然可用）

```csharp
// GameObject > UI > Canvas（创建 Canvas、EventSystem）

// UI 元素：
// - Text（改用 TextMeshPro）
// - Button
// - Image
// - Slider
// - Toggle
// - InputField
```

---

### UGUI 脚本

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro; // TextMeshPro

public class LegacyUI : MonoBehaviour {
    public TextMeshProUGUI scoreText;
    public Button playButton;
    public Slider volumeSlider;

    void Start() {
        // 更新文本
        scoreText.text = "分数：100";

        // 按钮点击
        playButton.onClick.AddListener(OnPlayClicked);

        // 滑块值变化
        volumeSlider.onValueChanged.AddListener(OnVolumeChanged);
    }

    void OnPlayClicked() {
        Debug.Log("开始游戏点击");
    }

    void OnVolumeChanged(float value) {
        AudioListener.volume = value;
    }
}
```

---

### TextMeshPro（更好的文本渲染）

```csharp
// 安装：Window > TextMeshPro > Import TMP Essential Resources

// 使用 TMP_Text 而非 Unity 的 Text 组件
using TMPro;

public TextMeshProUGUI tmpText;
tmpText.text = "高质量文本";
tmpText.fontSize = 24;
tmpText.color = Color.white;
```

---

## Canvas 设置（UGUI）

### 渲染模式

```csharp
// Screen Space - Overlay：UI 渲染在所有内容之上（不需要相机）
// Screen Space - Camera：UI 由特定相机渲染（允许效果）
// World Space：UI 在 3D 世界中（例如浮动血条）
```

### Canvas Scaler（响应式 UI）

```csharp
// UI Scale Mode：
// - Constant Pixel Size：UI 元素具有固定像素大小
// - Scale With Screen Size：UI 基于参考分辨率缩放（推荐）
// - Constant Physical Size：UI 元素具有固定物理大小（cm）

// 示例：Scale With Screen Size
// Reference Resolution：1920x1080
// Screen Match Mode：Match Width Or Height（0.5 = 平衡）
```

---

## 布局组（UGUI）

### Horizontal Layout Group

```csharp
// 自动水平排列子元素
// 添加：GameObject > Add Component > Horizontal Layout Group
```

### Vertical Layout Group

```csharp
// 自动垂直排列子元素
```

### Grid Layout Group

```csharp
// 以网格排列子元素
```

---

## 性能（UI Toolkit vs UGUI）

### UI Toolkit 优势
- ✅ 更快的渲染（保留模式）
- ✅ 更适合元素众多的复杂 UI
- ✅ 更简单的样式（类 CSS）
- ✅ 更适合动态 UI

### UGUI 优势
- ✅ 更成熟，文档更完善
- ✅ 与 Unity 编辑器更好的集成
- ✅ 对初学者更简单

---

## 常见模式

### 血条（UI Toolkit）

```csharp
var healthBar = root.Q<VisualElement>("health-bar");
healthBar.style.width = new StyleLength(new Length(healthPercent, LengthUnit.Percent));
```

### 血条（UGUI）

```csharp
public Image healthBarImage;

void UpdateHealth(float percent) {
    healthBarImage.fillAmount = percent; // 0-1
}
```

---

### 淡入/淡出（UI Toolkit）

```csharp
IEnumerator FadeIn(VisualElement element, float duration) {
    float elapsed = 0f;
    while (elapsed < duration) {
        elapsed += Time.deltaTime;
        element.style.opacity = Mathf.Lerp(0f, 1f, elapsed / duration);
        yield return null;
    }
}
```

---

## 调试

### UI Toolkit Debugger
- `Window > UI Toolkit > Debugger`
- 检查元素层级、样式、布局

### UGUI Event System Debugger
- 在层级视图中选择 EventSystem
- Inspector 显示活动输入模块、光线投射信息

---

## 来源
- https://docs.unity3d.com/6000.0/Documentation/Manual/UIElements.html
- https://docs.unity3d.com/Packages/com.unity.ui@2.0/manual/index.html
- https://docs.unity3d.com/Packages/com.unity.ugui@2.0/manual/index.html
