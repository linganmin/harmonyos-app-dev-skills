# 阿里云 OSS 文件上传集成

## 概述

在 HarmonyOS NEXT 应用中实现文件直传到阿里云 OSS 的标准方案。使用官方 `@aliyun/oss` SDK + STS 临时授权，提供安全、高效的文件上传能力。

## 技术方案

### 架构设计

```
┌─────────────┐   1. 请求临时凭证    ┌─────────────┐
│  鸿蒙客户端  │ ──────────────────▶ │  业务服务器  │
│             │ ◀──────────────────  │             │
└─────────────┘   2. 返回 STS Token  └─────────────┘
      │                                     │
      │ 3. 使用临时凭证直传               │ 调用 STS
      ▼                                     ▼
┌─────────────┐                      ┌─────────────┐
│  阿里云 OSS  │ ◀──────────────────  │  阿里云 STS  │
└─────────────┘                      └─────────────┘
```

**核心优势**：
- ✅ 使用官方 SDK，符合"标准化 + 生态复用"原则
- ✅ STS 临时授权，不暴露永久密钥
- ✅ 支持重试、进度回调等企业级能力

## 快速开始

### 1. 安装依赖

```bash
ohpm install @aliyun/oss
```

**包信息**：
- 官方包：`@aliyun/oss` 1.0.0-beta.1
- 文档：https://help.aliyun.com/zh/oss/developer-reference/harmony/

### 2. 使用封装工具类

项目已封装好 `OSSUploader` 工具类，位于 `common/src/main/ets/utils/OSSUploader.ets`

**基础用法**：

```typescript
import { OSSUploader } from '../utils/OSSUploader';

// 创建上传器
const uploader = new OSSUploader(
  'https://your-api.com/api/v1/sts-token',  // STS 凭证接口
  'user-token'                              // 用户认证 Token
);

// 简单上传
const fileUrl = await uploader.uploadFile(localPath, remotePath);

// 带进度回调
const fileUrl = await uploader.uploadFile(localPath, remotePath, {
  onProgress: (progress) => {
    console.log(`上传进度: ${progress}%`);
  },
  maxRetries: 3  // 最大重试次数
});

// 生成符合规范的远程路径
const remotePath = OSSUploader.generateRemotePath(
  userId,           // 用户 ID
  'recording.mp3',  // 原始文件名
  'audio'           // 路径前缀
);
// 输出: audio/123/1733745600_abc123.mp3
```

### 3. 完整业务示例

**声音录音上传服务**：

```typescript
import { OSSUploader } from '../utils/OSSUploader';
import { Logger } from '../utils/Logger';
import { common } from '@kit.AbilityKit';

export class VoiceRecordingService {
  private uploader: OSSUploader;
  
  constructor(context: common.UIAbilityContext) {
    this.uploader = new OSSUploader(
      'https://api.neara.com/api/v1/sts-token'
    );
  }
  
  async uploadRecording(
    localFilePath: string,
    userId: number,
    voiceName: string,
    onProgress?: (progress: number) => void
  ): Promise<string> {
    // 生成远程路径
    const remotePath = OSSUploader.generateRemotePath(
      userId,
      'recording.mp3',
      'voices'
    );
    
    // 上传文件
    const fileUrl = await this.uploader.uploadFile(
      localFilePath,
      remotePath,
      { onProgress, maxRetries: 3 }
    );
    
    // 调用业务接口创建声音记录
    await this.createVoiceRecord(voiceName, fileUrl, userId);
    
    return fileUrl;
  }
  
  private async createVoiceRecord(
    voiceName: string,
    audioUrl: string,
    userId: number
  ): Promise<void> {
    // POST /api/v1/voices
    const response = await fetch('https://api.neara.com/api/v1/voices', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.getUserToken()}`
      },
      body: JSON.stringify({
        voice_name: voiceName,
        gender: 2,
        audio_url: audioUrl,
        sample_text: this.getSampleText()
      })
    });
    
    const result = await response.json();
    if (result.code !== 0) {
      throw new Error(result.message);
    }
  }
}
```

**在页面中使用**：

```typescript
@Entry
@Component
struct VoiceLabPage {
  private service: VoiceRecordingService = 
    new VoiceRecordingService(getContext(this) as common.UIAbilityContext);
  
  @State uploadProgress: number = 0;
  @State isUploading: boolean = false;
  
  async handleUpload(localFilePath: string, voiceName: string) {
    try {
      this.isUploading = true;
      
      const fileUrl = await this.service.uploadRecording(
        localFilePath,
        123,  // userId
        voiceName,
        (progress) => {
          this.uploadProgress = progress;
        }
      );
      
      promptAction.showToast({ message: '上传成功' });
    } catch (error) {
      promptAction.showToast({ message: `上传失败: ${error.message}` });
    } finally {
      this.isUploading = false;
    }
  }
  
  build() {
    Column() {
      if (this.isUploading) {
        Progress({ 
          value: this.uploadProgress, 
          total: 100, 
          type: ProgressType.Linear 
        })
          .width('80%')
          .margin(20)
      }
      
      Button('开始上传')
        .onClick(() => {
          this.handleUpload('/cache/recording.mp3', '妈妈的声音');
        })
    }
  }
}
```

## 后端 STS 接口规范

### 接口设计

**地址**：`GET /api/v1/sts-token`

**请求头**：
```
Authorization: Bearer {用户Token}
```

**响应格式**：
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "accessKeyId": "STS.xxxxxxxx",
    "accessKeySecret": "xxxxxxxx",
    "securityToken": "CAISxxxxxxxx",
    "expiration": "2026-06-09T15:00:00Z",
    "region": "oss-cn-hangzhou",
    "bucket": "your-bucket-name"
  }
}
```

### Node.js 实现示例

```typescript
import STS from '@alicloud/sts-sdk';

export async function getStsToken(req, res) {
  const userId = req.user.id;
  
  const stsClient = new STS({
    accessKeyId: process.env.ALIBABA_CLOUD_ACCESS_KEY_ID,
    accessKeySecret: process.env.ALIBABA_CLOUD_ACCESS_KEY_SECRET,
  });

  try {
    const token = await stsClient.assumeRole(
      'acs:ram::YOUR_ACCOUNT_ID:role/upload-role',
      null,
      3600,  // Token 有效期 1 小时
      `user-${userId}`
    );

    res.json({
      code: 0,
      message: 'success',
      data: {
        accessKeyId: token.Credentials.AccessKeyId,
        accessKeySecret: token.Credentials.AccessKeySecret,
        securityToken: token.Credentials.SecurityToken,
        expiration: token.Credentials.Expiration,
        region: 'oss-cn-hangzhou',
        bucket: 'your-bucket-name'
      }
    });
  } catch (error) {
    res.status(500).json({ code: 500, message: error.message });
  }
}
```

### RAM 角色权限配置

在阿里云 RAM 控制台创建角色，授予以下权限策略：

```json
{
  "Version": "1",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "oss:PutObject",
        "oss:GetObject"
      ],
      "Resource": [
        "acs:oss:*:*:your-bucket/audio/${user_id}/*",
        "acs:oss:*:*:your-bucket/image/${user_id}/*"
      ]
    }
  ]
}
```

**权限最小化原则**：
- ✅ 仅允许 PutObject（上传）和 GetObject（读取）
- ✅ 限制上传路径为用户专属目录
- ❌ 不授予 DeleteObject、ListBucket 等权限

## OSSUploader API 参考

### 构造函数

```typescript
constructor(stsEndpoint: string, userToken?: string)
```

### 主要方法

| 方法 | 说明 | 返回值 |
|------|------|--------|
| `uploadFile(localPath, remotePath, options?)` | 简单上传 | Promise\<string\> |
| `uploadLargeFile(localPath, remotePath, options?)` | 分片上传（待实现） | Promise\<string\> |
| `smartUpload(localPath, remotePath, fileSize, options?)` | 智能上传（自动选择策略） | Promise\<string\> |
| `setUserToken(token)` | 设置用户 Token | void |
| `destroy()` | 销毁客户端 | void |

### 静态方法

| 方法 | 说明 | 返回值 |
|------|------|--------|
| `generateRemotePath(userId, fileName, prefix?)` | 生成远程路径 | string |

### UploadOptions 配置

```typescript
interface UploadOptions {
  onProgress?: (progress: number) => void;  // 进度回调 0-100
  maxRetries?: number;                      // 最大重试次数，默认 3
  useMultipart?: boolean;                   // 是否使用分片上传
  partSize?: number;                        // 分片大小，默认 5MB
}
```

## 错误处理

### 常见错误

| 错误信息 | 原因 | 处理方式 |
|---------|------|---------|
| `获取上传凭证失败` | 网络异常或服务端错误 | 提示用户重试 |
| `不支持的文件格式` | 文件格式不在白名单 | 提示支持的格式 |
| `InvalidAccessKeyId` | AccessKey 无效 | 自动刷新凭证 |
| `SecurityTokenExpired` | Token 过期 | 自动刷新凭证 |
| `NetworkError` | 网络错误 | 自动重试 |

### 错误处理示例

```typescript
try {
  const fileUrl = await uploader.uploadFile(localPath, remotePath);
} catch (error) {
  if (error.message.includes('获取上传凭证失败')) {
    promptAction.showToast({ message: '网络异常，请稍后重试' });
  } else if (error.message.includes('不支持的文件格式')) {
    promptAction.showToast({ message: '仅支持 mp3/wav/m4a/aac 格式' });
  } else {
    promptAction.showToast({ message: `上传失败: ${error.message}` });
  }
}
```

## 安全规范（强制要求）

⚠️ **永远不要**在客户端硬编码 AccessKey ID 和 AccessKey Secret  
⚠️ **必须**使用 STS 临时凭证进行上传  
⚠️ **必须**在服务端验证用户身份后才返回 STS Token  
⚠️ **必须**限制 STS Token 的权限范围（仅 PutObject，限定路径）  
⚠️ **建议**为不同用户设置独立的上传路径（如 `audio/{userId}/`）

## 性能优化

### 上传策略选择

| 文件大小 | 推荐方式 | 说明 |
|---------|---------|------|
| < 5MB | 简单上传 | 直接 PUT Object |
| 5MB - 100MB | 简单上传 | 适合大部分录音场景 |
| > 100MB | 分片上传 | 支持断点续传 |

### 优化建议

- ✅ 缓存 STS Token 直到过期前 5 分钟（工具类自动处理）
- ✅ 使用 OSS 加速域名（CDN）
- ✅ 选择距离用户最近的 Region
- ✅ 失败时使用指数退避重试策略（工具类自动处理）

## 工具类特性

`OSSUploader` 工具类已内置以下企业级能力：

1. **自动凭证管理**：STS Token 过期前 5 分钟自动刷新
2. **智能重试**：网络错误自动重试 3 次，指数退避
3. **凭证错误自动恢复**：Token 失效时自动刷新后重试
4. **文件格式验证**：仅允许 mp3/wav/m4a/aac 格式
5. **进度回调**：支持实时上传进度通知
6. **资源管理**：支持手动销毁客户端释放资源

## 相关文档

- **详细指南**：`docs/harmonyos/oss-upload-guide.md`（项目级文档）
- **工具类源码**：`common/src/main/ets/utils/OSSUploader.ets`
- **阿里云官方文档**：https://help.aliyun.com/zh/oss/developer-reference/harmony/
- **STS 临时授权**：https://help.aliyun.com/zh/oss/developer-reference/authorize-access-6

## 注意事项

1. **凭证管理**：OSSUploader 会自动管理 STS Token 的刷新，无需手动处理
2. **文件格式**：默认仅支持 mp3/wav/m4a/aac，可通过修改工具类扩展
3. **路径规范**：建议使用 `generateRemotePath` 方法生成符合规范的路径
4. **错误重试**：网络错误会自动重试 3 次，其他错误不重试
5. **资源释放**：长期不使用时调用 `destroy()` 方法释放资源

---

**最后更新**：2026-06-09  
**适用版本**：HarmonyOS NEXT API 12+
