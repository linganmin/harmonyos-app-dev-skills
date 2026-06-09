# 官方文档 URL 索引

> 由 `harmonyos-app-dev` 使用。
> **查 API 第一步优先 context7**:多数 ArkTS/ArkUI/Kit 文档已被 context7 的两个华为官方库收录,用它查签名 / 范式最省 token:
> - `/websites/developer_huawei_consumer_cn_doc_harmonyos-guides`(指南 / 范式)
> - `/websites/developer_huawei_consumer_cn_doc_harmonyos-references`(API 参考)
>
> **context7 未命中、或需按最新版核对**时,再用下面的 URL 索引定位,**用浏览器工具打开**(华为文档是 JS 渲染,`WebFetch` 抓不到正文);链接抓取自华为开发者文档中心(`developer.huawei.com/consumer/cn/doc/`,HarmonyOS 5.0 及以上),URL 可能随版本调整,失效时用 `WebSearch` 搜 "HarmonyOS <能力名> 文档" 重新定位。

大多数指南的 URL 形如:
`https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/<slug>`
下文 **G:** 表示该 base,只列 `<slug>`;其他 base 给出完整路径。

## 目录
- [入口与基础](#入口与基础)
- [应用框架](#应用框架)
- [系统](#系统)
- [媒体](#媒体)
- [图形](#图形)
- [应用服务](#应用服务)
- [AI](#ai)
- [专题](#专题)
- [元服务](#元服务)
- [工具与体验/质量](#工具与体验质量)
- [AppGallery Connect(云/分发)](#appgallery-connect云分发)

## 入口与基础
- 文档中心首页:`https://developer.huawei.com/consumer/cn/doc/`
- 应用开发导读:G:`application-dev-guide`
- 版本说明:`https://developer.huawei.com/consumer/cn/doc/harmonyos-releases/overview-allversion`
- 快速入门:G:`start-overview`
- 应用开发基础知识:G:`development-fundamentals`
- 学习 ArkTS 语言:G:`arkts-get-started`
- 资源分类与访问:G:`resource-categories-and-access`

## 应用框架
| Kit | slug(G:) | 用途 |
| --- | --- | --- |
| Ability Kit | `abilitykit-overview` | 程序框架/应用模型(UIAbility 等) |
| Accessibility Kit | `accessibilitykit-overview` | 无障碍 |
| ArkData | `data-mgmt-overview` | 首选项/关系型/KV 数据管理 |
| ArkUI | `arkui-overview` | 声明式 UI 框架 |
| ArkTS | `arkts-overview` | 应用开发语言 |
| ArkWeb | `web-component-overview` | Web 组件 |
| Background Tasks Kit | `background-task-overview` | 后台任务 |
| Core File Kit | `core-file-kit-intro` | 文件基础服务 |
| Data Augmentation Kit | `data-augmentation-kit-guide` | 知识库/RAG/端侧问答 |
| Form Kit | `formkit-overview` | 桌面卡片 |
| IME Kit | `ime-kit-intro` | 输入法 |
| IPC Kit | `ipc-rpc-overview` | 进程间通信 |
| Localization Kit | `i18n-l10n` | 国际化/本地化 |
| UI Design Kit | `ui-design-introduction` | HarmonyOS Design 界面套件 |

## 系统
| Kit | slug(G:) | 用途 |
| --- | --- | --- |
| Asset Store Kit | `asset-store-kit-overview` | 敏感数据安全存储 |
| Basic Services Kit | `basic-services-kit-overview` | 通用基础能力 |
| Connectivity Kit | `connectivity-kit-intro` | 蓝牙/NFC 等短距通信 |
| Crypto Architecture Kit | `crypto-architecture-kit-intro` | 加解密/签名 |
| Data Loss Prevention Kit | `dlp-overview` | 数据防泄漏 |
| Device Certificate Kit | `device-certificate-kit-intro` | 证书算法/管理 |
| Device Security Kit | `devicesecurity-introduction` | 设备状态/安全检测 |
| Distributed Service Kit | `distributedservice-kit-intro` | 分布式设备/硬件管理 |
| Driver Development Kit | `driverdevelopment-overview` | 外设驱动开发 |
| FAST Kit | `fast-introduction` | 高性能算法/数据结构 |
| Function Flow Runtime Kit | `ffrt-overview` | 并发任务调度 |
| Input Kit | `input-overview` | 多模输入 |
| MDM Kit | `mdm-kit-intro` | 企业设备管理 |
| Multimodal Awareness Kit | `multimodalawareness-kit-intro` | 多模态感知 |
| NearLink Kit | `nearlink-introduction` | 星闪通信 |
| Network Kit | `net-mgmt-overview` | 网络/HTTP/WebSocket |
| Network Boost Kit | `networkboost-introduction` | 网络加速/质量预测 |
| Online Authentication Kit | `onlineauthentication-introduction` | 免密认证 |
| Pen Kit | `pen-introduction` | 手写笔 |
| Performance Analysis Kit | `performance-analysis-kit-overview` | 事件/日志/trace 分析 |
| Remote Communication Kit | `remote-communication-introduction` | HTTP 请求 NAPI 封装 |
| Sensor Service Kit | `sensorservice-kit-intro` | 传感器/振动 |
| Service Collaboration Kit | `servicecollaborationkit-introduction` | 多端协同 |
| Telephony Kit | `telephony-overview` | 蜂窝通信 |
| Test Kit | `arkxtest-guidelines-0000001930756221` | 单元/UI 自动化测试 |
| Universal Keystore Kit | `huks-overview` | 密钥管理(HUKS) |
| User Authentication Kit | `user-authentication-overview` | 指纹/人脸/锁屏密码 |
| Wear Engine Kit | `we-business_introduction` | 华为穿戴设备能力 |

## 媒体
| Kit | slug(G:) | 用途 |
| --- | --- | --- |
| Audio Kit | `audio-kit-intro` | 音频播放/录制 |
| AVCodec Kit | `avcodec-kit-intro` | 音视频编解码/封装 |
| AVSession Kit | `avsession-overview` | 音视频播控 |
| Camera Kit | `camera-overview` | 相机 |
| DRM Kit | `drm-overview` | 数字版权保护 |
| Image Kit | `image-overview` | 图片解析/处理 |
| Media Kit | `media-kit-intro` | AVPlayer/AVRecorder |
| Media Library Kit | `photoaccesshelper-overview` | 相册/媒体文件管理 |
| Scan Kit | `scan-introduction` | 统一扫码 |
| Ringtone Kit | `ringtone-introduction` | 铃声设置 |

## 图形
| Kit | slug(G:) | 用途 |
| --- | --- | --- |
| AR Engine | `arengine-overview` | 增强现实 |
| ArkGraphics 2D | `arkgraphics2d-introduction` | 2D 图形绘制 |
| ArkGraphics 3D | `arkgraphics3d-overview` | 3D 场景绘制 |
| Graphics Accelerate Kit | `graphics-accelerate-introduction` | 图形加速/防卡顿 |
| Spatial Recon Kit | `spatial-recon-introduction` | 3D 内容处理/建模 |
| XEngine Kit | `xengine-kit-introduction` | GPU 加速引擎 |

## 应用服务
| Kit | slug(G:) | 用途 |
| --- | --- | --- |
| Account Kit | `account-introduction` | 华为账号登录 |
| Ads Kit | `ads-introduction` | 广告变现 |
| AppGallery Kit | `store-introduction` | 应用市场能力 |
| App Linking Kit | `applinking-introduction` | 延迟链接/直达 |
| Calendar Kit | `calendarmanager-overview` | 日历/日程 |
| Call Service Kit | `call-introduction` | VoIP 通话管理 |
| Cloud Foundation Kit | `cloudfoundation-introduction` | 云函数/云数据库/云存储 |
| Contacts Kit | `contacts-intro` | 联系人 |
| File Manager Service Kit | `filemanagerservice-introduction` | 文件管理 |
| Health Service Kit | `health-service-kit-ability` | 运动健康数据 |
| IAP Kit | `iap-introduction` | 应用内购买/订阅 |
| Game Service Kit | `gameservice-introduction` | 游戏服务 |
| Live View Kit | `liveview-introduction` | 实况窗 |
| Location Kit | `location-kit-intro` | 定位 |
| Map Kit | `map-introduction` | 地图 |
| Notification Kit | `notification-overview` | 本地通知 |
| Payment Kit | `payment-introduction` | 华为支付 |
| PDF Kit | `pdf-introduction` | PDF 打开/批注/水印 |
| Preview Kit | `preview-introduction` | 文件预览 |
| Push Kit | `push-kit-introduction` | 远程消息推送 |
| Reader Kit | `reader-introduction` | 电子书阅读 |
| Scenario Fusion Kit | `scenario-fusion-introduction` | 场景化融合组件 |
| Share Kit | `share-introduction` | 跨应用分享 |
| Wallet Kit | `wallet-introduction` | 钱包 |
| Weather Service Kit | `weather-service-introduction` | 天气数据 |

## AI
| Kit | slug(G:) | 用途 |
| --- | --- | --- |
| Agent Framework Kit | `hmaf-introduction` | 应用内拉起智能体 |
| Core Speech Kit | `core-speech-introduction` | 语音基础(TTS/ASR) |
| Core Vision Kit | `core-vision-introduction` | 机器视觉基础 |
| CANN Kit | `hiaifoundation-introduction` | 芯片使能/算子 |
| Intents Kit | `intents-introduction` | 意图框架 |
| MindSpore Lite Kit | `mindspore-lite-kit-introduction` | 端侧推理引擎 |
| Natural Language Kit | `natural-language-introduction` | 自然语言理解 |
| Neural Network Runtime Kit | `neural-network-runtime-kit-introduction` | 跨芯片推理运行时 |
| Speech Kit | `speech-production` | 朗读/字幕控件 |
| Vision Kit | `vision-introduction` | 场景化视觉 |

## 专题
- 一次开发,多端部署:G:`foreword`
- 自由流转:G:`distributed-overview`
- NDK 开发(C/C++):G:`ndk-development-overview`
- 最佳实践:`https://developer.huawei.com/consumer/cn/doc/best-practices/bpta-best-practices-overview`

## 元服务
- 版本说明:`https://developer.huawei.com/consumer/cn/doc/atomic-releases/atomic-releasenotes`
- 开发指南:`https://developer.huawei.com/consumer/cn/doc/atomic-guides/atomic-service-definition`
- API 参考:`https://developer.huawei.com/consumer/cn/doc/atomic-references/atomic-apis-intro`
- FAQ:`https://developer.huawei.com/consumer/cn/doc/atomic-faqs/faqs-operational`

## 工具与体验/质量
- DevEco Studio 使用指南:G:`ide-tools-overview`
- DevEco Service(ohpm-repo 私仓):G:`ide-ohpm-repo-overview`
- 应用测试:G:`app-testing-overview-0000001198515507`
- 兼容性体验建议:G:`compatibility-overview`
- 稳定性体验建议:G:`experience-suggestions-stability`
- 性能体验建议:G:`performance-overview`
- 功耗体验建议:G:`app-power-experience-standards-overview`
- 安全隐私体验建议:G:`security-privacy-experience-standards`
- 应用 FAQ:`https://developer.huawei.com/consumer/cn/doc/harmonyos-faqs/faqs-ux-design`

## AppGallery Connect(云/分发)
- 认证服务:`https://developer.huawei.com/consumer/cn/doc/app/agc-help-auth-0000002236336998`
- 云存储:G:`cloudfoundation-storage-service`
- 云数据库:G:`cloudfoundation-database-service`
- 云函数:G:`cloudfoundation-function-service`
- 预加载:G:`cloudfoundation-prefetch-service`
- 发布应用:`https://developer.huawei.com/consumer/cn/doc/app/agc-help-release-app-0000002271695230`
- 发布元服务:`https://developer.huawei.com/consumer/cn/doc/app/agc-help-release-atomic-0000002327731065`
- 维护应用/元服务:`https://developer.huawei.com/consumer/cn/doc/app/agc-help-maintain-0000002270829401`
