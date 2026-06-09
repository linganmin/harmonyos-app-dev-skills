---
name: harmonyos-app-dev
description: >-
  HarmonyOS NEXT / 鸿蒙原生应用开发技能。覆盖 ArkTS 语言、ArkUI 声明式 UI 与状态管理、Stage 应用模型(UIAbility)、
  DevEco Studio / hvigor 工程与 module.json5 / oh-package.json5 配置、华为各类 Kit(Network、ArkData、
  Notification、Account、Push、Map、Payment 等)集成,以及权威官方文档的检索方法。当用户开发或调试鸿蒙 App / 元服务、
  编写 .ets 文件、配置鸿蒙工程、设计 UIAbility、写 @Component/@Entry/@State 等声明式 UI、集成华为 Kit、或排查
  HarmonyOS 构建/签名/上架问题时,必须使用本技能。只要任务涉及 HarmonyOS、鸿蒙、ArkTS、ArkUI、DevEco、
  HarmonyOS NEXT、元服务 / atomic service、.ets、module.json5,即使用户没有明确点名,也应主动触发本技能,
  以获取正确的工程范式、API 心智模型与最新官方文档 URL,避免凭过时记忆编造不存在的 API。
---

# HarmonyOS 鸿蒙应用开发

帮助在 **HarmonyOS NEXT** 上用 **ArkTS + ArkUI** 开发原生应用与元服务。本技能提供正确的工程范式、API 心智模型,以及一套**权威官方文档的检索路径**——而不是把易过时的 API 细节背下来。

## 第一原则:先查文档,再写代码

鸿蒙生态**演进极快**,API、装饰器、配置字段在版本间常有变化。模型记忆中的 ArkTS/ArkUI API 很可能**已经过时或根本不存在**。凭记忆硬写,产出的是看起来合理、实则编译不过的代码。

因此核心工作方式是:**遇到任何具体 API、组件属性、Kit 接口、配置字段,先去官方文档确认当前签名,再动手。** 文档索引见 `references/official-docs-index.md`,它把华为开发者文档中心的 URL 按能力分好类。

### 如何查华为官方文档(重要,有坑)

**第一步优先 context7**:华为官方文档已被 context7 高质量收录,查 ArkTS/ArkUI/组件/Kit 的 API 签名与范式,先用它最省 token、最快(`resolve-library-id` → `query-docs`)。优先这两个**官方**库(High 信誉):

- `/websites/developer_huawei_consumer_cn_doc_harmonyos-guides` —— 指南 / 开发范式
- `/websites/developer_huawei_consumer_cn_doc_harmonyos-references` —— API 参考

权衡:context7 是镜像、可能略滞后。**稳定常见 API**(Navigation/Tabs/状态管理/布局/持久化)用它足够;遇到**最新 Kit、边缘 API、或拿不准是否最新**时,再用浏览器读官网核对。

**兜底:浏览器工具读官网**。华为文档站(`developer.huawei.com/consumer/cn/doc/...`)是**前端 JS 动态渲染的 SPA**:

- ❌ `WebFetch`、`fetch` 类工具**抓不到正文**——只会拿到页面标题和空壳,看起来像"页面没内容"。
- ✅ 用**浏览器工具**(如 Playwright MCP:`browser_navigate` 打开页面 → `browser_snapshot` 取可访问性树 → `Read` 读取 snapshot 文件)才能拿到真实内容。
- ✅ 找不到确切 URL 时,用 `WebSearch` 搜 "HarmonyOS <能力名> 官方文档",再用浏览器打开命中的 `developer.huawei.com` 链接。

把文档检索当成开发的第一步,不是最后的兜底。

## 技术栈心智模型

快速建立正确认知,细节进对应 reference:

| 层 | 是什么 | 关键认知 |
| --- | --- | --- |
| **ArkTS** | 应用主力语言 | TypeScript 的**收紧子集**:为性能和静态优化,**禁用**大量 TS 动态特性(见 `arkts-language.md`)。不要当成普通 TS 写。 |
| **ArkUI** | 声明式 UI 框架 | `@Component struct` + `build()` 描述 UI;状态驱动渲染(`@State/@Prop/@Link/@Provide`…)。范式接近 SwiftUI/Compose,不是 React。 |
| **Stage 模型** | 应用运行模型 | `UIAbility` 是应用组件;UI 在 `onWindowStageCreate` 里加载;能力与权限在 `module.json5` 声明。 |
| **工程/构建** | DevEco + hvigor | `.ets` 源码、`oh-package.json5` 依赖(ohpm)、`module.json5`/`app.json5` 配置、运行需签名。 |

## 标准开发工作流

1. **拆解需求 → 映射到能力**:这个功能要用到 UI?本地数据?网络?某个系统/服务 Kit?先列清单。
2. **查文档**:打开 `references/official-docs-index.md` 定位相关条目 → 用浏览器工具读取**当前** API 签名与示例。
3. **对齐工程结构**:确认放在哪个 module、`module.json5` 是否要声明 ability/权限、是否要加 ohpm 依赖。参见 `references/stage-and-project.md`;涉及第三方包选型、安装、审查时读 `references/third-party-packages.md`。
4. **写代码**:UI/状态按 `references/arkui-declarative.md`;语言写法守 `references/arkts-language.md` 的约束;集成 Kit 按 `references/common-kits.md` 选型。
5. **自检**:ArkTS 严格约束是否违反?状态是否真的会触发刷新?权限是否在 `module.json5` 声明且运行时申请?涉及发布看 `references/quality-and-release.md`。

## references 路由

按需读取,不要一次性全读进来:

- **`references/arkts-language.md`** — 写任何 ArkTS 逻辑、类型、并发(Worker/TaskPool)前。讲清它与 TS 的关键差异和禁用项。
- **`references/arkui-declarative.md`** — 写 UI、状态管理、布局容器、列表、导航/路由、自定义组件时。
- **`references/stage-and-project.md`** — 涉及应用模型(UIAbility 生命周期、Want、Context)、`module.json5`/`app.json5`、ohpm 依赖、hvigor 构建、签名打包时。
- **`references/third-party-packages.md`** — 需要在 ohpm 包中心查找、筛选、安装、升级第三方包时。覆盖 `https://ohpm.openharmony.cn/#/cn/home` 的查包流程、命令行校验、依赖落点与风险审查。
- **`references/mpchart.md`** — 使用 `@ohos/mpchart` 构建折线图、柱状图等图表时。包含安装、LineChart 基础范式、轴/图例/手势/数据刷新、深色模式适配与常见坑。
- **`references/aliyun-oss-upload.md`** — 实现文件直传到阿里云 OSS。基于官方 `@aliyun/oss` SDK + STS 临时授权的完整方案，包含封装工具类、后端接口规范、安全要求与最佳实践。
- **`references/ui-theming-dark-mode.md`** — 实现深色模式适配时:语义色资源配置(base/dark 目录)、系统栏内容色切换、HDS 组件颜色适配、SVG 图标着色、代码改造策略与验收清单。
- **`references/floating-tabs-minibar.md`** — 实现 HDS 浮动标签栏 + 迷你栏组合布局时(如音频/视频播放器):miniBar Builder 写法、barFloatingStyle 配置、渐变蒙版与材料效果、内容区域 padding 计算、响应式适配。
- **`references/common-kits.md`** — 需要网络、数据持久化(首选项/关系型/KV)、通知、账号登录、推送、定位、地图、支付等能力时——先来这里选对 Kit,再查官方文档。
- **`references/official-docs-index.md`** — **每次要查具体 API 都先来这**:华为文档中心按"应用框架/系统/媒体/图形/应用服务/AI/工具/AGC"分类的 URL 索引。
- **`references/quality-and-release.md`** — 权限模型、性能/功耗/稳定性体验、安全隐私合规、应用上架(AppGallery Connect)。

## 高频陷阱(不易从单页文档看出来的)

- **把 ArkTS 当 TS 写**:用 `any`、对象字面量当类型、给对象动态加属性、用 `as` 乱转——在 HarmonyOS NEXT 上会直接报错。先读 `arkts-language.md`。
- **状态不刷新**:UI 不更新,几乎总是状态装饰器用错(改了普通成员变量、或改的是 `@State` 对象的深层属性而没用 `@Observed/@ObjectLink`)。见 `arkui-declarative.md`。
- **UI 加载位置**:在 `UIAbility.onCreate` 里加载页面是错的;页面要在 `onWindowStageCreate(windowStage)` 里通过 `windowStage.loadContent(...)` 加载。
- **沉浸式只铺满、不避让**:调用 `setWindowLayoutFullScreen(true)` 后,必须同时读取 `getWindowAvoidArea(TYPE_SYSTEM/TYPE_NAVIGATION_INDICATOR)` 并监听 `on('avoidAreaChange')`,把顶部/底部避让高度写入状态,页面背景用 `expandSafeArea`,但实际内容、顶部栏、底部导航/按钮要用避让高度做 padding。详见 `references/immersive-system-bars.md`。
- **权限两步缺一**:`module.json5` 声明了权限 ≠ 拿到权限;`user_grant` 级权限还需运行时向用户申请。
- **编造 API**:对不确定的接口签名,宁可去文档确认,也不要"看起来对"地写出来——这是鸿蒙开发里返工最多的来源。
- **多模块 `.gitignore` 漏忽略 `oh_modules`**:官方模板常只写 `/oh_modules`(仅根级),多 HAR 工程要改成 `**/oh_modules`,否则 `git add` 会吞掉各子模块的依赖软链。
- **命令行首次构建的 "Missing module info" 警告**:刚改完工程结构(新增模块)后首次 `hvigorw assembleHap` 必报 `WARN: Missing module info for local modules '...'`,这是 PreBuild 早于 CreateModuleInfo 的**时序警告、会自愈**——最终 `BUILD SUCCESSFUL` 即无害,别误判为错误去瞎改。

> 当某个细节官方文档与本技能描述不一致时,**以当前官方文档为准**,并据此提示更新本技能。
