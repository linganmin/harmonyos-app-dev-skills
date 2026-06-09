# HarmonyOS 应用开发技能 / HarmonyOS App Development Skill

---

## 🇨🇳 简体中文

### 📖 项目简介

这是一个专为 **HarmonyOS NEXT** 原生应用开发设计的 AI 技能包，帮助开发者使用 **ArkTS + ArkUI** 构建高质量的鸿蒙应用与元服务。

本技能的核心理念是：**先查文档，再写代码**。鸿蒙生态演进快速，API 在版本间常有变化，因此本技能提供正确的工程范式、API 心智模型，以及一套权威官方文档的检索路径，而不是依赖易过时的 API 记忆。

### ✨ 核心特性

- **🎯 精准选型**：快速定位到正确的 Kit、组件和 API
- **📚 文档优先**：提供华为官方文档的结构化索引和高效检索方法
- **⚡ 避免踩坑**：总结高频陷阱和关键约束（ArkTS 语言限制、状态管理、权限模型等）
- **🔧 完整工作流**：覆盖从需求分析、工程配置、代码编写到发布上架的全流程
- **🧩 模块化知识**：按主题分类的 references 文档，按需查阅，避免信息过载

### 📂 目录结构

```
harmonyos-app-dev/
├── SKILL.md                           # 技能定义与核心工作流
├── evals/                             # 评估测试用例
│   └── evals.json
└── references/                        # 分类参考文档
    ├── official-docs-index.md         # 华为官方文档索引（必读）
    ├── arkts-language.md              # ArkTS 语言特性与约束
    ├── arkui-declarative.md           # ArkUI 声明式 UI 与状态管理
    ├── stage-and-project.md           # Stage 模型与工程配置
    ├── common-kits.md                 # 常用 Kit 选型指南
    ├── third-party-packages.md        # 第三方包管理与审查
    ├── mpchart.md                     # MPChart 图表库使用
    ├── immersive-system-bars.md       # 沉浸式状态栏/导航栏实现
    └── quality-and-release.md         # 质量管控与应用发布
```

### 🚀 快速开始

#### 适用场景

当你遇到以下任何情况时，应使用本技能：

- ✅ 开发或调试鸿蒙 App / 元服务
- ✅ 编写 `.ets` 文件（ArkTS 代码）
- ✅ 配置鸿蒙工程（`module.json5`、`oh-package.json5`）
- ✅ 设计 UIAbility 或实现声明式 UI（`@Component`、`@Entry`、`@State`）
- ✅ 集成华为 Kit（Network、ArkData、Notification、Push、Map、Payment 等）
- ✅ 排查 HarmonyOS 构建、签名或上架问题

#### 标准开发工作流

1. **拆解需求 → 映射到能力**  
   明确功能需要哪些 UI 组件、本地数据能力、网络请求、系统/服务 Kit

2. **查文档**  
   打开 `references/official-docs-index.md` 定位相关条目，用 context7 或浏览器工具读取当前 API 签名与示例

3. **对齐工程结构**  
   确认模块位置、`module.json5` 配置、ohpm 依赖，参考 `references/stage-and-project.md`

4. **编写代码**  
   - UI/状态管理：`references/arkui-declarative.md`
   - 语言约束：`references/arkts-language.md`
   - Kit 集成：`references/common-kits.md`

5. **自检验证**  
   检查 ArkTS 约束、状态装饰器、权限声明，涉及发布参考 `references/quality-and-release.md`

### 📋 技能覆盖范围

| 领域 | 内容 |
|------|------|
| **语言** | ArkTS 核心特性、类型系统、并发模型（Worker/TaskPool）、与 TypeScript 的差异 |
| **UI 框架** | ArkUI 声明式范式、组件系统、状态管理（@State/@Prop/@Link/@Provide 等）、布局容器、列表、导航/路由 |
| **应用模型** | Stage 模型、UIAbility 生命周期、Want、Context、应用启动流程 |
| **工程配置** | DevEco Studio、hvigor 构建、module.json5/app.json5 配置、ohpm 包管理、签名打包 |
| **系统能力** | Network Kit、ArkData（首选项/关系型/KV）、Notification、Account、Push、Universal Kit 等 |
| **第三方生态** | ohpm 包中心、依赖管理、安全审查、MPChart 等常用库 |
| **质量与发布** | 权限模型、性能/功耗优化、隐私合规、AppGallery Connect 上架 |

### ⚠️ 高频陷阱

- **把 ArkTS 当 TypeScript 写**：禁用 `any`、对象字面量类型、动态属性、随意 `as` 转换
- **状态不刷新**：装饰器使用错误，如修改普通成员变量或 `@State` 对象深层属性未使用 `@Observed/@ObjectLink`
- **UI 加载位置错误**：应在 `onWindowStageCreate` 中通过 `windowStage.loadContent()` 加载，而非 `onCreate`
- **沉浸式只铺满不避让**：必须同时获取 `avoidArea` 并监听变化，实际内容需用避让高度做 padding
- **权限声明不完整**：`module.json5` 声明 + `user_grant` 权限运行时申请，缺一不可
- **编造 API**：不确定的接口签名必须查文档确认，避免返工

### 🔍 文档检索优先级

1. **context7（首选）**：查询稳定常见 API，高效节省 token
   - `/websites/developer_huawei_consumer_cn_doc_harmonyos-guides`（指南）
   - `/websites/developer_huawei_consumer_cn_doc_harmonyos-references`（API 参考）

2. **浏览器工具（兜底）**：最新 Kit、边缘 API 或需核对最新版本时使用
   - 华为文档站是 SPA，`WebFetch` 抓不到内容，需用 Playwright 等浏览器工具

### 🤝 贡献指南

欢迎提交 Issue 或 Pull Request 来完善本技能：

- 更新过时的 API 信息
- 补充新增的 Kit 或组件说明
- 添加更多实战案例和最佳实践
- 修正文档错误或改进表述

### 📄 许可证

MIT License

---

## 🇬🇧 English

### 📖 Overview

This is an AI skill package designed for **HarmonyOS NEXT** native app development, helping developers build high-quality HarmonyOS applications and atomic services using **ArkTS + ArkUI**.

The core principle of this skill is: **Check documentation first, write code second**. The HarmonyOS ecosystem evolves rapidly with frequent API changes across versions. Therefore, this skill provides correct engineering paradigms, API mental models, and a structured pathway to authoritative official documentation, rather than relying on potentially outdated API memories.

### ✨ Key Features

- **🎯 Precise Selection**: Quickly locate the right Kits, components, and APIs
- **📚 Documentation First**: Provides structured index and efficient retrieval methods for Huawei official docs
- **⚡ Avoid Pitfalls**: Summarizes common traps and critical constraints (ArkTS language restrictions, state management, permission model, etc.)
- **🔧 Complete Workflow**: Covers the full process from requirement analysis, project configuration, coding to release
- **🧩 Modular Knowledge**: Topic-based references for on-demand consultation, avoiding information overload

### 📂 Directory Structure

```
harmonyos-app-dev/
├── SKILL.md                           # Skill definition & core workflow
├── evals/                             # Evaluation test cases
│   └── evals.json
└── references/                        # Categorized reference docs
    ├── official-docs-index.md         # Huawei official docs index (must-read)
    ├── arkts-language.md              # ArkTS language features & constraints
    ├── arkui-declarative.md           # ArkUI declarative UI & state management
    ├── stage-and-project.md           # Stage model & project configuration
    ├── common-kits.md                 # Common Kits selection guide
    ├── third-party-packages.md        # Third-party package management
    ├── mpchart.md                     # MPChart charting library usage
    ├── immersive-system-bars.md       # Immersive status/navigation bar implementation
    └── quality-and-release.md         # Quality control & app release
```

### 🚀 Getting Started

#### Use Cases

Use this skill when you encounter any of the following situations:

- ✅ Developing or debugging HarmonyOS apps / atomic services
- ✅ Writing `.ets` files (ArkTS code)
- ✅ Configuring HarmonyOS projects (`module.json5`, `oh-package.json5`)
- ✅ Designing UIAbility or implementing declarative UI (`@Component`, `@Entry`, `@State`)
- ✅ Integrating Huawei Kits (Network, ArkData, Notification, Push, Map, Payment, etc.)
- ✅ Troubleshooting HarmonyOS build, signing, or release issues

#### Standard Development Workflow

1. **Break Down Requirements → Map to Capabilities**  
   Identify required UI components, local data capabilities, network requests, system/service Kits

2. **Check Documentation**  
   Open `references/official-docs-index.md` to locate relevant entries, use context7 or browser tools to read current API signatures and examples

3. **Align Project Structure**  
   Confirm module location, `module.json5` configuration, ohpm dependencies, refer to `references/stage-and-project.md`

4. **Write Code**  
   - UI/State management: `references/arkui-declarative.md`
   - Language constraints: `references/arkts-language.md`
   - Kit integration: `references/common-kits.md`

5. **Self-Check & Verification**  
   Check ArkTS constraints, state decorators, permission declarations, refer to `references/quality-and-release.md` for release-related items

### 📋 Skill Coverage

| Domain | Content |
|--------|---------|
| **Language** | ArkTS core features, type system, concurrency (Worker/TaskPool), differences from TypeScript |
| **UI Framework** | ArkUI declarative paradigm, component system, state management (@State/@Prop/@Link/@Provide, etc.), layout containers, lists, navigation/routing |
| **Application Model** | Stage model, UIAbility lifecycle, Want, Context, app launch flow |
| **Project Config** | DevEco Studio, hvigor build, module.json5/app.json5 config, ohpm package management, signing & packaging |
| **System Capabilities** | Network Kit, ArkData (Preferences/RelationalDB/KV), Notification, Account, Push, Universal Kit, etc. |
| **Third-Party Ecosystem** | ohpm package hub, dependency management, security review, common libraries like MPChart |
| **Quality & Release** | Permission model, performance/power optimization, privacy compliance, AppGallery Connect publishing |

### ⚠️ Common Pitfalls

- **Treating ArkTS as TypeScript**: Forbidden use of `any`, object literal types, dynamic properties, arbitrary `as` casting
- **State Not Refreshing**: Incorrect decorator usage, such as modifying plain member variables or deep properties of `@State` objects without `@Observed/@ObjectLink`
- **Wrong UI Loading Location**: Should load in `onWindowStageCreate` via `windowStage.loadContent()`, not in `onCreate`
- **Immersive Without Avoidance**: Must obtain `avoidArea` and listen for changes, actual content needs padding with avoidance heights
- **Incomplete Permission Declaration**: Both `module.json5` declaration + runtime request for `user_grant` permissions required
- **Inventing APIs**: Always check documentation for uncertain interface signatures to avoid rework

### 🔍 Documentation Retrieval Priority

1. **context7 (Preferred)**: Query stable common APIs, efficient and token-saving
   - `/websites/developer_huawei_consumer_cn_doc_harmonyos-guides` (Guides)
   - `/websites/developer_huawei_consumer_cn_doc_harmonyos-references` (API Reference)

2. **Browser Tools (Fallback)**: Use for latest Kits, edge-case APIs, or when verification is needed
   - Huawei docs site is a SPA, `WebFetch` can't retrieve content, use Playwright or similar browser tools

### 🤝 Contributing

Contributions are welcome! Submit Issues or Pull Requests to improve this skill:

- Update outdated API information
- Add documentation for new Kits or components
- Contribute more practical cases and best practices
- Fix documentation errors or improve descriptions

### 📄 License

MIT License

---

<div align="center">

**Made with ❤️ for HarmonyOS Developers**

</div>
