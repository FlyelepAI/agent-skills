---
name: gen-hot-video
description: >-
  通过 Flyelep 爆款视频复刻 API，基于爆款视频风格生成产品复刻视频。
  当用户要求复刻爆款视频、模仿参考视频风格生成产品视频时使用此技能。
---
# Flyelep 爆款视频复刻
通过 Flyelep AI Tool API 复刻爆款视频风格，将产品视频素材融合到爆款参考视频的视觉风格中。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**
**注意：此接口为异步接口，只返回任务ID，需要通过 queryTaskResult 接口获取最终结果。**

## API 接口信息

- **认证方式**: 在请求头中传入 `secretKey`（密钥需由用户在 Flyelep 开放平台申请：https://www.flyelep.cn/controlboard）
- **Content-Type**: `application/json`

请求头示例：

```http
Content-Type: application/json
secretKey: 用户提供的API密钥
```

> **安全说明**：不要将真实密钥写入技能文件、示例代码仓库或持久化配置中，应在运行时由用户动态提供。

### 创建任务

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/generateHotVideo`
- **超时时间**: 建议 60-120 秒（获取任务结果需额外轮询，视频生成可能需要更长时间）

### 查询结果

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult`
- **超时时间**: 建议 30 秒

## 请求 Body

创建任务：
```json
{
  "replaceUrl": "https://example.com/product_video.mp4",
  "sourceUrl": "https://example.com/hot_video.mp4",
  "prompt": "突出产品功能，增强视觉冲击力",
  "additionalPrompt": "展示产品使用场景",
  "modelType": "pro",
  "resolution": "720p",
  "ratio": "1:1",
  "duration": 10,
  "subtitle": true,
  "language": "中文简体"
}
```

查询结果：
```json
{
  "agentGenerateTaskId": "任务ID"
}
```

## 响应格式

创建任务（异步）：
```json
{
  "code": 200,
  "data": {
    "agentGenerateTaskId": "2072923591164715009"
  }
}
```

查询任务结果：
```json
{
  "code": 200,
  "data": {
    "taskList": [
      {
        "taskStatus": 2,
        "executeResult": "https://example.com/result_video.mp4"
      }
    ]
  }
}
```

- `code=200` 表示调用成功
- `agentGenerateTaskId` 为异步任务ID，用于后续查询结果
- `taskStatus`: 0-待生成，1-生成中，2-生成成功，3-生成失败
- `executeResult` 为生成的视频URL
- 将结果视频展示给用户

## 参数说明

### 必传参数

> **重要**：以下必传参数必须通过询问用户获取，agent 不可自行填写。调用本技能时，应先向用户列出必传参数与可选参数表格，由用户确认或提供后再执行。

| 字段 | 默认值 | 说明 |
|------|--------|------|
| replaceUrl | - | 产品素材视频地址 |
| sourceUrl | - | 爆款参考视频地址，必须包含视频（4-15秒以内） |
| prompt | - | 提示词，最多1000个字符长度 |
| modelType | - | 模型类型：`pro`=Flyelep Dance 2.0，`fast`=Flyelep Dance 2.0 Fast |
| resolution | - | 分辨率 |
| duration | - | 视频时长（秒），4-15秒 |
| language | - | 生成语言 |

### 可选参数

| 字段 | 默认值 | 说明 |
|------|--------|------|
| additionalPrompt | - | 补充提示词（可选） |
| ratio | - | 视频生成比例 |
| subtitle | - | 是否添加字幕，`true`/`false` |

### 参数映射规则

**modelType**（模型类型）：
- `pro`：Flyelep Dance 2.0（高质量）
- `fast`：Flyelep Dance 2.0 Fast（快速生成）

**resolution**（分辨率）：
- 支持：`480p`、`720p`、`1080p`、`4k`

**ratio**（视频比例）：
- 支持：`1:1`、`3:4`、`4:3`、`16:9`、`9:16`、`21:9`

**duration**（视频时长）：
- 范围：4-15秒
- 建议根据需求选择合适的时长

**language**（生成语言）：
- 中文简体、中文繁体、英语、马来语、葡萄牙语、韩语、日语、西班牙语、俄语等

**replaceUrl**（产品素材）：
- 产品视频地址，用于替换爆款视频中的产品
- 支持视频格式
- 如果用户提供本地文件路径，先调用 image-upload 技能上传文件获取公网链接，再填入此参数

**sourceUrl**（爆款参考）：
- 爆款参考视频地址，用于提供风格参考
- 必须包含视频，时长在4-15秒以内
- 如果用户提供本地文件路径，先调用 image-upload 技能上传文件获取公网链接，再填入此参数

**subtitle**（字幕控制）：
- `true`：添加字幕
- `false`：不添加字幕
- 通过追加提示词可控制字幕内容

## 异步任务流程

> **重要**：本接口为异步接口，必须**先调用主接口获取 `agentGenerateTaskId`，然后调用 `queryTaskResult` 接口轮询任务结果**，不能省略轮询步骤。

1. 调用主接口（`generateHotVideo`）提交任务，从响应中获取 `agentGenerateTaskId`
2. 使用 `agentGenerateTaskId` 调用 `queryTaskResult` 接口轮询任务结果（建议每5-10秒查询一次）
3. 当 `taskStatus=2` 时，表示生成成功，获取 `executeResult` 结果
4. 当 `taskStatus=3` 时，表示生成失败

> **轮询策略**：视频生成耗时较长，超时时间建议设置为10分钟

## 调用示例

> **跨平台调用说明**：
> - 请求头必须包含 `Content-Type: application/json; charset=utf-8` 和 `secretKey`
> - **Windows/PowerShell**：因 GBK 编码问题，必须先将 JSON 写入临时文件 `payload_temp.json`（UTF-8 无 BOM），再用 `curl.exe --% --data-binary @payload_temp.json` 发送请求。使用 Write 工具创建文件，或用 .NET API `[System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))`。调用后用 `rm payload_temp.json` 清理。
> - **macOS/Linux**：bash/zsh 默认 UTF-8，可直接内联 JSON：`curl -X POST URL -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --data-binary 'JSON单行内容'`

### 示例 1：完整流程 - 提交任务并查询结果

**前置步骤**：向用户索取产品素材视频和爆款参考视频的路径或 URL。如用户提供本地文件，先调用 image-upload 技能上传获取公网链接。

**Windows/PowerShell**：

步骤 1：创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "replaceUrl": "https://example.com/product_video.mp4",
  "sourceUrl": "https://example.com/hot_video.mp4",
  "prompt": "突出产品功能，增强视觉冲击力",
  "modelType": "pro",
  "resolution": "720p",
  "ratio": "1:1",
  "duration": 10,
  "subtitle": true,
  "language": "中文简体"
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"replaceUrl":"https://example.com/product_video.mp4","sourceUrl":"https://example.com/hot_video.mp4","prompt":"突出产品功能，增强视觉冲击力","modelType":"pro","resolution":"720p","ratio":"1:1","duration":10,"subtitle":true,"language":"中文简体"}', [System.Text.UTF8Encoding]::new($false))
```

步骤 2：执行创建任务请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/generateHotVideo" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

步骤 4：使用返回的 `agentGenerateTaskId` 创建查询请求 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "agentGenerateTaskId": "2072923591164715009"
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"agentGenerateTaskId":"2072923591164715009"}', [System.Text.UTF8Encoding]::new($false))
```

步骤 5：执行查询请求（每5-10秒轮询，直到 `taskStatus=2`）：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary @payload_temp.json
```

步骤 6：清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：

创建任务：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/generateHotVideo" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary '{"replaceUrl":"https://example.com/product_video.mp4","sourceUrl":"https://example.com/hot_video.mp4","prompt":"突出产品功能，增强视觉冲击力","modelType":"pro","resolution":"720p","ratio":"1:1","duration":10,"subtitle":true,"language":"中文简体"}'
```

查询结果（每5-10秒轮询，直到 `taskStatus=2`）：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary '{"agentGenerateTaskId":"2072923591164715009"}'
```

### 示例 2：快速模式复刻（fast模型）

**前置步骤**：向用户索取产品素材视频和爆款参考视频的路径或 URL。如用户提供本地文件，先调用 image-upload 技能上传获取公网链接。

**Windows/PowerShell**：

步骤 1：创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "replaceUrl": "https://example.com/product_video.mp4",
  "sourceUrl": "https://example.com/hot_video.mp4",
  "prompt": "展示产品卖点，节奏紧凑",
  "additionalPrompt": "快节奏剪辑，突出关键特性",
  "modelType": "fast",
  "resolution": "480p",
  "ratio": "9:16",
  "duration": 6,
  "subtitle": true,
  "language": "英文"
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"replaceUrl":"https://example.com/product_video.mp4","sourceUrl":"https://example.com/hot_video.mp4","prompt":"展示产品卖点，节奏紧凑","additionalPrompt":"快节奏剪辑，突出关键特性","modelType":"fast","resolution":"480p","ratio":"9:16","duration":6,"subtitle":true,"language":"英文"}', [System.Text.UTF8Encoding]::new($false))
```

步骤 2：执行创建任务请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/generateHotVideo" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

步骤 4：使用返回的 `agentGenerateTaskId` 查询结果（参考示例 1 步骤 4-6）

**macOS/Linux**：

创建任务：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/generateHotVideo" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary '{"replaceUrl":"https://example.com/product_video.mp4","sourceUrl":"https://example.com/hot_video.mp4","prompt":"展示产品卖点，节奏紧凑","additionalPrompt":"快节奏剪辑，突出关键特性","modelType":"fast","resolution":"480p","ratio":"9:16","duration":6,"subtitle":true,"language":"英文"}'
```

查询结果（参考示例 1 的查询结果命令，替换 `agentGenerateTaskId`）

## 常见错误及解决方案

| 错误 | 原因与解决 |
|------|-----------|
| HTTP 401 / `code` 非 200 | `secretKey` 无效、缺失或已过期，确认请求头是否正确传入 |
| HTTP 405 Not Allowed | 请求方法错误，必须使用 `POST` |
| sourceUrl视频时长超出范围 | 参考视频必须在4-15秒以内 |
| prompt 超过1000字符 | 缩短提示词内容 |
| duration 超出范围 | 视频时长需在4-15秒之间 |
| 服务繁忙（9999错误码） | 稍后重试 |
| taskStatus=3 生成失败 | 检查视频素材质量，尝试更换素材或调整prompt |
| 视频生成超时 | 视频生成耗时较长，增大超时时间并继续轮询 |

## 执行流程

1. **向用户询问 `secretKey`**（API 密钥必须由用户提供，agent 不可自行填写）
2. 收集产品素材视频 URL 和爆款参考视频 URL（如用户提供本地文件，先调用 image-upload 技能上传获取公网链接）
3. 与用户确认复刻需求，构造 `prompt` 和 `additionalPrompt`，选择 `modelType`、`resolution`、`ratio`、`duration`、`language`
4. 在请求头中传入 `secretKey`，调用创建任务接口，获取 `agentGenerateTaskId`
5. 使用 `agentGenerateTaskId` 轮询查询结果接口（每5-10秒一次），直到 `taskStatus=2`
6. 将返回的结果视频展示给用户

复刻时，prompt 应指导AI：
- 保持爆款视频的整体风格、节奏和视觉效果
- 将产品自然地融入到场景中
- 突出产品卖点和关键信息
- 保持流畅的动态效果

**示例prompt：**
- "保持爆款视频的节奏感，将产品自然融入场景"
- "突出产品功能展示，配合动态视觉效果"
- "延续原视频的电商促销风格，产品主体突出"
- "展示产品使用场景，营造氛围感染力"

**additionalPrompt 示例（补充提示词）：**
- "快节奏剪辑，突出关键特性"
- "配合字幕和品牌logo"
- "转场流畅，视觉冲击力强"
