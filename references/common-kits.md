# 常用 Kit 选型速查(按"我要做什么"组织)

> 由 `harmonyos-app-dev` 使用。先用本表把需求映射到正确的 Kit,再到 `official-docs-index.md` 取 URL,用浏览器工具读当前 API。**不要凭印象写 Kit 接口。**

## 按场景选 Kit

| 我要做… | 用哪个 Kit / 能力 | 备注 |
| --- | --- | --- |
| HTTP 请求 / 上传下载 / WebSocket | **Network Kit**(`@kit.NetworkKit` 的 `http`、`request`、`webSocket`) | 端云传输主通道 |
| 轻量键值存储(配置/开关) | **ArkData → 用户首选项 Preferences** | 类似 SharedPreferences |
| 结构化/关系型本地库 | **ArkData → 关系型数据库 RelationalStore**(SQLite) | 复杂查询、事务 |
| 分布式 KV / 跨设备同步数据 | **ArkData → 分布式 KV Store** | 多端协同 |
| 读写应用/用户文件 | **Core File Kit** | 沙箱文件、文档选择 |
| 敏感小数据(令牌/密码) | **Asset Store Kit** | 安全存储,优于明文首选项 |
| 本地通知(提醒/角标) | **Notification Kit** | 端侧发布 |
| 远程推送(服务端下发) | **Push Kit** | 需服务端 + AGC 配置 |
| 后台还要继续干活 | **Background Tasks Kit** | 短时任务 / 长时任务 / 延迟任务,受系统约束 |
| 重计算并行 | **Function Flow Runtime(FFRT)** / TaskPool | 见 `arkts-language.md` 并发 |
| 华为账号一键登录 | **Account Kit** | 拿匿名手机号/UnionID 等需授权 |
| 应用内支付 / 订阅 | **Payment Kit**(华为支付)、**IAP Kit**(应用内购买/订阅) | 商品/订阅用 IAP |
| 定位 | **Location Kit** | 需位置权限 |
| 地图展示 | **Map Kit** | |
| 播放/录制音视频 | **Media Kit**(`AVPlayer`/`AVRecorder`)、**Audio Kit**、**AVCodec Kit** | 编解码用 AVCodec |
| 拍照/相机 | **Camera Kit** | |
| 选相册图片/视频 | **Media Library Kit**(`photoAccessHelper`) | |
| 图片解码/编辑 | **Image Kit** | |
| 扫码 | **Scan Kit** | 系统级扫码入口 |
| 桌面卡片 | **Form Kit**(`FormExtensionAbility`) | 见 Stage 模型扩展组件 |
| 跨应用分享内容 | **Share Kit** | |
| 日历/日程、联系人 | **Calendar Kit**、**Contacts Kit** | |
| 生物识别(指纹/人脸/锁屏密码) | **User Authentication Kit** | |
| 加解密 / 密钥管理 | **Crypto Architecture Kit**、**Universal Keystore(HUKS)** | |
| 蓝牙 / NFC / Wi-Fi P2P | **Connectivity Kit** | 短距通信 |
| 星闪 | **NearLink Kit** | 低功耗高速短距 |
| 传感器 / 振动 | **Sensor Service Kit** | |
| 电话/短信能力 | **Telephony Kit** | |
| 端侧语音(TTS/ASR) | **Core Speech Kit**;朗读/字幕控件用 **Speech Kit** | |
| 端侧视觉(OCR/人脸等) | **Core Vision Kit** / **Vision Kit** | |
| 端侧模型推理 | **MindSpore Lite Kit** / Neural Network Runtime | |
| 云函数/云数据库/云存储 | **Cloud Foundation Kit** + AGC 云开发(Serverless) | 见文档索引 AGC 段 |
| 运动健康数据 | **Health Service Kit** | 基于华为账号授权 |
| 实时状态上屏(订单/配送) | **Live View Kit**(实况窗) | |
| 应用测试(单元/UI) | **Test Kit**(arkxtest:Hypium 单元 + UI 测试) | 见 `quality-and-release.md` |

## 与本项目(慢病伴)强相关的能力

慢病伴是家庭慢病协同 App,以下能力大概率会用到,选型时优先核对:

- **账号登录** → Account Kit(华为账号一键登录)。
- **蓝牙血压计/血糖仪对接** → Connectivity Kit(BLE)。
- **服药提醒** → Notification Kit + Background Tasks Kit(到点可靠触发);远程提醒可叠加 Push Kit。
- **本地记录持久化** → ArkData 关系型数据库(指标/用药记录),首选项存设置。
- **复诊资料包 PDF** → PDF Kit(生成/批注)。
- **订阅付费** → IAP Kit(订阅)/ Payment Kit。
- **朗读(适老端 TTS)** → Core Speech Kit / Speech Kit。

> 这些只是选型指引;具体 API、权限名、回调签名仍以官方文档为准。
