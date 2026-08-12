---
name: image-clarity-enhance
description: >-
  通过 Flyelep AI 工具接口增强图片清晰度，支持单张或批量处理。
  当用户要求做 AI 超清、提升图片清晰度、批量增强模糊图片时使用此技能。
---
# Flyelep AI超清

通过 Flyelep AI Tool API 增强图片清晰度，并返回增强后的新图片 URL。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**

## API 接口信息

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/imageClarityEnhance`
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
  "enhanceStrength": "light"
}
```

## 响应格式

```json
{
  "code": 200,
  "msg": "操作成功",
  "data": "https://example.com/clear1.jpg,https://example.com/clear2.jpg"
}
```

- `code=200` 表示调用成功
- `msg` 为接口返回说明
- `data` 为超清后的图片地址
- 多张图片时，`data` 中多个 URL 以英文逗号 `,` 分隔
- 返回结果应按逗号拆分后逐个展示给用户，不要回读图片内容

## 参数说明

### 必传参数

> **重要**：以下必传参数必须通过询问用户获取，agent 不可自行填写。调用本技能时，应先向用户列出必传参数与可选参数表格，由用户确认或提供后再执行。

| 字段 | 默认值 | 说明 |
|------|--------|------|
| imgUrls | - | 多张图片链接字符串，英文逗号分隔 |
| enhanceStrength | - | 增强强度：`light`、`standard`、`strong` |

### 参数映射规则

**imgUrls**：
- 接口要求传字符串，不是数组
- 单张图片时直接传一个 URL 字符串
- 多张图片时，用英文逗号 `,` 按顺序拼接
- 每个链接都应为公网可访问的图片直链，不要传网页地址
- 如果用户提供本地文件路径，先调用 image-upload 技能上传文件获取公网链接，再填入此参数

**enhanceStrength**：
- `light`：轻度增强
- `standard`：标准增强
- `strong`：强力增强

推荐默认规则：

- 用户未指定增强强度时，默认传 `light`
- 用户强调“尽量自然、轻微增强”时，传 `light`
- 用户强调“尽可能清晰、强力修复”时，传 `strong`

## 图片规格限制

文档明确要求源图满足以下限制：

- 格式仅支持：`JPG`、`PNG`、`BMP`
- 最短边不低于 `10px`
- 最长边不超过 `5000px`
- 长宽比不超过 `4:1`
- 文件大小不超过 `8MB`

如果任一条件不满足，接口可能报错或处理失败。

## 调用示例

> **跨平台调用说明**：
> - 请求头必须包含 `Content-Type: application/json; charset=utf-8` 和 `secretKey`
> - **Windows/PowerShell**：因 GBK 编码问题，必须先将 JSON 写入临时文件 `payload_temp.json`（UTF-8 无 BOM），再用 `curl.exe --% --data-binary @payload_temp.json` 发送请求。使用 Write 工具创建文件，或用 .NET API `[System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))`。调用后用 `rm payload_temp.json` 清理。
> - **macOS/Linux**：bash/zsh 默认 UTF-8，可直接内联 JSON：`curl -X POST URL -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --data-binary 'JSON单行内容'`

### 示例 1：单张图片标准超清

**前置步骤**：向用户索取图片路径或 URL。如用户提供本地文件，先调用 image-upload 技能上传获取公网链接。

**Windows/PowerShell**：

步骤 1：创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "imgUrls": "https://example.com/img1.jpg",
  "enhanceStrength": "standard"
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"imgUrls":"https://example.com/img1.jpg","enhanceStrength":"standard"}', [System.Text.UTF8Encoding]::new($false))
```

步骤 2：执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/imageClarityEnhance" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/imageClarityEnhance" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary '{"imgUrls":"https://example.com/img1.jpg","enhanceStrength":"standard"}'
```

### 示例 2：批量图片轻度超清

**前置步骤**：向用户索取图片路径或 URL。如用户提供本地文件，先调用 image-upload 技能上传获取公网链接。

**Windows/PowerShell**：

步骤 1：创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "imgUrls": "https://example.com/img1.jpg,https://example.com/img2.jpg",
  "enhanceStrength": "light"
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"imgUrls":"https://example.com/img1.jpg,https://example.com/img2.jpg","enhanceStrength":"light"}', [System.Text.UTF8Encoding]::new($false))
```

步骤 2：执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/imageClarityEnhance" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/imageClarityEnhance" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary '{"imgUrls":"https://example.com/img1.jpg,https://example.com/img2.jpg","enhanceStrength":"light"}'
```

## 常见错误及解决方案

| 错误 | 原因与解决 |
|------|-----------|
| HTTP 401 / `code` 非 200 | `secretKey` 无效、缺失或已过期，确认请求头是否正确传入 |
| HTTP 405 Not Allowed | 请求方法错误，必须使用 `POST` |
| `imgUrls` 格式错误 | 该字段必须是字符串，多张图用英文逗号分隔，不是 JSON 数组 |
| 图片 URL 无法访问 | 传入的链接不是公网直链、已过期，或源站限制访问 |
| `enhanceStrength` 非法 | 只支持 `light`、`standard`、`strong` |
| 图片规格不符合要求 | 检查格式、尺寸、长宽比和文件大小是否满足文档限制 |
| 请求超时 | 批量图片较多或增强强度较高时，可适当增大超时时间 |

## 执行流程

1. **向用户询问 `secretKey`**（API 密钥必须由用户提供，agent 不可自行填写）
2. 收集一张或多张图片 URL（如用户提供本地文件，先调用 image-upload 技能上传获取公网链接）
3. 将多张 URL 用英文逗号拼接为 `imgUrls`
4. 根据用户意图选择 `enhanceStrength`
5. 确认源图尽量满足规格限制
6. 在请求头中传入 `secretKey`，调用接口
7. 将返回的结果按逗号拆分后逐个展示

该接口不接收自然语言提示词，不需要构造额外文案。当用户只是说“帮我变清晰一点”时，优先使用 `light`；如果用户已经用过无损放大 skill，但更需要遵守文档里的规格限制和 `light/standard/strong` 语义，则优先使用此技能。
