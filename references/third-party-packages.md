# 第三方包与 ohpm 包中心

> 由 `harmonyos-app-dev` 在需要找三方库、比较包、安装依赖或升级依赖时读取。包中心入口: `https://ohpm.openharmony.cn/#/cn/home`。

## 查找第三方包

**推荐资源**：

- **官方包中心**: `https://ohpm.openharmony.cn/#/cn/home` — 权威来源，实时更新
- **社区精选列表**: `https://hu-qi.github.io/ohpm-awesome/` — 社区整理的优质第三方包分类索引，涵盖 UI 组件、工具库、网络、图表、动画等常用场景，适合快速浏览和选型参考

**查找流程**：

1. 打开 ohpm 包中心首页: `https://ohpm.openharmony.cn/#/cn/home`。
2. 在搜索框输入能力关键词或包名:
   - 能力词: `chart`、`二维码`、`calendar`、`network`、`image`。
   - 包名: `@ohos/mpchart`。
   - 同时尝试中英文关键词,很多包使用英文标签。
3. 进入候选包详情页后先看这些字段:
   - **包名**: `@scope/name`,安装和 import 以这里为准。
   - **最新版本 / 更新时间**:优先选仍在维护的包。
   - **兼容 SDK / API**:确认不低于当前工程 target/compatible SDK,也不要依赖明显过旧 API。
   - **License**:确认能被当前项目使用,优先 Apache-2.0、MIT 等清晰许可。
   - **仓库 / 文档 / README**:优先选有示例、变更记录、Issue 入口的包。
   - **依赖项与权限**:看是否引入额外 Kit、系统能力、权限、native 库或体积风险。
4. 用命令行交叉确认元信息:

```bash
ohpm info <package-name>
ohpm info @ohos/mpchart
```

如果官网是动态页面、普通 fetch 看不到正文,用浏览器工具打开页面读取可访问性树;如果只是确认包元信息,`ohpm info` 通常更直接。

## 安装与依赖落点

在**实际消费这个包的 module 目录**执行安装,不要默认在根目录装:

```bash
cd features/patient
ohpm install @ohos/mpchart
```

安装后检查:

- 当前 module 的 `oh-package.json5` 写入了 `dependencies`。
- 当前 module 的 `oh-package-lock.json5` 更新了解析版本。
- 根工程或对应 module 构建能找到该包。

多模块工程里,谁 import 这个三方包,谁声明依赖。不要为了方便把 feature 专用包放进 `common`,除非多个模块确实共享这套能力。

## 选型原则

- 优先使用 HarmonyOS/OpenHarmony 原生 ArkTS/ArkUI 包,避免 Web/npm 包直接迁移。
- 优先选包名、README、仓库、License、兼容 SDK 都清晰的包。
- 医疗、支付、账号、隐私、加密、地图等敏感能力优先官方 Kit;三方包只做 UI 或非关键辅助逻辑。
- 先做最小验证页面或组件,确认能 `hvigorw assembleHap` 通过,再扩散到业务页面。
- 新增包后记录构建警告。三方包内部 ArkTS warning 可以接受,但不能掩盖业务代码错误。

## 常用命令

```bash
ohpm install
ohpm install <package-name>
ohpm uninstall <package-name>
ohpm info <package-name>
ohpm list
ohpm dependency-check
ohpm update <package-name>
ohpm clean
```

DevEco Studio 工程建议使用其自带工具链:

```bash
export DEVECO_SDK_HOME=/Applications/DevEco-Studio.app/Contents/sdk
export PATH="/Applications/DevEco-Studio.app/Contents/tools/node/bin:/Applications/DevEco-Studio.app/Contents/tools/hvigor/bin:/Applications/DevEco-Studio.app/Contents/tools/ohpm/bin:$PATH"
```

## 提交前检查

- 不提交 `oh_modules`、`.hvigor`、`build`。
- 提交 `oh-package.json5` 和 `oh-package-lock.json5`。
- 运行 `ohpm install` 和 `hvigorw assembleHap`。
- 若依赖只为某个页面服务,确认没有把三方类型泄漏到跨模块公共 API。
