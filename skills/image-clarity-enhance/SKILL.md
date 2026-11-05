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
  "imageUrlList": ["https://example.com/img1.jpg", "https://example.com/img2.jpg"],
  "enhanceStrength": "light"
}
```

## 响应格式

```json
{
  "code": 200,
  "data": "https://example.com/clear1.jpg,https://example.com/clear2.jpg",
  "result": {
    "fileUrlList": ["https://example.com/clear1.jpg", "https://example.com/clear2.jpg"],
    "taskId": "210438182055381xxxx"
  }
}
```

- `code=200` 表示调用成功；失败时 `code` 非 200，原因在 `msg` 里（成功时 `msg` 通常不返回）
- `result.fileUrlList`：超清结果图片 URL 数组，按输入顺序排列，**优先读取这个字段**
- `result.taskId`：本次任务的主任务 ID（字符串），可用 `/poster/queryTaskResult` 查询每张图的明细
- `data`：兼容旧版的字段，值与 `fileUrlList` 一致，单张为 URL 字符串，多张为英文逗号 `,` 分隔的字符串；新接入请读 `result.fileUrlList`
- 批量时部分图片失败，只返回成功的结果，按成功张数计费；全部失败时 `code` 非 200，`msg` 为第一张失败的原因
- 返回结果逐个展示给用户，不要回读图片内容

## 参数说明

### 必传参数

> **重要**：以下必传参数必须通过询问用户获取，agent 不可自行填写。调用本技能时，应先向用户列出必传参数与可选参数表格，由用户确认或提供后再执行。

| 字段 | 默认值 | 说明 |
|------|--------|------|
| imageUrlList | - | 图片 URL 数组，单张也传数组；建议单次不超过 6 张 |
| enhanceStrength | light | 增强强度：`light`、`standard`、`strong`，不传按 `light` |

### 可选参数

| 字段 | 默认值 | 说明 |
|------|--------|------|
| physicalWidthMm | - | 印刷成品物理宽度（毫米），与 `physicalHeightMm` 都填且大于 0 时按 300DPI 输出印刷级尺寸 |
| physicalHeightMm | - | 印刷成品物理高度（毫米），需与 `physicalWidthMm` 同时传 |

### 参数映射规则

**imageUrlList**：
- 接口要求传 JSON 字符串数组，单张图片也写成 `["url"]`
- 每个链接都应为公网可访问的图片直链，不要传网页地址
- 多张图片按顺序处理，结果也按输入顺序返回；张数越多耗时越长，建议单次不超过 6 张，更多时分批调用
- 旧字段 `imgUrls`（英文逗号分隔的字符串）和 `imageUrl`（单张）仍兼容，优先级 `imageUrlList` > `imgUrls` > `imageUrl`；新接入统一用 `imageUrlList`
- 都不传会报「图片链接不能为空」
- 如果用户提供本地文件路径，先按「本地文件上传」章节换取公网直链，再填入此参数

**enhanceStrength**：
- `light`：轻度增强，按基础单价计费
- `standard`：标准增强，按 2 倍单价计费
- `strong`：强力增强，按 3 倍单价计费
- 强度越高效果越明显，结果图分辨率也越高（底层超分倍数 light 约 2 倍、standard 约 4 倍、strong 约 8 倍）
- 不传或传空时按 `light` 处理并计费（单张、多张一致）；大小写不敏感，传以上三个之外的值会报错 `enhanceStrength 仅支持 light / standard / strong`

**physicalWidthMm / physicalHeightMm**：
- 仅在用户明确要求印刷尺寸时传，单位毫米，两个字段都要传且大于 0
- 不传时输出普通超清图

推荐默认规则：

- 用户未指定增强强度时，默认传 `light`
- 用户强调“尽量自然、轻微增强”时，传 `light`
- 用户强调“尽可能清晰、强力修复”时，传 `strong`

## 图片规格限制

源图需满足以下限制：

- 长边不超过 `2048px`
- 短边不低于 `256px`
- 文件大小不超过 `10MB`
- 建议使用 `JPG`、`PNG`

不满足时该张图片处理失败，`msg` 会给出具体原因（如「图片尺寸超出 AIGC 超分范围」「图片尺寸过小」「图片大小超出 AIGC 超分范围」）；超长边的图片可先压缩到 2048px 以内再调用。

## 本地文件上传

用户提供的是本地文件路径而不是公网直链时，先把文件上传换取直链，再调用本接口。已安装 `file-upload` 技能时以该技能为准；未安装时按下面的说明直接调用上传接口。

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/file/upload`
- **请求方式**: `multipart/form-data`，文件字段名固定为 `file`，单次只能上传一个文件，多个文件并发调用多次
- **认证方式**: 请求头传 `secretKey`，与本技能使用同一个密钥
- **超时时间**: 图片建议 60-120 秒
- **不要手动设置 `Content-Type` 请求头**，让 HTTP 客户端自动生成带 boundary 的值，手写会导致服务端解析失败
- 图片仅支持 `bmp`、`gif`、`jpg`、`jpeg`、`png`，`webp` 需先转成 `png` 或 `jpg`；文件名必须带正确后缀，服务端靠它判断格式
- 原文件名不会出现在 URL 里，中文名、空格、特殊字符都能直接上传，不需要提前改名
- 上传不消耗算力，但服务端不做去重：同一个文件在一次任务里只上传一次，记下 `fullPath` 复用

成功响应取 `data.fullPath` 作为公网直链，不带签名，**只保证 24 小时内可访问**，拿到后应尽快用于生成任务：

```json
{
  "code": 200,
  "msg": null,
  "data": {
    "relativePath": "cos_ai_agent_h_24/2026-11-05/3f2a9c1b7d84e6f5a012.png",
    "fullPath": "https://agent-1404002717.cos.ap-guangzhou.myqcloud.com/cos_ai_agent_h_24/2026-11-05/3f2a9c1b7d84e6f5a012.png",
    "serviceProvider": null
  }
}
```

判断成功只看 `code`，业务失败时 HTTP 状态码仍是 200，`code` 为 500 或 9999，原因在 `msg` 里。

```bash
# Windows/PowerShell（用 curl.exe，PowerShell 里的 curl 是 Invoke-WebRequest 的别名）
curl.exe -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/file/upload" -H "secretKey: 你的密钥" --max-time 120 -F "file=@C:/path/to/product.png"

# macOS/Linux
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/file/upload" -H "secretKey: 你的密钥" --max-time 120 -F "file=@./product.png"
```

图片入桶前会先过内容审核，审核不通过整个请求失败，需换图重试。拿到 `code=9999`、`msg` 为 `服务繁忙，请稍后再试` 时，先自查三项：是否漏了 `secretKey` 请求头、表单字段名是否为 `file`、文件是否超出服务端体积上限。密钥、格式、审核、体积类错误重试无效，只有网络超时、5xx 和存储类异常值得重试。

## 调用示例

> **跨平台调用说明**：
> - 请求头必须包含 `Content-Type: application/json; charset=utf-8` 和 `secretKey`
> - **Windows/PowerShell**：因 GBK 编码问题，必须先将 JSON 写入临时文件 `payload_temp.json`（UTF-8 无 BOM），再用 `curl.exe --% --data-binary @payload_temp.json` 发送请求。使用 Write 工具创建文件，或用 .NET API `[System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))`。调用后用 `rm payload_temp.json` 清理。
> - **macOS/Linux**：bash/zsh 默认 UTF-8，可直接内联 JSON：`curl -X POST URL -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --data-binary 'JSON单行内容'`

### 示例 1：单张图片标准超清

**前置步骤**：向用户索取图片路径或 URL。如用户提供本地文件，先按「本地文件上传」章节换取公网直链。

**Windows/PowerShell**：

步骤 1：创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "imageUrlList": ["https://example.com/img1.jpg"],
  "enhanceStrength": "standard"
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"imageUrlList":["https://example.com/img1.jpg"],"enhanceStrength":"standard"}', [System.Text.UTF8Encoding]::new($false))
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
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/imageClarityEnhance" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary '{"imageUrlList":["https://example.com/img1.jpg"],"enhanceStrength":"standard"}'
```

### 示例 2：批量图片轻度超清

**前置步骤**：向用户索取图片路径或 URL。如用户提供本地文件，先按「本地文件上传」章节换取公网直链。

**Windows/PowerShell**：

步骤 1：创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "imageUrlList": ["https://example.com/img1.jpg", "https://example.com/img2.jpg"],
  "enhanceStrength": "light"
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"imageUrlList":["https://example.com/img1.jpg","https://example.com/img2.jpg"],"enhanceStrength":"light"}', [System.Text.UTF8Encoding]::new($false))
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
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/imageClarityEnhance" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary '{"imageUrlList":["https://example.com/img1.jpg","https://example.com/img2.jpg"],"enhanceStrength":"light"}'
```

## 常见错误及解决方案

| 错误 | 原因与解决 |
|------|-----------|
| HTTP 401 / `code` 非 200 | `secretKey` 无效、缺失或已过期，确认请求头是否正确传入 |
| HTTP 405 Not Allowed | 请求方法错误，必须使用 `POST` |
| `图片链接不能为空` | 没传 `imageUrlList`，或数组为空 |
| `imageUrlList` 格式错误 | 该字段必须是 JSON 字符串数组，不是逗号拼接的字符串 |
| 图片 URL 无法访问 / `图片下载失败…` | 传入的链接不是公网直链、已过期，或源站限制访问 |
| `enhanceStrength 仅支持 light / standard / strong` | 强度传了非法值，改为三个可选值之一 |
| `图片尺寸超出 AIGC 超分范围` / `图片尺寸过小` / `图片大小超出 AIGC 超分范围` | 源图不满足「图片规格限制」，压缩或换图后重试 |
| `积分不足，请充值或联系管理员` 等算力不足提示 | 账户算力不够（按张数 × 强度倍数预估），充值或降低强度后重试 |
| 请求超时 | 多张图片按顺序处理，张数多或强度高时耗时长，可增大超时时间或分批调用 |

## 执行流程

1. **向用户询问 `secretKey`**（API 密钥必须由用户提供，agent 不可自行填写）
2. 收集一张或多张图片 URL（如用户提供本地文件，先按「本地文件上传」章节换取公网直链）
3. 将 URL 按顺序放入 `imageUrlList` 数组（单张也用数组）
4. 根据用户意图选择 `enhanceStrength`
5. 确认源图尽量满足规格限制
6. 在请求头中传入 `secretKey`，调用接口
7. 读取 `result.fileUrlList`，将结果图片逐个展示（旧字段 `data` 仅作兼容）

该接口不接收自然语言提示词，不需要构造额外文案。当用户只是说“帮我变清晰一点”时，用本技能并传 `light`；当用户明确要求“放大尺寸、提分辨率”时，改用无损放大（image-enlarge）技能，那里的 `scalingRatio` 是放大倍数。
