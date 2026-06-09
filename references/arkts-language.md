# ArkTS 语言要点(与 TypeScript 的关键差异)

> 由 `harmonyos-app-dev` 使用。ArkTS 随版本演进,**具体规则以官方文档为准**:
> - 从 TS 到 ArkTS 的适配规则:`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/typescript-to-arkts-migration-guide`
> - 入门:`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-get-started`
> - 概述:`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-overview`

## 为什么 ArkTS 不是 TypeScript

ArkTS 在 TS 基础上**做减法**:移除会拖慢运行时、妨碍静态优化(AOT 编译)的动态特性,换取性能与确定性。把它当普通 TS 写,是鸿蒙编译报错的头号来源。下面这些约束不是风格建议,是**编译期硬规则**。

## 必须遵守的核心约束

**1. 禁止 `any` / `unknown`** — 用具体类型、联合类型或泛型。
```ts
let v: any = getData()           // ❌
let v: UserInfo = getData()      // ✅
```

**2. 对象必须有明确类型** — 不能用无类型对象字面量当"通用袋子";字面量要能匹配已声明的 `class`/`interface`。
```ts
let p = { name: 'A', age: 1 }                    // ❌ 无类型字面量
interface Person { name: string; age: number }
let p: Person = { name: 'A', age: 1 }            // ✅
```

**3. 禁止运行时改变对象结构** — 不能动态增删属性、`delete obj.x`。需要动态键值用 `Map` 或 ArkTS `collections`。
```ts
obj.newField = 1                 // ❌
let m = new Map<string, number>(); m.set('k', 1) // ✅
```

**4. 名义类型(nominal),不是结构化类型** — 类型兼容看声明而非"形状一样";实现接口要显式 `implements`。

**5. 限制类型断言 `as`** — 不能用 `as` 做不安全转换;优先类型守卫(`instanceof` / 自定义守卫)。

**6. 用 `let` / `const`,禁止 `var`。**

**7. 禁止 `eval`、`Function` 构造、运行时代码生成。**

**8. 限制 `globalThis`、`arguments`、函数 `this` 重绑定**(`apply`/`call`/`bind` 的部分用法受限);用箭头函数保留词法 `this`。

**9. 索引/动态键受限** — `obj[dynamicKey]` 受限;用 `Map`、`Record<K,V>`(按文档)替代动态键对象。

## 文件与模块

- **`.ets`**:ArkTS + UI(含 ArkUI 装饰器),页面/组件必须用 `.ets`。`.ts` 可写纯逻辑。
- ES Module:`import` / `export`;鸿蒙 Kit 统一从 `@kit.*` 导入,如 `import { http } from '@kit.NetworkKit'`、`import { window } from '@kit.ArkUI'`。

## 异步与并发

- **IO/异步**:单线程事件循环 + `async`/`await` / `Promise`。
- **CPU 密集 / 并行**:
  - **TaskPool**(推荐):任务级并发,自动调度回收,适合短时任务。
  - **Worker**:长驻独立线程,`postMessage` 通信,适合持续后台计算。
- **跨线程数据**:普通对象跨线程是**拷贝**传递;要真正共享须用 `@Sendable` 标注的 Sendable 对象、或 ArkTS `collections`(Sendable 容器)。

## 错误处理

`try/catch`;系统接口失败抛 `BusinessError`(带 `code`/`message`),据 `code` 分支处理。异步用 `await` 包 `try/catch` 或 `.catch()`。

## 标准能力速记

- 容器:`Array` / `Map` / `Set`;高性能可共享容器在 `@kit.ArkTS` 的 `collections`。
- 工具:`@kit.ArkTS`(`util`、`JSON`、`process` 等)。
- 不确定某 API 的导入路径或签名时,**去文档确认 `@kit.*` 归属**,不要猜。
