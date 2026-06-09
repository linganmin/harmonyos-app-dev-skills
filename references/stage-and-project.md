# Stage 应用模型与工程结构

> 由 `harmonyos-app-dev` 使用。**只用 Stage 模型**(FA 模型已废弃,新项目不要用)。配置字段随版本演进,**以官方文档为准**:
> - 应用模型 / Ability Kit:`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/abilitykit-overview`
> - 应用开发导读:`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-guide`
> - DevEco Studio:`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ide-tools-overview`

## 应用组件

- **`UIAbility`**:有界面的应用组件,对应"最近任务"里的一个任务。应用至少有一个(常为 `EntryAbility`)。
- **`ExtensionAbility`**:无独立任务的扩展组件,按类型派生——`ServiceExtensionAbility`(后台服务)、`FormExtensionAbility`(桌面卡片)、`InputMethodExtensionAbility`(输入法)、`WorkSchedulerExtensionAbility` 等。
- **`AbilityStage`**:Module 级容器,Module 首次加载的回调入口。

## UIAbility 生命周期(顺序是关键)

```
onCreate(want, launchParam)            // 创建,获取启动参数
  → onWindowStageCreate(windowStage)   // 【在这里 loadContent 加载首页】
      → onForeground / onBackground    // 前后台切换
  → onWindowStageDestroy
→ onDestroy
```

```ets
onWindowStageCreate(windowStage: window.WindowStage): void {
  windowStage.loadContent('pages/Index', (err) => {
    if (err.code) { /* 处理失败 */ }
  })
}
```

**误区**:不要在 `onCreate` 里加载 UI——窗口尚未就绪;页面必须在 `onWindowStageCreate` 拿到 `windowStage` 后加载。

## Want 与 Context

- **`Want`**:组件跳转的载体。**显式**(指定 `bundleName` + `abilityName`)用于应用内/已知目标;**隐式**(按 `action` + `entities` + `uri/type` 匹配)用于"打开某类能力"。`context.startAbility(want)` 发起。
- **`Context`**:能力上下文。`UIAbilityContext`(`startAbility`、`terminateSelf`)、应用级 `Context`(`filesDir`、`resourceManager`、`createModuleContext`)。组件内常用 `getContext(this)` 获取。

## 工程结构(典型 Stage 工程)

```
MyApp/
├── AppScope/app.json5            # 应用级:bundleName、versionCode/Name、icon、label
├── entry/                         # 主 module(type=entry)
│   ├── src/main/
│   │   ├── ets/
│   │   │   ├── entryability/EntryAbility.ets   # UIAbility 实现
│   │   │   └── pages/Index.ets                 # 页面(@Entry)
│   │   ├── resources/             # base/element(色/字符串/尺寸)、media、profile
│   │   └── module.json5           # 本 module 配置(见下)
│   └── oh-package.json5           # 本 module 依赖
├── oh-package.json5               # 工程级依赖
├── build-profile.json5            # 产品 / 签名配置(signingConfigs、products)
└── hvigorfile.ts                  # 构建入口
```

## module.json5 关键字段

```json5
{
  "module": {
    "name": "entry",
    "type": "entry",                 // entry | feature | har | hsp
    "deviceTypes": ["phone", "tablet"],
    "abilities": [{
      "name": "EntryAbility",
      "srcEntry": "./ets/entryability/EntryAbility.ets",
      "exported": true,
      "skills": [{
        "actions": ["ohos.want.action.home"],
        "entities": ["entity.system.home"]
      }]
    }],
    "requestPermissions": [
      { "name": "ohos.permission.INTERNET" }
    ]
  }
}
```

## 包类型

- **HAP**(Harmony Ability Package):**部署 / 安装单元**,由 entry / feature module 编译而来。
- **HAR**:静态共享库,编译期合入,适合复用代码与资源。
- **HSP**:动态共享包,运行期共享,减小包体、供多 HAP 共用。

## 依赖与构建

- **ohpm**:包管理(类比 npm)。`ohpm install <pkg>`;依赖写进 `oh-package.json5` 的 `dependencies`;三方库在 ohpm 中心仓。
- **hvigor**:构建系统。DevEco Studio 图形化构建,或命令行 `hvigorw assembleHap`。
- **签名**:调试期可用 DevEco **自动签名**;发布需在 AppGallery Connect 申请发布证书与 Profile,配到 `build-profile.json5` 的 `signingConfigs`。真机与模拟器运行都需有效签名。

## 多 HAR 模块工程(手动新建 HAR)

大型应用常拆成多个 HAR(如 `common` / `domain` / 各 `feature`),代码与资源按模块复用。**手动**新建一个 HAR 需要下面这些文件(缺一不可,否则 sync / 构建报错):

```
myhar/
├── Index.ets                 # 模块公共 API 边界,只从这里 export;消费方按包名导入
├── build-profile.json5       # apiType:"stageMode" + obfuscation 指向两个 rules.txt + targets:[{name:"default"}]
├── hvigorfile.ts             # export default { system: harTasks }
├── oh-package.json5          # name:"@ohos/xxx", main:"Index.ets", dependencies
├── consumer-rules.txt        # 可空(一行注释即可)
├── obfuscation-rules.txt
└── src/main/
    ├── module.json5          # type:"har", deviceTypes 与 entry 保持一致
    └── resources/base/element/string.json   # 必须非空(≥1 条),空资源会报 11203005
```

- **`BuildProfile.ets`、`oh-package-lock.json5` 由工具生成,不要手建**(`ohpm install` / 构建时产生)。
- **包名 ≠ 模块名**:`oh-package.json5` 的 `name`(`@ohos/xxx`,消费方 `import` 用)与 `module.json5` 的 `name`(模块标识、根 `build-profile` 注册用)是两回事,可以不同。
- **注册到根工程**:在**根** `build-profile.json5` 的 `modules` 数组加 `{ name, srcPath, targets:[{ name:"default", applyToProducts:["default"] }] }`。
- **本地依赖**:消费方 `oh-package.json5` 写 `"@ohos/xxx": "file:<相对路径>"`;按目录层级数清 `../`(如 `./features/a` 引 `./common` 是 `file:../../common`)。
- **跨模块导入走包名**:`import { X } from '@ohos/xxx'`(经 `Index.ets`),不要相对路径跨模块引用。
- **依赖必须单向无环**:基础设施模块当叶子,平级 feature 互不依赖,共享逻辑下沉到公共模块。
- 命令行改完工程结构后可直接 `ohpm install` + `hvigorw assembleHap`;**在 DevEco GUI 里需先点 Sync** 才识别新模块。
- **`.gitignore` 要用 `**/oh_modules`**(官方模板常只忽略根级 `/oh_modules`),否则多模块下 `git add` 会误提交各子模块的依赖软链。
- 首次命令行构建报 `WARN: Missing module info for local modules` 是时序警告、会自愈,`BUILD SUCCESSFUL` 即无害。

## 一次开发,多端部署

鸿蒙强调跨设备自适应:用响应式布局(`GridRow` / `GridCol` 栅格、`breakpoints` 断点、媒体查询)、在 `deviceTypes` 声明目标端,一套代码适配手机 / 平板 / 折叠屏。详见官方"一次开发,多端部署"专题:`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/foreword`。
