---
name: ai-writing-assist
description: >-
  通过 Flyelep AI 帮写 API 辅助生成创意文案，可用于优化用户提示词。
  当用户要求生成文案、优化提示词、获取创意灵感时使用此技能。
---
# Flyelep AI帮写
通过 Flyelep AI Tool API 辅助生成创意文案，可用于优化用户提示词或获取创意灵感。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**

## API 接口信息
- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/assistedGeneration`
- **Content-Type**: `application/json`
- **超时时间**: 建议 60-120 秒

## 认证方式
所有 AI 工具接口均需在请求头中传入 `secretKey`。该密钥需由用户在 Flyelep 开放平台申请获得：https://www.flyelep.cn/controlboard 。

请求头示例：

```http
Content-Type: application/json
secretKey: 用户提供的API密钥
```

> **安全说明**：`secretKey` 必须放在请求头中，这是 AI 工具接口的统一鉴权要求。不要将真实密钥写入技能文件、示例代码仓库或持久化配置中，应在运行时由用户动态提供。

## 请求 Body
```json
{
  "query": "用户需求描述，最多1000个字符",
  "fileUrlList": ["https://example.com/img1.png", "https://example.com/img2.png"],
  "generateType": "image"
}
```

## 响应格式
成功：
```json
{
  "code": 200,
  "data": {
    "options": [
      "创意文案选项1",
      "创意文案选项2",
      "创意文案选项3",
      "创意文案选项4"
    ]
  }
}
```
- `code=200` 表示调用成功
- `data.options` 为 AI 生成的多个创意文案选项数组
- 将所有选项展示给用户供其选择

失败：
```json
{
  "code": 500,
  "msg": "错误信息"
}
```

## 参数说明
### 必传参数
| 字段 | 默认值 | 说明 |
|------|--------|------|
| query | - | 用户需求描述，最多1000个字符长度 |

### 可选参数
| 字段 | 默认值 | 说明 |
|------|--------|------|
| fileUrlList | - | 文件URL地址数组，最多上传6张，建议单张图片大小在10MB以内 |
| generateType | - | 生成类型：`image`或空表示图片生成，`video`表示视频生成 |

## 参数映射规则
### generateType（生成类型）
- `image` 或不传：图片生成场景
- `video`：视频生成场景

### fileUrlList（参考文件）
- 数组格式，支持多个URL
- 最多6个文件
- 用于提供参考图片帮助AI生成更精准的文案

### 结果处理
- `data.options` 是一个数组，包含多个备选文案
- 每个选项是一个完整的创意文案，通常包含产品卖点、使用场景、用户群体、背景风格等信息
- 将所有选项展示给用户，让用户选择或作为创作参考

## 调用示例
- **重要**：在 Windows/PowerShell 环境下调用 API 时，必须采用以下流程：**先将请求体 JSON 写入当前工作目录下的临时文件 `payload_temp.json`，再通过 Shell 工具调用 `curl.exe --data-binary @payload_temp.json` 发送请求**。这是因为 PowerShell 使用 GBK 编码，而服务端使用 UTF-8 解析，直接在命令行中嵌入中文 JSON 会导致乱码。同时必须设置 `Content-Type: application/json; charset=utf-8` 请求头。
- **注意**：在 Windows 环境下使用 `curl.exe`（而非 `curl`，后者在 PowerShell 中是 `Invoke-WebRequest` 的别名）。必须在 `curl.exe` 后加 `--%` 停止 PowerShell 解析，否则 `@` 会被误判为 splatting 操作符导致报错。
- **文件创建方式**：根据可用工具选择其一（均需确保 UTF-8 **无 BOM** 编码，否则服务端 JSON 解析会在 position 0 报错）：
  - **方式 A（有 Write 工具）**：使用 Write 工具创建 `payload_temp.json`
  - **方式 B（无 Write 工具）**：使用 Shell 的 .NET API 创建文件（`Set-Content -Encoding UTF8` 会带 BOM，不可用）
- **清理**：API 返回结果后，务必删除 `payload_temp.json` 临时文件。

**示例 1：基础文案生成**

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "query": "钓鱼竿"
}
```
> 方式 B（无 Write 工具）：
> ```powershell
> $json = '{"query":"钓鱼竿"}'
> [System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))
> ```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/assistedGeneration" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**示例 2：带参考图的文案优化（图片场景）**

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "query": "为这款蓝牙耳机生成电商文案",
  "fileUrlList": ["https://example.com/product1.png", "https://example.com/product2.png"],
  "generateType": "image"
}
```
> 方式 B（无 Write 工具）：
> ```powershell
> $json = '{"query":"为这款蓝牙耳机生成电商文案","fileUrlList":["https://example.com/product1.png","https://example.com/product2.png"],"generateType":"image"}'
> [System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))
> ```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/assistedGeneration" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**示例 3：视频场景文案生成**

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "query": "为这款智能手表生成短视频脚本文案",
  "generateType": "video"
}
```
> 方式 B（无 Write 工具）：
> ```powershell
> $json = '{"query":"为这款智能手表生成短视频脚本文案","generateType":"video"}'
> [System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))
> ```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/assistedGeneration" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

## 常见错误及解决方案
| 错误 | 原因与解决 |
|------|-----------|
| HTTP 401 / `code` 非 200 | `secretKey` 无效、缺失或已过期，确认请求头是否正确传入 |
| HTTP 405 Not Allowed | 请求方法错误，必须使用 `POST` |
| query 超过1000字符 | 缩短 query 内容 |
| 服务繁忙（9999错误码） | 稍后重试 |
| fileUrlList 文件过多 | 最多上传6个文件 |
| 单张图片超过10MB | 压缩图片大小 |

## 提示词处理
当用户需要：
- 优化产品描述、生成电商文案
- 获取创意灵感、润色提示词
- 为海报或视频生成配套文案

收集用户的基础描述，如有参考图片则一并上传，调用此接口获取多个创意选项展示给用户。

**AI帮写结果通常包含以下结构：**
- 产品卖点
- 使用场景
- 用户群体
- 背景风格建议

这些信息可直接用于后续的图片或视频生成任务。
