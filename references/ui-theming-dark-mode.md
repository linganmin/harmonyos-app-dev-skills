# HarmonyOS 深色模式适配

> 由 `harmonyos-app-dev` 使用。帮助开发者为 HarmonyOS 应用实现深色模式适配，包括语义色资源配置、系统栏适配、HDS 组件适配。

## 概述

深色模式适配的核心是**语义化色彩资源**：页面组件不直接写业务颜色或十六进制色值，而是通过语义色描述用途，再由 `base` 和 `dark` 资源目录提供具体色值。

**关键原则：**
- 使用 `$r('app.color.semantic_name')` 替代硬编码颜色
- 在 `resources/base/element/color.json` 和 `resources/dark/element/color.json` 中定义同名语义色
- 应用默认跟随系统颜色模式（`ColorMode.COLOR_MODE_NOT_SET`）
- 深色模式不是简单反色，需要重新定义视觉层级

## 一、语义色资源体系

### 1.1 推荐语义色命名

| 名称 | 用途 | 浅色示例 | 深色示例 |
| --- | --- | --- | --- |
| `page_bg` | 页面背景 | `#F8F3EA` | `#101713` |
| `surface` | 卡片、列表项、输入区域背景 | `#FFFFFF` | `#18221E` |
| `surface_muted` | 弱强调背景、浅按钮背景 | `#E5F2EC` | `#1F332D` |
| `text_primary` | 主要标题和正文 | `#21312D` | `#F4F1E8` |
| `text_secondary` | 说明文字 | `#68756F` | `#B7C2BA` |
| `text_tertiary` | 禁用态、弱提示 | `#9AA8A1` | `#7F8D86` |
| `brand` | 主品牌色、主按钮、选中态 | `#1E6B5D` | `#6FD1B8` |
| `brand_soft` | 品牌弱背景 | `#D9EEE4` | `#183A32` |
| `danger` | 危险操作、紧急提示 | `#A43F4B` | `#F08A92` |
| `danger_soft` | 危险弱背景 | `#F5DFE1` | `#3D1F24` |
| `accent_brown` | 特色强调色（如声音、训练等暖色场景） | `#8B5E34` | `#D6A66E` |
| `accent_brown_soft` | 暖色弱背景 | `#F1E1CC` | `#392A1D` |
| `divider` | 分割线、描边 | `#E5E2D9` | `#293832` |
| `tab_mask` | 底部浮动标签栏渐变蒙版 | `#66F8F3EA` | `#66101713` |
| `system_bar_content` | 状态栏、导航栏内容色 | `#21312D` | `#F4F1E8` |

### 1.2 资源配置示例

**`products/default/src/main/resources/base/element/color.json`**

```json
{
  "color": [
    { "name": "page_bg", "value": "#F8F3EA" },
    { "name": "surface", "value": "#FFFFFF" },
    { "name": "text_primary", "value": "#21312D" },
    { "name": "brand", "value": "#1E6B5D" },
    { "name": "system_bar_content", "value": "#21312D" }
  ]
}
```

**`products/default/src/main/resources/dark/element/color.json`**

```json
{
  "color": [
    { "name": "page_bg", "value": "#101713" },
    { "name": "surface", "value": "#18221E" },
    { "name": "text_primary", "value": "#F4F1E8" },
    { "name": "brand", "value": "#6FD1B8" },
    { "name": "system_bar_content", "value": "#F4F1E8" }
  ]
}
```

### 1.3 组件使用示例

```typescript
@Entry
@Component
struct HomePage {
  build() {
    Column() {
      Text('今晚建议')
        .fontColor($r('app.color.text_primary'))
      Button('开始生成')
        .backgroundColor($r('app.color.brand'))
    }
    .backgroundColor($r('app.color.page_bg'))
  }
}
```

## 二、系统栏适配

### 2.1 系统栏内容色切换

在 `UIAbility` 的 `onWindowStageCreate` 中设置：

```typescript
import { window } from '@kit.ArkUI';
import { ConfigurationConstant } from '@kit.AbilityKit';

export default class DefaultAbility extends UIAbility {
  onWindowStageCreate(windowStage: window.WindowStage): void {
    windowStage.getMainWindow((err, mainWindow) => {
      const colorMode = this.context.config.colorMode;
      const systemBarContentColor = 
        colorMode === ConfigurationConstant.ColorMode.COLOR_MODE_DARK
          ? '#F4F1E8' : '#21312D';
      
      mainWindow.setWindowLayoutFullScreen(true);
      mainWindow.setWindowSystemBarProperties({
        statusBarColor: '#00000000',
        navigationBarColor: '#00000000',
        statusBarContentColor: systemBarContentColor,
        navigationBarContentColor: systemBarContentColor
      });
      
      windowStage.loadContent('pages/Index');
    });
  }
}
```

### 2.2 监听颜色模式变化

```typescript
export default class DefaultAbility extends UIAbility {
  private mainWindow?: window.Window;
  
  onWindowStageCreate(windowStage: window.WindowStage): void {
    windowStage.getMainWindow((err, mainWindow) => {
      this.mainWindow = mainWindow;
      this.updateSystemBarStyle();
    });
  }
  
  onConfigurationUpdate(newConfig: Configuration): void {
    super.onConfigurationUpdate(newConfig);
    this.updateSystemBarStyle();
  }
  
  private updateSystemBarStyle(): void {
    if (!this.mainWindow) return;
    const colorMode = this.context.config.colorMode;
    const systemBarContentColor = 
      colorMode === ConfigurationConstant.ColorMode.COLOR_MODE_DARK
        ? '#F4F1E8' : '#21312D';
    
    this.mainWindow.setWindowSystemBarProperties({
      statusBarContentColor: systemBarContentColor,
      navigationBarContentColor: systemBarContentColor
    });
  }
}
```

## 三、HDS 组件适配

### 3.1 HdsTabs 颜色配置

```typescript
HdsTabs({ controller: this.tabsController }) {
  TabContent() { /* ... */ }
    .tabBar(this.tabBuilder('首页', 0))
}
.barFloatingStyle({
  gradientMask: {
    maskColor: $r('app.color.tab_mask'),
    maskHeight: 120
  }
})

@Builder
tabBuilder(title: string, index: number) {
  Text(title)
    .fontColor(this.selectedIndex === index 
      ? $r('app.color.brand') 
      : $r('app.color.text_tertiary'))
}
```

### 3.2 SVG 图标着色

```typescript
Image($r('app.media.ic_home'))
  .width(24)
  .height(24)
  .fillColor(this.selected 
    ? $r('app.color.brand')
    : $r('app.color.text_tertiary'))
```

如果 SVG 内部写死了颜色，需要：
1. 修改 SVG 源文件，移除 `fill` 属性
2. 或准备两套 SVG，分别放在 `resources/base/media` 和 `resources/dark/media`

## 四、代码改造策略

### 4.1 替换硬编码颜色

```typescript
// ❌ 旧写法
.backgroundColor('#F8F3EA')
.fontColor('#21312D')

// ✅ 新写法
.backgroundColor($r('app.color.page_bg'))
.fontColor($r('app.color.text_primary'))
```

### 4.2 搜索验证

```bash
rg -n "fontColor\('#|backgroundColor\('#|fillColor\('#" \
  products/default/src/main/ets --type ts
```

允许保留：完全透明 `#00000000`、第三方组件兼容代码

## 五、常见陷阱

1. **深色模式 ≠ 简单反色** - 需要重新设计层级，深色背景上的卡片应略亮于背景
2. **系统栏内容色固定** - 必须根据颜色模式动态切换，否则深色模式下图标看不见
3. **HDS API 类型限制** - 部分 API 只接受 `string`，需通过 `resourceManager` 获取色值
4. **SVG 内部颜色写死** - `fillColor` 属性无效，需修改 SVG 或准备 dark 版本

## 六、官方文档参考

- **颜色资源配置**：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/resource-categories-and-access`
- **沉浸式系统栏**：参见 `references/immersive-system-bars.md`
- **HdsTabs 组件**：`https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-arkui-advanced-hdstabs`

