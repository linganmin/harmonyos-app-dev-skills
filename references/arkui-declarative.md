# ArkUI 声明式 UI 与状态管理

> 由 `harmonyos-app-dev` 使用。组件、属性、装饰器随版本演进,**以官方文档为准**:
> - ArkUI 概述:`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkui-overview`
> - 状态管理、组件参考:在文档中心"应用框架 → ArkUI"下查找(用浏览器工具读取,WebFetch 抓不到正文)。

## 组件的基本形态

```ets
@Entry
@Component
struct Index {
  @State count: number = 0
  build() {
    Column() {
      Text(`计数:${this.count}`).fontSize(20)
      Button('加一').onClick(() => { this.count++ })
    }
    .width('100%')
    .padding(16)
  }
}
```

- **`@Entry`**:页面入口(一个页面文件一个);**`@Component`**:可复用组件;**`@Reusable`**:可回收复用(长列表性能)。
- **`build()`**:描述 UI,**根部应是单个容器组件**(如 `Column`)。
- 属性是**链式调用**(`.width().height()`);容器组件用**尾随闭包 `{}`** 承载子组件。
- 自定义组件名首字母大写,参数经构造传入:`MyCard({ title: 'x' })`。

## 状态管理 V1

核心思想:**状态变化 → 自动重新执行受影响的 `build` → 刷新 UI**。改普通成员变量**不会**刷新。

| 装饰器 | 作用 | 方向 |
| --- | --- | --- |
| `@State` | 组件私有可变状态 | 自身 |
| `@Prop` | 父传子,单向(值拷贝) | 父→子 |
| `@Link` | 父子双向绑定(引用) | 双向 |
| `@Provide` / `@Consume` | 跨层级共享,无需逐层透传 | 祖先→后代 |
| `@Observed` + `@ObjectLink` | 观察**对象/嵌套属性**变化 | 配对使用 |
| `@Watch('cb')` | 状态变更时回调 | — |
| `@StorageLink` / `@StorageProp` | 绑定全局 `AppStorage` | — |
| `@LocalStorageLink` / `@LocalStorageProp` | 绑定页面级 `LocalStorage` | — |

**最常见的"不刷新"陷阱**:`@State obj` 改 `obj.inner.field` 不触发刷新——把内层类标 `@Observed`,子组件用 `@ObjectLink` 接收该内层对象。数组/对象的深层变更尤其要注意。

**装饰器的 key 必须是字符串字面量**:`@StorageLink('currentRole')`、`@StorageProp('bp')`、`@Provide('x')` 等的 key **不能用常量/变量**(写 `@StorageLink(Keys.ROLE)` 不通过)。这与"键名集中常量管理"存在张力——只有 `PersistentStorage.persistProp(...)`、`AppStorage.setOrCreate(...)` 这些 API 调用处能用常量,装饰器处只能硬编码同名字面量,二者要保持一致。

## 状态管理 V2(较新,API 12+)

华为推出 V2:`@ComponentV2`、`@Local`、`@Param`、`@Once`、`@Event`、`@ObservedV2` + `@Trace`、`@Monitor`、`@Provider` / `@Consumer`,对深层观测和组件通信更清晰。

**V1 与 V2 不能在同一组件混用。** 用哪套取决于工程约定与 API 版本——动手前**先确认现有代码用的是哪套**,再保持一致。新建工程可查官方"状态管理 V2"决定。

## 布局与常用组件

- 线性:`Column` / `Row`(配 `justifyContent` / `alignItems`);层叠:`Stack`;弹性:`Flex`;相对:`RelativeContainer`。
- 列表:`List` + `ListItem`;**长列表用 `LazyForEach`**(配 `IDataSource`,按需创建节点);网格 `Grid` / `GridRow`。
- 滚动 `Scroll`;轮播 `Swiper`;分页 `Tabs` + `TabContent`。**`Scroll` 内容不足一屏时默认垂直居中**(`align` 默认 Center),要从顶部渲染的页面必须显式 `.align(Alignment.Top)`。
- 下拉刷新 `Refresh({ refreshing: $$this.x }) { Scroll(...) }.onRefreshing(cb)`:刷新动画**不会自动收起**,请求完成后所有分支(成功/失败/早退)都要手动 `this.x = false`。
- 渲染控制:`if / else if / else`、`ForEach`(小数据集)、`LazyForEach`(大数据集)。
- **ForEach 键值生成函数决定是否重渲染**:键值相同的项被视为"数据未变化",直接复用旧 UI——即使数据源换成了内容不同的新对象。因此键值只用 `id` 时,编辑/刷新后该行不会更新。小列表用 `(item) => JSON.stringify(item)`(内容变 → key 变 → 重建该行);长列表或高频局部更新用 `@Observed` class + `@ObjectLink` 子组件做属性级观察。

## 导航

- **推荐 `Navigation`**:`NavPathStack` 管理路由栈,`NavDestination` 定义目标页,原生支持一次开发多端(自适应单/双栏)。
  - **构造**:`Navigation(this.pathStack)`,其中 `pathStack: NavPathStack = new NavPathStack()`。
  - **自适应单/双栏**:`.mode(NavigationMode.Auto)` 按宽度自动切单(`Stack`)/双(`Split`)栏;要按断点精确控制,订阅断点值(`@StorageProp` + 媒体查询)后据此返回 `NavigationMode.Stack`(窄屏)/`NavigationMode.Split`(宽屏 lg/xl)。
  - **子页注册有两种**:① `.navDestination(this.PageMap)`,`PageMap` 是 `@Builder(name: string)`,`if/else` 分发到各 `NavDestination` 组件——**免配置、适合骨架/小规模**;② 系统路由表 `router_map.json`(`module.json5` 配 `routerMap`)+ `pushPathByName('name', param)`——**更解耦、适合规模化与跨模块**。
  - **跳转/回退**:`pathStack.pushPath({ name })` 或 `pushPathByName(name, param)`(param 可传 `null`);`pathStack.pop()` 回退。子页用 `.onReady((ctx) => this.pathStack = ctx.pathStack)` 或 `@Consume` 拿到栈。
- 旧 `@kit.ArkUI` 的 `router`(`router.pushUrl`)仍可用,但新应用优先 `Navigation`。
- **router → Navigation 渐进迁移模式**(存量 router 工程加新页面时验证过):
  - 主框架 `@Entry` 页内用 `Navigation(stack)` 包住底部 `Tabs`,新页面全部做成 `NavDestination` 走 `PageMap`;存量 `@Entry` 页(登录/引导等应用级流程)暂留 router,两套并存互不干扰。
  - `NavPathStack` 放进**单例 service**(如 `NavService.getInstance().stack`),任意深度的子组件直接 `push(name, param)` / `pop()`,不用逐层透传也不依赖 `@Consume`。
  - NavDestination 取参用 `.onReady((ctx: NavDestinationContext) => ctx.pathInfo.param)`——`PageMap` 的 `@Builder (name: string)` 拿不到参数。
  - **全局悬浮件(迷你播放器等)必须放在 Navigation 外层的 Stack 上**:放在 Navigation 首页内容里,push 子页后会被 NavDestination 盖住。
- 弹窗/浮层:`CustomDialog`、`bindSheet`(半模态)、`bindContentCover`(全屏)、`promptAction.showToast` / `showDialog`。
  - **一个节点只能绑一个 `bindSheet`**:同一节点链式绑定多个会相互顶替(点 B 弹出 A、原弹窗失效)。多弹窗用「单 bindSheet + `@State activeSheet` 切换 builder 内容」。
  - `height: SheetSize.FIT_CONTENT`(API 11+)随内容自适应,要求 builder **根节点高度不能用百分比**;内部滚动区改用 `constraintSize({ maxHeight })` 限高。
  - sheet 内自定义组件与宿主通信:`@Link` 双向 + 父组件 `@State @Watch` 监听事件字符串可用;每次打开用自增 `sessionId`(`@Prop @Watch`)触发表单重置,因为 builder 内组件只创建一次不会重走 `aboutToAppear`。

## 生命周期

- 组件:`aboutToAppear()`(创建后、build 前)/ `aboutToDisappear()`(销毁前)。
- 页面(`@Entry`):`onPageShow()` / `onPageHide()` / `onBackPress()`。

## 资源、单位与复用

- 资源引用:`$r('app.string.name')`、`$r('app.media.icon')`、`$r('app.color.brand')`、`$rawfile('config.json')`。字符串/颜色/尺寸入 `resources/base/element`,图片入 `media`。
- 尺寸单位:**`vp`**(虚拟像素,默认数值即 vp)、**`fp`**(字体,随系统字号);避免写死像素。
- UI 复用:`@Builder`(可复用构建函数)、`@BuilderParam`(组件插槽)、`@Styles` / `@Extend`(样式复用)、`@AnimatableExtend`(可动画属性)。
