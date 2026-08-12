---
name: image-enlarge
description: >-
  通过 Flyelep AI 工具接口对图片进行无损放大，支持单张或批量处理。
  当用户要求提升清晰度、放大图片尺寸、做超清增强、批量增强商品图时使用此技能。
---
# Flyelep 无损放大

通过 Flyelep AI Tool API 对图片进行无损放大处理，并返回增强后的图片 URL。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**

## API 接口信息

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/enlarge`
- **Content-Type**: `application/json`
- **认证方式**: 在请求头中传入 `secretKey`（密钥需由用户在 Flyelep 开放平台申请：https://www.flyelep.cn/controlboard）
- **超时时间**: 建议 120-300 秒

请求头示例：

```http
Content-Type: application/json
secretKey: 用户提供的API密钥
```

> **安全说明**：不要将真实密钥写入技能文件、示例代码仓库或持久化配置中，应在运行时由用户动态提供。

## 请求 Body

```json
{
  "imgUrls": "https://example.com/img1.jpg,https://example.com/img2.jpg",
  "scalingRatio": 2
}
```

## 响应格式

```json
{
  "code": 200,
  "msg": "操作成功",
  "data": "https://example.com/enlarged1.jpg,https://example.com/enlarged2.jpg"
}
```

- `code=200` 表示调用成功
- `msg` 为接口返回说明
- `data` 为放大后的图片地址
- 多张图片时，`data` 中多个 URL 以英文逗号 `,` 分隔
- 返回结果应按逗号拆分后逐个展示给用户，不要回读图片内容

## 参数说明

### 必传参数

> **重要**：以下必传参数必须通过询问用户获取，agent 不可自行填写。调用本技能时，应先向用户列出必传参数与可选参数表格，由用户确认或提供后再执行。

| 字段 | 默认值 | 说明 |
|------|--------|------|
| imgUrls | - | 图片链接字符串，多张时使用英文逗号分隔 |
| scalingRatio | - | 放大倍数，取值范围 `1`-`8`，计费档位为 `2`、`4`、`8` |

### 参数映射规则

**imgUrls**：
- 接口要求传字符串，不是数组
- 单张图片时直接传一个 URL 字符串
- 多张图片时，用英文逗号 `,` 按顺序拼接
- 每个链接都应为公网可访问的图片直链，不要传网页地址
- 如果用户提供本地文件路径，先调用 file-upload 技能上传文件获取公网链接，再填入此参数

**scalingRatio**：
- 含义是放大倍数，不是增强强度档位
- 取值必须在 `1`-`8` 之间，超出范围接口直接报错「放大倍数必须是8倍以内」
- 计费只区分 `2`、`4`、`8` 三档：`2` 为基础单价，`4` 为 2 倍单价，`8` 为 3 倍单价；其余取值按基础单价计费
- 工作流侧默认按 2 倍无损放大处理

推荐默认规则：

- 用户未指定倍数时，默认传 `2`
- 用户要求“放大 4 倍 / 更大尺寸”时，传 `4`
- 用户要求“尽可能放大”时，传 `8`

> **说明**：该接口按倍数放大原图，只做无损放大，不接收自然语言提示词。如果用户真正想要的是“提升清晰度、修复模糊”，应改用 AI 超清（image-clarity-enhance）技能，那里的 `enhanceStrength` 才是强度档位。

## 调用示例

> **跨平台调用说明**：
> - 请求头必须包含 `Content-Type: application/json; charset=utf-8` 和 `secretKey`
> - **Windows/PowerShell**：因 GBK 编码问题，必须先将 JSON 写入临时文件 `payload_temp.json`（UTF-8 无 BOM），再用 `curl.exe --% --data-binary @payload_temp.json` 发送请求。使用 Write 工具创建文件，或用 .NET API `[System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))`。调用后用 `rm payload_temp.json` 清理。
> - **macOS/Linux**：bash/zsh 默认 UTF-8，可直接内联 JSON：`curl -X POST URL -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --data-binary 'JSON单行内容'`

### 示例 1：单张图片 2 倍放大

**前置步骤**：向用户索取图片路径或 URL。如用户提供本地文件，先调用 file-upload 技能上传获取公网链接。

**Windows/PowerShell**：

步骤 1：创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "imgUrls": "https://example.com/img1.jpg",
  "scalingRatio": 2
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"imgUrls":"https://example.com/img1.jpg","scalingRatio":2}', [System.Text.UTF8Encoding]::new($false))
```

步骤 2：执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/enlarge" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/enlarge" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary '{"imgUrls":"https://example.com/img1.jpg","scalingRatio":2}'
```

### 示例 2：批量图片 4 倍放大

**前置步骤**：向用户索取图片路径或 URL。如用户提供本地文件，先调用 file-upload 技能上传获取公网链接。

**Windows/PowerShell**：

步骤 1：创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "imgUrls": "https://example.com/img1.jpg,https://example.com/img2.jpg",
  "scalingRatio": 4
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"imgUrls":"https://example.com/img1.jpg,https://example.com/img2.jpg","scalingRatio":4}', [System.Text.UTF8Encoding]::new($false))
```

步骤 2：执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/enlarge" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/enlarge" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary '{"imgUrls":"https://example.com/img1.jpg,https://example.com/img2.jpg","scalingRatio":4}'
```

## 常见错误及解决方案

| 错误 | 原因与解决 |
|------|-----------|
| HTTP 401 / `code` 非 200 | `secretKey` 无效、缺失或已过期，确认请求头是否正确传入 |
| HTTP 405 Not Allowed | 请求方法错误，必须使用 `POST` |
| `imgUrls` 格式错误 | 该字段必须是字符串，多张图用英文逗号分隔，不是 JSON 数组 |
| 图片 URL 无法访问 | 传入的链接不是公网直链、已过期，或源站限制访问 |
| `放大倍数必须是8倍以内` | `scalingRatio` 超出 `1`-`8`，改用 `2`、`4` 或 `8` |
| 接口提示图片规格不符合要求 | 换用更规范的图片尺寸或格式后重试 |
| 请求超时 | 批量图片较多或放大倍数较高时，可适当增大超时时间 |

## 执行流程

1. **向用户询问 `secretKey`**（API 密钥必须由用户提供，agent 不可自行填写）
2. 收集一张或多张图片 URL（如用户提供本地文件，先调用 file-upload 技能上传获取公网链接）
3. 将多张 URL 用英文逗号拼接为 `imgUrls`
4. 根据用户意图确定 `scalingRatio`（放大倍数，未指定时用 `2`）
5. 在请求头中传入 `secretKey`，调用接口
6. 将返回的结果按逗号拆分后逐个展示

该接口不接收自然语言提示词，不需要构造额外文案。如果用户只是说“帮我变清晰一点”而没有提尺寸，应改用 AI 超清（image-clarity-enhance）技能；只有明确要“放大、提尺寸、提分辨率”时才用本技能。
