# HarmonyOS 浮动标签栏 + 迷你栏

> 由 `harmonyos-app-dev` 使用。帮助开发者实现 HDS 浮动标签栏（Floating Tabs）与迷你栏（MiniBar）组合布局。

## 概述

浮动标签栏是 HarmonyOS Design System（HDS）提供的底部导航组件，支持在标签栏上方添加迷你栏（如音频播放器控制条）。该组合提供：
- **常驻播放控制**：用户在任意 Tab 都能快速访问播放器
- **空间优化**：将播放控制整合到底部浮动容器，节省内容区域
- **视觉统一**：迷你栏 + 标签栏形成完整悬浮卡片，层次清晰
- **符合规范**：严格遵循 HDS 设计系统

## 一、核心实现

### 1.1 迷你栏 Builder

```typescript
@Builder
miniBarBuilder() {
  Row() {
    // 播放状态图标
    Column() {
      Image($r('sys.media.ohos_ic_public_play'))
        .width(40)
        .height(40)
        .fillColor($r('app.color.brand'))
    }
    .width(48)
    .height(48)
    .justifyContent(FlexAlign.Center)
    .margin({ left: 4, right: 8 })

    // 播放信息
    Column() {
      Text('正在播放：小鹿和勇敢的云朵')
        .fontSize(14)
        .fontWeight(FontWeight.Medium)
        .fontColor($r('app.color.text_primary'))
        .maxLines(1)
        .textOverflow({ overflow: TextOverflow.Ellipsis })
      Text('睡前故事 · 08:24')
        .fontSize(12)
        .fontColor($r('app.color.text_secondary'))
        .margin({ top: 2 })
    }
    .alignItems(HorizontalAlign.Start)
    .layoutWeight(1)

    // 暂停/播放按钮
    Column() {
      Image($r('sys.media.ohos_ic_public_pause'))
        .width(24)
        .height(24)
        .fillColor($r('app.color.brand'))
    }
    .width(40)
    .height(40)
    .justifyContent(FlexAlign.Center)
    .margin({ right: 4 })
    .onClick(() => {
      // 处理播放/暂停
    })
  }
  .width('100%')
  .height(56)
  .padding({ left: 8, right: 8 })
  .onClick(() => {
    // 跳转到播放器详情页
  })
}
```

### 1.2 配置 HdsTabs

```typescript
import { HdsTabs } from '@kit.ArkUI';
import { hdsMaterial } from '@kit.ArkUI';

@Entry
@Component
struct Home {
  @State selectedTabIndex: number = 0;
  @StorageProp('bottomAvoidPx') bottomAvoidPx: number = 0;
  private tabsController: TabsController = new TabsController();
  
  build() {
    HdsTabs({
      controller: this.tabsController,
      index: this.selectedTabIndex,
      barPosition: BarPosition.End
    }) {
      TabContent() { /* 首页内容 */ }
      TabContent() { /* 作品内容 */ }
      TabContent() { /* 我的内容 */ }
    }
    .barOverlap(true)
    .barPosition(BarPosition.End)
    .barHeight(64)
    .barFloatingStyle({
      barSideMargin: 16,
      barBottomMargin: 8 + px2vp(this.bottomAvoidPx),
      barWidth: {
        smallWidth: 'calc(100% - 32vp)',
        mediumWidth: 360,
        largeWidth: 420
      },
      gradientMask: {
        maskColor: $r('app.color.tab_mask'),
        maskHeight: 120  // 适应迷你栏高度
      },
      systemMaterialEffect: {
        materialType: hdsMaterial.MaterialType.IMMERSIVE,
        materialLevel: hdsMaterial.MaterialLevel.ADAPTIVE
      },
      miniBar: {
        miniBarBuilder: () => this.miniBarBuilder()
      }
    })
    .onChange((index: number) => {
      this.selectedTabIndex = index;
    })
  }
}
```

**关键参数调整：**
- `gradientMask.maskHeight`: 从 92 增加到 120，适应迷你栏（56vp）
- `miniBar.miniBarBuilder`: 传入自定义迷你栏构建函数

### 1.3 内容区域 padding 调整

```typescript
// 每个 Tab 的内容区域
Column() {
  // 页面内容
}
.padding({
  left: 20,
  right: 20,
  top: 12,
  bottom: 172 + px2vp(this.bottomAvoidPx)
})
```

**高度计算：**
```
总 padding = 迷你栏 + 标签栏 + 间距 + 渐变区域 + 避让高度
         = 56 + 64 + 16 + 36 + bottomAvoidPx
         = 172 + bottomAvoidPx
```

## 二、视觉效果层次

```
┌─────────────────────────┐
│   渐变蒙版（120vp）      │  ← 从内容区域平滑过渡
│                         │
│  ┌───────────────────┐  │
│  │   迷你栏（56vp）   │  │  ← 沉浸式材料效果
│  ├───────────────────┤  │
│  │  标签栏（64vp）    │  │  ← 沉浸式材料效果
│  └───────────────────┘  │
│                         │
│   底部边距 + 避让高度    │
└─────────────────────────┘
```

**材料效果：**
- `MaterialType.IMMERSIVE`: 毛玻璃质感
- `gradientMask`: 平滑过渡
- 圆角容器：统一的悬浮卡片
- `barOverlap: true`: 悬浮在内容之上

## 三、响应式适配

| 屏幕尺寸 | 浮动容器宽度 |
|---------|-------------|
| 小屏 (<600vp) | `calc(100% - 32vp)` 自适应 |
| 中屏 (600-840vp) | 360vp 固定宽度 |
| 大屏 (>840vp) | 420vp 固定宽度 |

## 四、动态状态管理

```typescript
@State isPlaying: boolean = false;
@State currentTrack: string = '小鹿和勇敢的云朵';
@State currentDuration: string = '08:24';

@Builder
miniBarBuilder() {
  Row() {
    Image(this.isPlaying ? 
      $r('sys.media.ohos_ic_public_pause') : 
      $r('sys.media.ohos_ic_public_play'))
      .fillColor($r('app.color.brand'))
    
    Column() {
      Text(`正在播放：${this.currentTrack}`)
        .fontColor($r('app.color.text_primary'))
      Text(`睡前故事 · ${this.currentDuration}`)
        .fontColor($r('app.color.text_secondary'))
    }
    .layoutWeight(1)
  }
  .height(56)
  .onClick(() => {
    // 跳转到播放器
    this.getUIContext().getRouter().pushUrl({ url: 'pages/Player' });
  })
}
```

## 五、常见陷阱

1. **maskHeight 不足** - 如果只保留默认值 92，迷你栏底部会被渐变遮挡，必须增加到 120
2. **内容 padding 未调整** - 忘记增加底部 padding，导致内容被迷你栏遮挡
3. **HDS API 版本** - 确保使用支持 `miniBar` 配置的 HDS 版本（API 12+）
4. **颜色硬编码** - 迷你栏中的颜色应使用语义色资源，配合深色模式适配

## 六、官方文档参考

- **浮动标签栏 + 迷你栏**：`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ui-design-hds-tabs-bar-floating`
- **HdsTabs API**：`https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-arkui-advanced-hdstabs`
- **材料效果**：`https://developer.huawei.com/consumer/cn/doc/HarmonyOS-Guides/ui-design-hds-component-material`

---

> **最佳实践总结**：浮动标签栏 + 迷你栏是音频、视频类应用的标准布局模式。迷你栏高度建议 56vp，确保足够的触摸目标尺寸。整个浮动容器使用沉浸式材料效果，配合渐变蒙版实现自然过渡。



