# 质量、权限与发布

> 由 `harmonyos-app-dev` 使用。流程与字段以官方为准,相关 URL 见 `official-docs-index.md` 的"工具与体验/质量""AppGallery Connect"段。

## 权限模型(最容易出 bug)

权限是**声明 + 申请两步**,缺一不可:

1. **声明**:在 `module.json5` 的 `requestPermissions` 列出权限名(如 `ohos.permission.INTERNET`、`ohos.permission.LOCATION`)。
2. **按等级处理**:
   - **system_grant(系统授予)**:安装即生效,声明即可(如 INTERNET)。
   - **user_grant(用户授予)**:必须**运行时弹窗申请**——用 `abilityAccessCtrl.createAtManager().requestPermissionsFromUser(context, [...])`,并处理用户拒绝/永久拒绝的分支。
3. 部分高敏权限需 **ACL** 或更高 APL 等级应用方可申请;敏感权限要在隐私说明中如实披露。

**原则**:最小权限。只申请功能真正需要的;能用 system-picker(如相册选择、文档选择)免权限完成的,就不要申请敏感权限。

## 测试(arkxtest)

- **单元测试**:Hypium 框架,断言业务逻辑。
- **UI 测试**:UiTest,模拟点击/滑动验证界面。
- 通过 **Test Kit** 接入,DevEco Studio 可运行。详见 `official-docs-index.md` → Test Kit / 应用测试。

## 体验四项(华为有专门的"体验建议"文档,上架会被牵引)

- **性能**:不要阻塞主线程(耗时操作放 TaskPool/异步);长列表用 `LazyForEach` + `@Reusable`;关注冷启动时延、丢帧率。
- **稳定性**:消除崩溃(crash)、卡死(freeze)、应用无响应;用 Performance Analysis Kit 抓 hiAppEvent/日志/trace 定位。
- **功耗**:节制后台任务与高频传感器/定位;遵循 Background Tasks Kit 的约束,不要长期占用后台。
- **安全隐私**:最小化数据收集;敏感数据用 Asset Store / HUKS 存储而非明文;提供清晰的隐私声明与撤销路径。

> 慢病伴这类健康应用对隐私与合规尤其敏感:健康数据收集、家庭共享、撤销与删除都要在隐私说明中讲清,并遵守"安全隐私体验建议"。

## 构建产物与签名

- 产物:**HAP**(部署单元)、**HAR**(静态库)、**HSP**(动态共享包)。
- 调试:DevEco Studio **自动签名**即可在真机/模拟器运行。
- 发布:在 AppGallery Connect 申请**发布证书 + Profile**,配置到 `build-profile.json5` 的 `signingConfigs`,再打**发布版**包。

## 上架流程(AppGallery Connect)

1. 在 AGC 创建项目与应用,拿到 `bundleName`(与 `app.json5` 一致)。
2. 配置发布签名证书与 Profile。
3. 用发布证书构建 App 包(`.app`,内含 HAP/HSP)。
4. 在 AGC 填写应用信息、上传包、提交审核;元服务/游戏有各自的发布入口。
5. 维护:版本更新、升级、下架、删除均在 AGC。

详细步骤见 `official-docs-index.md` → AppGallery Connect 段的"发布应用/维护应用"。
