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
- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/generateHotVideo`
- **Content-Type**: `application/json`
- **超时时间**: 建议 60-120 秒（获取任务结果需额外轮询，视频生成可能需要更长时间）

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

## 响应格式
提交请求（异步）：
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

## 参数映射规则
### modelType（模型类型）
- `pro`：Flyelep Dance 2.0（高质量）
- `fast`：Flyelep Dance 2.0 Fast（快速生成）

### resolution（分辨率）
- 支持：`480p`、`720p`、`1080p`、`4k`

### ratio（视频比例）
- 支持：`1:1`、`3:4`、`4:3`、`16:9`、`9:16`、`21:9`

### duration（视频时长）
- 范围：4-15秒
- 建议根据需求选择合适的时长

### language（生成语言）
- 中文简体、中文繁体、英语、马来语、葡萄牙语、韩语、日语、西班牙语、俄语等

### replaceUrl（产品素材）
- 产品视频地址，用于替换爆款视频中的产品
- 支持视频格式

### sourceUrl（爆款参考）
- 爆款参考视频地址，用于提供风格参考
- 必须包含视频，时长在4-15秒以内

### subtitle（字幕控制）
- `true`：添加字幕
- `false`：不添加字幕
- 通过追加提示词可控制字幕内容

## 异步任务查询
生成视频为异步流程，需要：
1. 调用 `generateHotVideo` 提交任务，获取 `agentGenerateTaskId`
2. 轮询调用 `queryTaskResult` 查询任务状态
3. 当 `taskStatus=2` 时，表示生成成功，获取结果

### 查询任务结果接口
- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult`
- **请求体**:
```json
{
  "agentGenerateTaskId": "任务ID"
}
```

- **轮询策略**：建议每5-10秒查询一次，视频生成耗时较长，超时时间建议设置为10分钟

## 调用示例
- **重要**：调用 API 时，必须设置 `Content-Type: application/json; charset=utf-8` 请求头。以下分平台说明：
- **Windows/PowerShell 环境**：
  - 必须采用以下流程：**先将请求体 JSON 写入当前工作目录下的临时文件 `payload_temp.json`，再通过 Shell 工具调用 `curl.exe --data-binary @payload_temp.json` 发送请求**。这是因为 PowerShell 使用 GBK 编码，而服务端使用 UTF-8 解析，直接在命令行中嵌入中文 JSON 会导致乱码。
  - 使用 `curl.exe`（而非 `curl`，后者在 PowerShell 中是 `Invoke-WebRequest` 的别名）。必须在 `curl.exe` 后加 `--%` 停止 PowerShell 解析，否则 `@` 会被误判为 splatting 操作符导致报错。
  - **文件创建方式**：根据可用工具选择其一（均需确保 UTF-8 **无 BOM** 编码，否则服务端 JSON 解析会在 position 0 报错）：
    - **方式 A（有 Write 工具）**：使用 Write 工具创建 `payload_temp.json`
    - **方式 B（无 Write 工具）**：使用 Shell 的 .NET API 创建文件（`Set-Content -Encoding UTF8` 会带 BOM，不可用）
- **macOS/Linux 环境**：
  - bash/zsh 默认使用 UTF-8 编码，可直接内联中文 JSON，无需临时文件。命令中使用 `curl`（无需 `.exe`，无需 `--%`）。
  - 推荐内联写法：`curl -X POST URL -H "..." -H "..." --data-binary 'JSON单行内容'`，一步完成。
  - 也可使用临时文件方式：`curl --data-binary @payload_temp.json`。
- **清理**：API 返回结果后，务必删除 `payload_temp.json` 临时文件（如使用了临时文件）。

**示例 1：提交爆款视频复刻任务**

步骤 1：创建 `payload_temp.json`，内容如下：
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
> 方式 B（无 Write 工具）：
> ```powershell
> $json = '{"replaceUrl":"https://example.com/product_video.mp4","sourceUrl":"https://example.com/hot_video.mp4","prompt":"突出产品功能，增强视觉冲击力","modelType":"pro","resolution":"720p","ratio":"1:1","duration":10,"subtitle":true,"language":"中文简体"}'
> [System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))
> ```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/generateHotVideo" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary @payload_temp.json
```

> **macOS/Linux 内联写法**（无需临时文件）：
> ```bash
> curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/generateHotVideo" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary '{"replaceUrl":"https://example.com/product_video.mp4","sourceUrl":"https://example.com/hot_video.mp4","prompt":"突出产品功能，增强视觉冲击力","modelType":"pro","resolution":"720p","ratio":"1:1","duration":10,"subtitle":true,"language":"中文简体"}'
> ```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**示例 2：查询任务结果**

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "agentGenerateTaskId": "2072923591164715009"
}
```
> 方式 B（无 Write 工具）：
> ```powershell
> $json = '{"agentGenerateTaskId":"2072923591164715009"}'
> [System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))
> ```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary @payload_temp.json
```

> **macOS/Linux 内联写法**（无需临时文件）：
> ```bash
> curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary '{"agentGenerateTaskId":"2072923591164715009"}'
> ```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**示例 3：快速模式复刻（fast模型）**

步骤 1：创建 `payload_temp.json`，内容如下：
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
> 方式 B（无 Write 工具）：
> ```powershell
> $json = '{"replaceUrl":"https://example.com/product_video.mp4","sourceUrl":"https://example.com/hot_video.mp4","prompt":"展示产品卖点，节奏紧凑","additionalPrompt":"快节奏剪辑，突出关键特性","modelType":"fast","resolution":"480p","ratio":"9:16","duration":6,"subtitle":true,"language":"英文"}'
> [System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))
> ```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/generateHotVideo" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary @payload_temp.json
```

> **macOS/Linux 内联写法**（无需临时文件）：
> ```bash
> curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/generateHotVideo" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary '{"replaceUrl":"https://example.com/product_video.mp4","sourceUrl":"https://example.com/hot_video.mp4","prompt":"展示产品卖点，节奏紧凑","additionalPrompt":"快节奏剪辑，突出关键特性","modelType":"fast","resolution":"480p","ratio":"9:16","duration":6,"subtitle":true,"language":"英文"}'
> ```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

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

## 提示词处理
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
