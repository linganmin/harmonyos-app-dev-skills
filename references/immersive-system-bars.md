# 沉浸式系统栏与底部导航安全区

用于 HarmonyOS NEXT / ArkUI 应用实现沉浸式状态栏、底部导航条和手势指示区避让。

## 原则

- 在 `UIAbility.onWindowStageCreate()` 中配置窗口,不要在页面组件里首次开启沉浸式。
- `setWindowLayoutFullScreen(true)` 只负责让布局延伸到状态栏和底部导航区域,不负责内容避让。
- 开启沉浸式后必须读取并监听避让区:
  - `window.AvoidAreaType.TYPE_SYSTEM`: 状态栏、挖孔等顶部系统避让区域。
  - `window.AvoidAreaType.TYPE_NAVIGATION_INDICATOR`: 底部手势条/导航指示区。
- 背景可以扩展进安全区,可点击内容、顶部栏、底部导航和底部按钮必须避让。
- 不要为了普通业务页面隐藏系统导航栏;只在视频、游戏、全屏播放器等临时场景考虑隐藏。

## Ability 侧窗口配置

```ts
import { UIAbility } from '@kit.AbilityKit';
import { BusinessError } from '@kit.BasicServicesKit';
import { hilog } from '@kit.PerformanceAnalysisKit';
import { window } from '@kit.ArkUI';

const DOMAIN = 0x0000;

export default class EntryAbility extends UIAbility {
  onWindowStageCreate(windowStage: window.WindowStage): void {
    windowStage.loadContent('pages/Index', (err) => {
      if (err.code) {
        return;
      }

      const mainWindow: window.Window = windowStage.getMainWindowSync();
      this.configureImmersiveWindow(mainWindow);
    });
  }

  private configureImmersiveWindow(mainWindow: window.Window): void {
    mainWindow.setWindowLayoutFullScreen(true).catch((err: BusinessError) => {
      hilog.error(DOMAIN, 'immersive', 'setWindowLayoutFullScreen failed: %{public}s', JSON.stringify(err));
    });

    mainWindow.setWindowSystemBarProperties({
      statusBarColor: '#00000000',
      navigationBarColor: '#00000000',
      statusBarContentColor: '#21312D',
      navigationBarContentColor: '#21312D'
    }).catch((err: BusinessError) => {
      hilog.error(DOMAIN, 'immersive', 'setWindowSystemBarProperties failed: %{public}s', JSON.stringify(err));
    });

    this.updateAvoidAreas(mainWindow);
    mainWindow.on('avoidAreaChange', (data: window.AvoidAreaOptions) => {
      if (data.type === window.AvoidAreaType.TYPE_SYSTEM) {
        AppStorage.setOrCreate('topAvoidPx', data.area.topRect.height);
      }
      if (data.type === window.AvoidAreaType.TYPE_NAVIGATION_INDICATOR) {
        AppStorage.setOrCreate('bottomAvoidPx', data.area.bottomRect.height);
      }
    });
  }

  private updateAvoidAreas(mainWindow: window.Window): void {
    const systemArea: window.AvoidArea = mainWindow.getWindowAvoidArea(window.AvoidAreaType.TYPE_SYSTEM);
    const navigationArea: window.AvoidArea =
      mainWindow.getWindowAvoidArea(window.AvoidAreaType.TYPE_NAVIGATION_INDICATOR);

    AppStorage.setOrCreate('topAvoidPx', systemArea.topRect.height);
    AppStorage.setOrCreate('bottomAvoidPx', navigationArea.bottomRect.height);
  }
}
```

## 页面侧布局

页面读取 `AppStorage` 中的避让区。避让区数值来自 `window.AvoidArea` 的 rect 高度,通常按 px 处理,页面尺寸需要转换为 vp。

> 当前 SDK 仍可用全局 `px2vp()`,但会有弃用警告。若项目已统一使用 UIContext 的单位换算或 LengthMetrics,优先沿用项目约定。

```ts
@Entry
@Component
struct Home {
  @StorageProp('topAvoidPx') topAvoidPx: number = 0;
  @StorageProp('bottomAvoidPx') bottomAvoidPx: number = 0;

  build() {
    Column() {
      Row() {
        Text('首页')
      }
      .padding({ top: 18 + px2vp(this.topAvoidPx), left: 16, right: 16, bottom: 10 })

      Scroll() {
        Column() {
          // content
        }
      }
      .layoutWeight(1)

      this.BottomNavigation()
    }
    .width('100%')
    .height('100%')
    .backgroundColor('#F8F3EA')
    .expandSafeArea([SafeAreaType.SYSTEM], [SafeAreaEdge.TOP, SafeAreaEdge.BOTTOM])
  }

  @Builder BottomNavigation() {
    Row() {
      Text('首页').layoutWeight(1).textAlign(TextAlign.Center)
      Text('作品').layoutWeight(1).textAlign(TextAlign.Center)
      Text('我的').layoutWeight(1).textAlign(TextAlign.Center)
    }
    .width('100%')
    .height(64 + px2vp(this.bottomAvoidPx))
    .padding({ bottom: px2vp(this.bottomAvoidPx) })
    .backgroundColor('#FFFFFF')
  }
}
```

## Stack 浮动底栏模式

如果底部导航或播放器控制条用 `Stack` 浮在底部,必须给滚动内容留出底部空间,否则列表末尾会被遮挡。

```ts
Stack() {
  Scroll() {
    Column() {
      // content
    }
    .padding({ bottom: 80 + px2vp(this.bottomAvoidPx) })
  }

  BottomBar()
    .align(Alignment.Bottom)
}
```

普通页面更推荐用 `Column { Scroll().layoutWeight(1); BottomBar() }`,布局更稳。

## 深色页面系统栏图标

播放器、视频页、深色全屏页进入时要切换系统栏内容色,离开时恢复。

```ts
import { BusinessError } from '@kit.BasicServicesKit';
import { window } from '@kit.ArkUI';

@Entry
@Component
struct Player {
  aboutToAppear(): void {
    this.setSystemBarContentColor('#FFFFFF');
  }

  aboutToDisappear(): void {
    this.setSystemBarContentColor('#21312D');
  }

  private setSystemBarContentColor(color: string): void {
    const context = this.getUIContext().getHostContext();
    if (context === undefined) {
      return;
    }

    window.getLastWindow(context).then((mainWindow: window.Window) => {
      mainWindow.setWindowSystemBarProperties({
        statusBarColor: '#00000000',
        navigationBarColor: '#00000000',
        statusBarContentColor: color,
        navigationBarContentColor: color
      });
    }).catch((err: BusinessError) => {
      console.error(`Failed to set system bar style: ${JSON.stringify(err)}`);
    });
  }
}
```

## 自检清单

- Ability 加载首页成功后才调用窗口 API。
- 已调用 `setWindowLayoutFullScreen(true)`。
- 已读取 `TYPE_SYSTEM` 和 `TYPE_NAVIGATION_INDICATOR`。
- 已注册 `avoidAreaChange` 并更新状态。
- 背景使用 `expandSafeArea`,内容使用 top/bottom 避让 padding。
- 底部导航条高度和 padding 包含底部避让区。
- 深色页面进入/离开时切换系统栏内容色。
- 构建后检查 DevEco 是否提示“只开启沉浸式未处理避让区”的规则告警。
