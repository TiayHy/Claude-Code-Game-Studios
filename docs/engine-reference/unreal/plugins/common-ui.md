# Unreal Engine 5.7 — CommonUI 插件

**最后验证：** 2026-02-13
**状态：** 生产就绪
**插件：** `CommonUI`（内置，需在插件中启用）

---

## 概述

**CommonUI** 是一个跨平台 UI 框架，可自动处理手柄、鼠标和触摸的输入路由。
它专为需要在 PC、主机和移动平台上无缝运行的游戏而设计，最大限度地减少平台相关代码。

**使用 CommonUI 的场景：**
- 多平台游戏（主机 + PC）
- 自动手柄/鼠标/触摸输入路由
- 输入无关的 UI（同一 UI 适用于任何输入方式）
- 控件焦点和导航
- 快捷键栏和输入提示

**不要使用 CommonUI 的场景：**
- 仅 PC 游戏且仅使用鼠标的 UI（标准 UMG 更简单）
- 无导航需求的简单 UI

---

## 与标准 UMG 的主要区别

| 功能 | 标准 UMG | CommonUI |
|------|----------|----------|
| **输入处理** | 每个控件手动处理 | 自动路由 |
| **焦点管理** | 基础 | 高级导航 |
| **平台切换** | 手动检测 | 自动处理 |
| **输入提示** | 硬编码图标 | 按平台动态显示 |
| **屏幕栈** | 手动管理 | 内置可激活控件 |

---

## 设置

### 1. 启用插件

`Edit > Plugins > CommonUI > Enabled > Restart`

### 2. 配置项目设置

`Project Settings > Plugins > CommonUI`：
- **Default Input Type**：Gamepad（或自动检测）
- **Platform-Specific Settings**：按平台配置输入图标

### 3. 创建 Common Input Settings 资源

1. Content Browser > Input > Common Input Settings
2. 按平台配置输入数据：
   - Default Gamepad Data
   - Default Mouse & Keyboard Data
   - Default Touch Data

---

## 核心控件

### CommonActivatableWidget（屏幕管理）

可激活/停用的屏幕/菜单基类。

```cpp
#include "CommonActivatableWidget.h"

UCLASS()
class UMyMenuWidget : public UCommonActivatableWidget {
    GENERATED_BODY()

protected:
    virtual void NativeOnActivated() override {
        Super::NativeOnActivated();
        // 菜单现在可见且获得焦点
        UE_LOG(LogTemp, Warning, TEXT("菜单已激活"));
    }

    virtual void NativeOnDeactivated() override {
        Super::NativeOnDeactivated();
        // 菜单现在隐藏
        UE_LOG(LogTemp, Warning, TEXT("菜单已停用"));
    }

    virtual UWidget* NativeGetDesiredFocusTarget() const override {
        // 返回应接收焦点的控件（例如第一个按钮）
        return PlayButton;
    }

private:
    UPROPERTY(meta = (BindWidget))
    TObjectPtr<UCommonButtonBase> PlayButton;
};
```

---

### CommonButtonBase（输入感知的按钮）

替代标准 UMG Button。自动处理手柄/鼠标/键盘输入。

```cpp
#include "CommonButtonBase.h"

UCLASS()
class UMyMenuWidget : public UCommonActivatableWidget {
    GENERATED_BODY()

protected:
    UPROPERTY(meta = (BindWidget))
    TObjectPtr<UCommonButtonBase> PlayButton;

    virtual void NativeConstruct() override {
        Super::NativeConstruct();

        // 绑定按钮点击（适用于任何输入方式）
        PlayButton->OnClicked().AddUObject(this, &UMyMenuWidget::OnPlayClicked);

        // 设置按钮文本
        PlayButton->SetButtonText(FText::FromString(TEXT("Play")));
    }

    void OnPlayClicked() {
        UE_LOG(LogTemp, Warning, TEXT("点击了 Play"));
    }
};
```

---

### CommonTextBlock（样式化文本）

支持 CommonUI 样式的文本控件。

```cpp
UPROPERTY(meta = (BindWidget))
TObjectPtr<UCommonTextBlock> TitleText;

TitleText->SetText(FText::FromString(TEXT("Main Menu")));
```

---

### CommonActionWidget（输入提示）

显示输入提示（例如 "Press A to Continue"，自动显示正确的按钮图标）。

```cpp
UPROPERTY(meta = (BindWidget))
TObjectPtr<UCommonActionWidget> ConfirmActionWidget;

// 绑定到输入动作
ConfirmActionWidget->SetInputAction(ConfirmInputActionData);
// 自动显示正确的图标（Xbox 上是 A，PlayStation 上是 X，PC 上是 Enter）
```

---

## 控件栈（屏幕管理）

### CommonActivatableWidgetStack

管理屏幕栈（例如：主菜单 → 设置 → 按键设置）。

```cpp
#include "Widgets/CommonActivatableWidgetContainer.h"

UPROPERTY(meta = (BindWidget))
TObjectPtr<UCommonActivatableWidgetStack> WidgetStack;

// 将新屏幕压入栈
void ShowSettingsMenu() {
    WidgetStack->AddWidget(USettingsMenuWidget::StaticClass());
}

// 弹出当前屏幕（返回）
void GoBack() {
    WidgetStack->DeactivateWidget();
}
```

---

## 输入动作（CommonUI 风格）

### 定义输入动作

创建 **Common Input Action Data Table**：
1. Content Browser > Miscellaneous > Data Table
2. 行结构：`CommonInputActionDataBase`
3. 添加动作行（Confirm、Cancel、Navigate 等）

示例行：
- **Action Name**：Confirm
- **Default Input**：Gamepad Face Button Bottom（A/Cross）
- **Alternate Inputs**：Enter（键盘）、Left Mouse Button

---

### 在控件中绑定输入动作

```cpp
#include "Input/CommonUIActionRouterBase.h"

UCLASS()
class UMyWidget : public UCommonActivatableWidget {
    GENERATED_BODY()

protected:
    virtual void NativeOnActivated() override {
        Super::NativeOnActivated();

        // 绑定输入动作
        FBindUIActionArgs BindArgs(ConfirmInputAction, FSimpleDelegate::CreateUObject(this, &UMyWidget::OnConfirm));
        BindArgs.bDisplayInActionBar = true; // 在快捷栏中显示
        RegisterUIActionBinding(BindArgs);
    }

    void OnConfirm() {
        UE_LOG(LogTemp, Warning, TEXT("已确认"));
    }

private:
    UPROPERTY(EditDefaultsOnly, Category = "Input")
    FDataTableRowHandle ConfirmInputAction;
};
```

---

## 焦点与导航

### 自动手柄导航

CommonUI 自动处理手柄导航（D-Pad/摇杆在按钮之间移动）。

```cpp
// 在 Widget Blueprint 中：
// - 如果控件继承自 CommonButton/CommonUserWidget，则自动可导航
// - 焦点顺序由控件层级和布局决定
```

### 自定义焦点导航

```cpp
// 重写焦点导航
virtual UWidget* NativeGetDesiredFocusTarget() const override {
    return FirstButton; // 返回应接收焦点的控件
}
```

---

## 输入模式（游戏 vs UI）

### 切换输入模式

```cpp
#include "CommonUIExtensions.h"

// 切换到仅 UI 模式（暂停游戏，显示光标）
UCommonUIExtensions::PushStreamedGameplayUIInputConfig(this, FrontendInputConfig);

// 返回游戏模式（隐藏光标，恢复游戏玩法）
UCommonUIExtensions::PopInputConfig(this);
```

---

## 平台特定输入图标

### 配置输入图标

1. 为每个平台创建 **Common Input Base Controller Data** 资源：
   - Gamepad（Xbox、PlayStation、Switch）
   - Mouse & Keyboard
   - Touch

2. 分配平台特定图标：
   - Gamepad Face Button Bottom：`A`（Xbox）、`Cross`（PlayStation）
   - Confirm Key：`Enter` 图标

3. 分配到 **Common Input Settings** 资源

### 自动显示正确的图标

```cpp
// CommonActionWidget 自动为当前平台显示正确的图标
UPROPERTY(meta = (BindWidget))
TObjectPtr<UCommonActionWidget> JumpActionWidget;

JumpActionWidget->SetInputAction(JumpInputActionData);
// 在 Xbox 上显示 "A"，在 PlayStation 上显示 "Cross"，在 PC 上显示 "Space"
```

---

## 常见模式

### 带导航的主菜单

```cpp
UCLASS()
class UMainMenuWidget : public UCommonActivatableWidget {
    GENERATED_BODY()

protected:
    UPROPERTY(meta = (BindWidget))
    TObjectPtr<UCommonButtonBase> PlayButton;

    UPROPERTY(meta = (BindWidget))
    TObjectPtr<UCommonButtonBase> SettingsButton;

    UPROPERTY(meta = (BindWidget))
    TObjectPtr<UCommonButtonBase> QuitButton;

    virtual void NativeConstruct() override {
        Super::NativeConstruct();

        PlayButton->OnClicked().AddUObject(this, &UMainMenuWidget::OnPlayClicked);
        SettingsButton->OnClicked().AddUObject(this, &UMainMenuWidget::OnSettingsClicked);
        QuitButton->OnClicked().AddUObject(this, &UMainMenuWidget::OnQuitClicked);
    }

    virtual UWidget* NativeGetDesiredFocusTarget() const override {
        return PlayButton; // 打开菜单时聚焦 "Play" 按钮
    }

    void OnPlayClicked() { /* 开始游戏 */ }
    void OnSettingsClicked() { /* 打开设置 */ }
    void OnQuitClicked() { /* 退出游戏 */ }
};
```

---

### 带返回操作的暂停菜单

```cpp
UCLASS()
class UPauseMenuWidget : public UCommonActivatableWidget {
    GENERATED_BODY()

protected:
    UPROPERTY(EditDefaultsOnly, Category = "Input")
    FDataTableRowHandle BackInputAction; // 在 Blueprint 中分配 "Cancel" 动作

    virtual void NativeOnActivated() override {
        Super::NativeOnActivated();

        // 绑定 "Back" 输入（B/Circle/Escape）
        FBindUIActionArgs BindArgs(BackInputAction, FSimpleDelegate::CreateUObject(this, &UPauseMenuWidget::OnBack));
        RegisterUIActionBinding(BindArgs);
    }

    void OnBack() {
        DeactivateWidget(); // 关闭暂停菜单
    }
};
```

---

## 性能提示

- 使用 **CommonActivatableWidgetStack** 管理屏幕（自动处理激活/停用）
- 避免每帧创建/销毁控件（复用控件）
- 对复杂菜单使用 **Lazy Widgets**（仅在需要时创建）

---

## 调试

### CommonUI 调试命令

```cpp
// 控制台命令：
// CommonUI.DumpActivatableTree - 显示活动控件层级
// CommonUI.DumpActionBindings - 显示已注册的输入动作
```

---

## 参考来源
- https://docs.unrealengine.com/5.7/en-US/commonui-plugin-for-advanced-user-interfaces-in-unreal-engine/
- https://docs.unrealengine.com/5.7/en-US/commonui-quickstart-guide-for-unreal-engine/
