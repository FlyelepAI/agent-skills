---
name: intelligent-extension
description: >-
  通过 Flyelep AI 工具接口对图片进行智能延展，支持批量处理并指定目标比例。
  当用户要求扩图、延展图片边缘、补全画布、适配 16:9/1:1 等比例时使用此技能。
---
# Flyelep 图片智能延展

通过 Flyelep AI Tool API 对一张或多张图片进行智能延展。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**

## API 接口信息

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/intelligentExtension`
- **Content-Type**: `application/json`
- **认证方式**: 在请求头中传入 `secretKey`（密钥需由用户在 Flyelep 开放平台申请：https://www.flyelep.cn/controlboard）
- **超时时间**: 建议 120-300 秒（批量图片处理可能耗时更长）

请求头示例：

```http
Content-Type: application/json
secretKey: 用户提供的API密钥
```

> **安全说明**：不要将真实密钥写入技能文件、示例代码仓库或持久化配置中，应在运行时由用户动态提供。

## 请求 Body

```json
{
  "imgUrlList": [
    "https://example.com/image1.jpg",
    "https://example.com/image2.jpg"
  ],
  "ratio": "16:9"
}
```

## 响应格式

```json
{
  "code": 200,
  "msg": "操作成功",
  "data": [
    "https://example.com/result1.jpg",
    "https://example.com/result2.jpg"
  ]
}
```

- `code=200` 表示调用成功
- `msg` 为接口返回说明
- `data` 为延展后的图片 URL 数组，与 `imgUrlList` 的输入顺序一致
- 返回结果应逐个展示给用户，不要回读图片内容

## 参数说明

### 必传参数

> **重要**：以下必传参数必须通过询问用户获取，agent 不可自行填写。调用本技能时，应先向用户列出必传参数与可选参数表格，由用户确认或提供后再执行。

| 字段 | 默认值 | 说明 |
|------|--------|------|
| imgUrlList | - | 源图片 URL 列表，支持批量传入多张图片 |
| ratio | - | 目标延展比例 |

### 参数映射规则

**imgUrlList**：
- 接口要求传字符串数组，每个元素为一张图片 URL
- 每个链接都应为公网可访问的图片直链，不要传网页地址
- 保留用户原始图片顺序，不要擅自重排 `imgUrlList`
- 如果用户提供本地文件路径，先调用 image-upload 技能上传文件获取公网链接，再填入此参数

**ratio（目标比例）**：
仅支持以下比例：

- `1:1`
- `2:3`
- `3:2`
- `3:4`
- `4:3`
- `9:16`
- `16:9`
- `21:9`

当用户明确指定目标比例时，原样传入 `ratio`。

当用户仅表达“改成横版”或“适配视频封面”等模糊需求时，可按上下文推断最合理的比例：

- 横版内容优先考虑 `16:9`
- 方图内容优先考虑 `1:1`
- 竖版内容优先考虑 `9:16`

如果上下文不足以安全推断，先向用户确认目标比例。

## 调用示例

> **跨平台调用说明**：
> - 请求头必须包含 `Content-Type: application/json; charset=utf-8` 和 `secretKey`
> - **Windows/PowerShell**：因 GBK 编码问题，必须先将 JSON 写入临时文件 `payload_temp.json`（UTF-8 无 BOM），再用 `curl.exe --% --data-binary @payload_temp.json` 发送请求。使用 Write 工具创建文件，或用 .NET API `[System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))`。调用后用 `rm payload_temp.json` 清理。
> - **macOS/Linux**：bash/zsh 默认 UTF-8，可直接内联 JSON：`curl -X POST URL -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --data-binary 'JSON单行内容'`

### 示例 1：单张图片延展为横版 16:9

**前置步骤**：向用户索取图片路径或 URL。如用户提供本地文件，先调用 image-upload 技能上传获取公网链接。

**Windows/PowerShell**：

步骤 1：创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "imgUrlList": [
    "https://example.com/image1.jpg"
  ],
  "ratio": "16:9"
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"imgUrlList":["https://example.com/image1.jpg"],"ratio":"16:9"}', [System.Text.UTF8Encoding]::new($false))
```

步骤 2：执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/intelligentExtension" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/intelligentExtension" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary '{"imgUrlList":["https://example.com/image1.jpg"],"ratio":"16:9"}'
```

### 示例 2：批量图片延展为方图 1:1

**前置步骤**：向用户索取图片路径或 URL。如用户提供本地文件，先调用 image-upload 技能上传获取公网链接。

**Windows/PowerShell**：

步骤 1：创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "imgUrlList": [
    "https://example.com/image1.jpg",
    "https://example.com/image2.jpg"
  ],
  "ratio": "1:1"
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"imgUrlList":["https://example.com/image1.jpg","https://example.com/image2.jpg"],"ratio":"1:1"}', [System.Text.UTF8Encoding]::new($false))
```

步骤 2：执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/intelligentExtension" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/intelligentExtension" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary '{"imgUrlList":["https://example.com/image1.jpg","https://example.com/image2.jpg"],"ratio":"1:1"}'
```

## 常见错误及解决方案

| 错误 | 原因与解决 |
|------|-----------|
| HTTP 401 / `code` 非 200 | `secretKey` 无效、缺失或已过期，确认请求头是否正确传入密钥 |
| HTTP 405 Not Allowed | 请求方法错误，必须使用 `POST` |
| `imgUrlList` 为空 | 至少传入一张待处理图片 URL |
| 比例不支持 | `ratio` 只能使用接口支持的固定枚举值 |
| 处理超时 | 批量图片较多或源图较大时可适当增大超时时间 |
| 返回数组数量异常 | 先核对源图 URL 是否可访问，再重试请求 |

## 执行流程

1. **向用户询问 `secretKey`**（API 密钥必须由用户提供，agent 不可自行填写）
2. 收集用户提供的图片 URL 列表（如用户提供本地文件，先调用 image-upload 技能上传获取公网链接）
3. 确定目标比例 `ratio`
4. 在请求头中传入 `secretKey`，调用接口
5. 按输入顺序展示返回的图片 URL

该接口不接收自然语言提示词，不需要构造额外文案。保留用户原始图片顺序，不要擅自重排 `imgUrlList`。
