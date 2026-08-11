---
name: image-upload
description: >-
  通过 Flyelep 开放接口把本地图片上传到云存储，返回可公网访问的图片直链。
  当用户提供的是本地图片文件而不是 URL，或需要为其它 Flyelep 技能（抠图、翻译、延展、局部重绘等）准备 imgUrls 入参时使用此技能。
---
# Flyelep 图片上传
通过 Flyelep 开放接口把本地图片文件上传到云存储，并返回永久可访问的图片 URL。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**

## API 接口信息
- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/file/upload`
- **Content-Type**: `multipart/form-data`
- **认证方式**: 在请求头中传入 `secretKey`
- **超时时间**: 建议 60-120 秒

## 认证方式
需在请求头中传入 `secretKey`。该密钥需由用户在 Flyelep 开放平台申请获得：https://www.flyelep.cn/controlboard 。

请求头示例：

```http
secretKey: 用户提供的API密钥
```

> **安全说明**：不要将真实密钥写入技能文件、示例代码仓库或持久化配置中，应在运行时由用户动态提供。

## 请求参数
本接口使用 `multipart/form-data` 上传，不是 JSON body。

| 字段 | 类型 | 说明 |
|------|------|------|
| file | 文件 | 图片二进制文件，`multipart/form-data` 方式上传 |

单次请求只能上传一个文件。需要上传多张时并发调用多次，每次得到一个独立 URL。

**不要手动设置 `Content-Type` 请求头**，让 HTTP 客户端自动生成带 boundary 的值，手写会导致服务端解析失败。

## 响应格式
统一响应结构：

```json
{
  "code": 200,
  "msg": null,
  "data": {
    "relativePath": "cos_ai_agent/2026-08-11/3f2a9c1b7d84e6f5a012.png",
    "fullPath": "https://agent-1404002717.cos.ap-guangzhou.myqcloud.com/cos_ai_agent/2026-08-11/3f2a9c1b7d84e6f5a012.png",
    "serviceProvider": null
  }
}
```

- `code=200` 表示调用成功
- **判断成功只看 `code`，不要只看 HTTP 状态码**：业务失败时 HTTP 仍是 200，`code` 会是 500，原因在 `msg` 里
- `data.fullPath` 为完整可访问 URL，永久有效、不带签名、不会过期
- `data.relativePath` 为对象存储的内部 key

## 参数映射规则
### 该用哪个返回字段
- 展示给用户、或作为其它 Flyelep 技能的 `imgUrls` / `imageUrl` 入参，一律用 `fullPath`
- `relativePath` 是存储侧内部标识，除非接口明确要求，否则不要传

### 图片格式限制
- 仅支持 `bmp`、`gif`、`jpg`、`jpeg`、`png`
- 后缀取自文件名，取不到时回退到 Content-Type
- 其它格式（含 `webp`）会被直接拒绝，需先转成 `png` 或 `jpg` 再传

### 内容审核
图片会先压缩再走内容审核，审核不通过则整个请求失败，不会产生半成品文件。审核加上传通常几秒，超时不要设得太短。

## 调用示例
本接口没有 JSON body，因此不存在中文编码问题，无需创建临时文件。

- **Windows/PowerShell 环境**：使用 `curl.exe`（而非 `curl`，后者在 PowerShell 中是 `Invoke-WebRequest` 的别名）。`-F` 的值用引号包住，`@` 不会被误解析。
- **macOS/Linux 环境**：直接使用 `curl`。

**示例 1：上传单张图片（Windows/PowerShell）**

```bash
curl.exe -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/file/upload" -H "secretKey: 你的密钥" --max-time 120 -F "file=@C:/path/to/product.png"
```

**示例 2：上传单张图片（macOS/Linux）**

```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/file/upload" -H "secretKey: 你的密钥" --max-time 120 -F "file=@./product.png"
```

**示例 3：上传后接着调用其它技能**

1. 上传本地图片，从响应里取出 `data.fullPath`
2. 把该 URL 作为 `imgUrls` 传给抠图、翻译、延展等技能

```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/aiImageMatting" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary '{"imgUrls":"上一步返回的 fullPath"}'
```

## 常见错误及解决方案
| 错误 | 原因与解决 |
|------|-----------|
| `msg` 为 `上传的图片不能为空` | 没带 `file` 字段或文件是 0 字节，检查表单字段名是否为 `file` |
| `msg` 为 `图片格式不支持，仅支持：...` | 后缀不在白名单，先转成 `png` 或 `jpg` |
| `msg` 为 `密钥不能为空!` | 没带 `secretKey` 请求头 |
| `msg` 为 `密钥无效!` | 密钥错误或已失效，向用户重新索取 |
| `msg` 提示图片违规 | 内容审核未通过，需更换图片，重试无效 |
| 服务端解析失败 | 手动设置了 `Content-Type` 导致 boundary 丢失，去掉该请求头 |
| 请求超时 | 图片较大时适当增大超时时间 |

密钥问题、格式问题、审核不通过重试都不会好，不要自动重试。只有网络超时和 5xx 值得重试。

## 使用时机
当用户给的是本地图片文件路径而不是公网 URL 时，先用此技能上传拿到直链，再调用其它 Flyelep 图片处理技能。如果用户已经提供了公网可访问的图片直链，则不需要此技能，直接调用目标技能即可。
