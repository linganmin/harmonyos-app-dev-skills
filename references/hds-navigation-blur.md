# HDS 通用动态模糊导航(HdsNavigation)

> 由 `harmonyos-app-dev` 使用。实现华为 HDS 规范的「标题栏随滚动渐变模糊」导航,对应官方指南
> `ui-design-navigation-dynamic-blur`。组件来自 `@kit.UIDesignKit`,API 以本机 SDK 声明为准:
> `/Applications/DevEco-Studio.app/Contents/sdk/default/hms/ets/api/@hms.hds.hdsBaseComponent.d.ets`。

## 心智模型

- **`HdsNavigation` 是视觉脚手架,不是路由**。它只提供标题栏(大标题/小标题、返回键、菜单)+ 内容区 + 滚动模糊联动;页面跳转仍由 `Navigation`/`NavPathStack` 或 router 负责。因此它可以安全地**嵌在 `NavDestination` 内部**(NavDestination 自身 `hideTitleBar(true)`),也可以用在 Tabs 的 TabContent 里。
- 模糊联动靠 **`bindToScrollable([scroller])`**:把内容区 Scroll/List 的 `Scroller` 交给它,标题栏据滚动偏移做 GRADIENT_BLUR。`Scroller` 与组件层级无关,中间隔一层 `Refresh` 也能正常工作。
- `titleMode` 三种(`HdsNavigationTitleMode`):`FREE`(默认,大标题随滚动收起——适合 Tab 一级页)、`FULL`、`MINI`(固定小标题——适合二级页)。

## 范式一:Tab 一级页(FREE 大标题 + 标题栏菜单 + 下拉刷新)

```ets
import { LengthMetrics, SymbolGlyphModifier } from '@kit.ArkUI';
import { HdsNavigation, HdsNavigationTitleMode, ScrollEffectType } from '@kit.UIDesignKit';

@Component
export struct StoriesTabView {
  @State refreshing: boolean = false;
  scroller: Scroller = new Scroller();

  build() {
    HdsNavigation() {
      Refresh({ refreshing: $$this.refreshing }) {
        Scroll(this.scroller) {
          Column() { /* 页面内容 */ }
            .padding({ left: 20, right: 20, top: 4, bottom: 172 })
        }
        .width('100%').height('100%')
        .align(Alignment.Top)          // 必加:Scroll 内容不足一屏默认居中
        .scrollBar(BarState.Off)
        .edgeEffect(EdgeEffect.Spring, { alwaysEnabled: true })
        .clip(false)                   // 不裁切,让内容滚入标题栏模糊区
      }
      .onRefreshing(() => this.reload())  // 完成后所有分支手动 this.refreshing = false
      .layoutWeight(1)
    }
    .titleBar({
      enableComponentSafeArea: true,   // 标题栏自行处理状态栏避让,内容区不再加 topAvoidPx
      avoidLayoutSafeArea: true,
      style: {
        scrollEffectOpts: {
          enableScrollEffect: true,
          scrollEffectType: ScrollEffectType.GRADIENT_BLUR,
          blurEffectiveStartOffset: LengthMetrics.vp(0),
          blurEffectiveEndOffset: LengthMetrics.vp(20)
        }
      },
      content: {
        title: { mainTitle: '故事', subTitle: '按场景、年龄和早教目标挑选' },
        menu: {                        // 标题栏右侧图标按钮
          value: [{
            content: {
              icon: new SymbolGlyphModifier($r('sys.symbol.magnifyingglass')),
              label: '搜索',
              action: () => NavService.getInstance().push('StorySearch')
            }
          }]
        }
      }
    })
    .titleMode(HdsNavigationTitleMode.FREE)
    .hideBackButton(true)              // Tab 页无返回
    .bindToScrollable([this.scroller])
    .width('100%').height('100%')
    .backgroundColor($r('app.color.page_bg'))
    .expandSafeArea([SafeAreaType.SYSTEM], [SafeAreaEdge.BOTTOM])
  }
}
```

## 范式二:二级页(NavDestination + MINI 小标题 + 返回键)

```ets
build() {
  NavDestination() {
    HdsNavigation() {
      Scroll(this.scroller) { /* 内容 */ }
        .align(Alignment.Top)
        .layoutWeight(1)
    }
    .titleBar({
      enableComponentSafeArea: true,
      avoidLayoutSafeArea: true,
      style: { scrollEffectOpts: { /* 同上 GRADIENT_BLUR */ } },
      content: {
        title: { mainTitle: '宝宝' },
        backIcon: { label: '返回', action: () => NavService.getInstance().pop() }
      }
    })
    .titleMode(HdsNavigationTitleMode.MINI)
    .hideBackButton(false)
    .bindToScrollable([this.scroller])
    .width('100%').height('100%')
    .backgroundColor($r('app.color.page_bg'))
    // bindSheet 等弹层挂在 HdsNavigation 链上即可
  }
  .hideTitleBar(true)                  // 关键:隐藏 NavDestination 自带标题栏,避免双标题
  .backgroundColor($r('app.color.page_bg'))
  .expandSafeArea([SafeAreaType.SYSTEM], [SafeAreaEdge.TOP, SafeAreaEdge.BOTTOM])
}
```

同一 App 内**所有二级页统一用此返回键**(titleBar backIcon),不要自绘 `‹` 按钮,深浅色与无障碍由 HDS 处理。

## 关键 API 结构(摘自 SDK d.ets)

- `titleBar(options?: HdsNavigationTitleBarOptions)`,其中 `content`:
  - `title: { mainTitle: ResourceStr, subTitle?: ResourceStr }`
  - `backIcon?: { icon?: IconType, label?: ResourceStr, action?: Callback<void> }`
  - `menu?: { value?: Array<{ content?: { icon?: IconType, label?, action? }, badge? }>, maxCount?: number }`(默认最多 3 个)
- `IconType = ResourceStr | SymbolGlyphModifier | PixelMap` —— 系统符号图标用 `new SymbolGlyphModifier($r('sys.symbol.xxx'))`。
- `titleMode(HdsNavigationTitleMode.FREE | FULL | MINI)`;`hideBackButton(boolean)`;`bindToScrollable(Scroller[])`。

## 配套:骨架屏占位(loading.jpg 风格)

数据加载期间用浅色圆角块 + 呼吸透明度动画占位;无限往复动画在 `onAppear` 用一次 `animateTo` 驱动:

```ets
@Component
export struct SkeletonBlock {
  blockWidth: Length = '100%';
  blockHeight: Length = 16;
  radius: number = 8;
  @State pulse: boolean = false;

  build() {
    Column()
      .width(this.blockWidth).height(this.blockHeight).borderRadius(this.radius)
      .backgroundColor($r('app.color.surface_muted'))
      .opacity(this.pulse ? 0.45 : 1)
      .onAppear(() => {
        this.getUIContext().animateTo({
          duration: 800, curve: Curve.EaseInOut,
          iterations: -1, playMode: PlayMode.Alternate
        }, () => { this.pulse = true; });
      })
  }
}
```

在其上组合 `SkeletonCard`(左方块 + 右侧条状)与 `SkeletonPage`(chips 行 + N 张卡),页面里 `if (this.loading) { SkeletonPage() } else { 真实内容 }`。纯 mock 页可模拟 400-600ms 加载以统一体验。

## 坑

- **双标题栏**:忘记给外层 `NavDestination` 加 `hideTitleBar(true)`,会出现系统标题栏 + HDS 标题栏叠加。
- **内容区重复避让**:`enableComponentSafeArea/avoidLayoutSafeArea` 开启后标题栏已避让状态栏,内容 padding 再加 `topAvoidPx` 会出现一截空白。
- **模糊不生效**:`bindToScrollable` 传的 Scroller 必须就是实际滚动容器在用的那个实例;给 Scroll 套 `Refresh` 不影响,但换成内层 List 滚动时要换绑 List 的 scroller。
- **`Refresh` 永远转圈**:`onRefreshing` 的每个出口(成功/失败/未登录早退)都要把 `refreshing` 置回 `false`。
