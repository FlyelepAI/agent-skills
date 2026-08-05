---
name: generate-video
description: >-
  通过 Flyelep 异步生成视频 API，根据文本提示词生成产品视频或创意视频。
  当用户要求生成视频、AI生成视频、文本转视频、根据描述生成视频时使用此技能。
---
# Flyelep 异步生成视频

通过 Flyelep AI Tool API 异步生成视频，支持基于文本提示词创建产品展示视频或创意视频。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**
**注意：此接口为异步接口，只返回任务ID，需要通过 queryTaskResult 接口获取最终结果。**

## API 接口信息
- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateVideo`
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
  "query": "为这款智能手表制作一个展示视频，突出运动监测功能",
  "platformType": "Amazon",
  "languageType": "英语",
  "videoModelType": "pro",
  "aspectRatio": "16:9",
  "needVoice": true,
  "duration": 10,
  "resolution": "720p",
  "videoTag": "主图视频",
  "referenceImageStr": "https://example.com/product1.png,https://example.com/product2.png"
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

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| query | string(0,1000) | - | 用户生成海报的具体需求描述（最多1000个字符长度） |
| platformType | string | Amazon | 平台类型 |
| languageType | string | 英语 | 语言 |
| videoModelType | String | - | 视频模型 |
| needVoice | Boolean | true | 是否生成配音音频 |
| duration | Integer | - | 视频时长（单位：秒，最少4秒，最高建议15秒） |
| resolution | String | - | 分辨率 |
| videoTag | String | - | 视频业务标签 |

### 可选参数
| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| aspectRatio | String | adaptive | 画面比例 |
| referenceImageStr | String | - | 参考图片，多个以英文逗号分隔 |
| referenceVideoStr | String | - | 参考视频，多个以英文逗号分隔 |
| referenceAudioStr | String | - | 参考音频，多个以英文逗号分隔 |
| firstFrame | String | - | 首帧图片 URL |
| lastFrame | String | - | 尾帧图片 URL |

## 参数映射规则

### query（需求描述）
- 用户生成海报的具体需求描述
- 最多1000个字符长度
- 直接传入用户对视频的生成需求

### platformType（平台类型）
支持以下平台：
- 跨境电商：`Amazon`、`temu`、`Shopee`、`TikTok Shop`、`AliExpress`、`阿里巴巴国际站`、`OZON`、`Lazada`、`DHgate`、`Coupang`、`11Street`、`Wayfair`、`Etsy`、`Noon`、`eBay`
- 中文电商：`淘宝`、`京东`、`拼多多`、`1688`、`小红书`、`抖音`
- 默认值：`Amazon`

### languageType（语言类型）
- 支持：`英语`、`中文简体`
- 默认值：`英语`

### videoModelType（视频模型）
- `pro`：Flyelep Video 2.0 Pro（高质量）
- `fast`：Flyelep Video 2.0（快速生成）

### aspectRatio（画面比例）
- `adaptive`：智能比例（默认）
- `1:1`、`4:3`、`3:4`、`16:9`、`9:16`、`21:9`

### needVoice（配音控制）
- `true`：有配音（默认）
- `false`：无配音

### duration（视频时长）
- 范围：最少4秒，最高建议15秒
- 单位：秒

### resolution（分辨率）
- 支持：`480p`、`720p`、`1080p`、`2K`、`4K`

### videoTag（视频业务标签）
- `主图视频`：产品主图展示
- `种草视频`：产品种草推荐
- `品牌宣传`：品牌形象宣传

### 生成方式
- 全能参考模式：使用参考图片、视频、音频生成视频，用户可以提供参考图片、视频、音频，但也可以不提供，此时视频会根据这些参考内容生成。
- 首尾帧模式：使用首帧图片、尾帧图片生成视频，用户必须提供首帧图片、尾帧图片，视频会根据这些图片生成。
- 在提交请求前，请询问用户使用哪种模式。

### referenceImageStr（参考图片）
- 当用户指定使用全能参考模式的时候，才可以使用，但可以为空；如果是首尾帧模式的时候，此参数必须为空，不能使用
- 多个以英文逗号分隔
- 最多6张图片，单个不超过10MB
- **注意**：图片、视频和音频文件总数不能超出6个

### referenceVideoStr（参考视频）
- 当用户指定使用全能参考模式的时候，才可以使用，但可以为空；如果是首尾帧模式的时候，此参数必须为空，不能使用
- 多个以英文逗号分隔
- 最多3条视频，单个不超过50MB
- 总时长不能超过15秒
- **禁止人像**
- **注意**：图片、视频和音频文件总数不能超出6个

### referenceAudioStr（参考音频）
- 当用户指定使用全能参考模式的时候，才可以使用，但可以为空；如果是首尾帧模式的时候，此参数必须为空，不能使用
- 多个以英文逗号分隔
- 最多3条音频，单个不超过50MB
- 总时长不能超过15秒
- **注意**：图片、视频和音频文件总数不能超出6个

### firstFrame（首帧图片）
- 当用户指定使用首尾帧模式的时候，才可以使用，此时要求该参数不能为空
- 只能上传一张图片
- **重要**：首帧图片不能同图片、视频、音频一起使用

### lastFrame（尾帧图片）
- 当用户指定使用首尾帧模式的时候，才可以使用，此时要求该参数不能为空
- 只能上传一张图片
- **重要**：尾帧图片不能同图片、视频、音频一起使用

## 异步任务查询
生成视频为异步流程，需要：
1. 调用 `generateVideo` 提交任务，获取 `agentGenerateTaskId`
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

- **轮询策略**：建议每5-10秒查询一次，视频生成耗时较长，超时时间建议设置为20分钟

## 调用示例
- **重要**：在 Windows/PowerShell 环境下调用 API 时，必须采用以下流程：**先将请求体 JSON 写入当前工作目录下的临时文件 `payload_temp.json`，再通过 Shell 工具调用 `curl.exe --data-binary @payload_temp.json` 发送请求**。这是因为 PowerShell 使用 GBK 编码，而服务端使用 UTF-8 解析，直接在命令行中嵌入中文 JSON 会导致乱码。同时必须设置 `Content-Type: application/json; charset=utf-8` 请求头。
- **注意**：在 Windows 环境下使用 `curl.exe`（而非 `curl`，后者在 PowerShell 中是 `Invoke-WebRequest` 的别名）。必须在 `curl.exe` 后加 `--%` 停止 PowerShell 解析，否则 `@` 会被误判为 splatting 操作符导致报错。
- **文件创建方式**：根据可用工具选择其一（均需确保 UTF-8 **无 BOM** 编码，否则服务端 JSON 解析会在 position 0 报错）：
  - **方式 A（有 Write 工具）**：使用 Write 工具创建 `payload_temp.json`
  - **方式 B（无 Write 工具）**：使用 Shell 的 .NET API 创建文件（`Set-Content -Encoding UTF8` 会带 BOM，不可用）
- **清理**：API 返回结果后，务必删除 `payload_temp.json` 临时文件。

**示例 1：提交视频生成任务（基础）**

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "query": "为这款智能手表制作一个展示视频，突出运动监测功能",
  "platformType": "Amazon",
  "languageType": "英语",
  "videoModelType": "pro",
  "aspectRatio": "16:9",
  "needVoice": true,
  "duration": 10,
  "resolution": "720p",
  "videoTag": "主图视频"
}
```
> 方式 B（无 Write 工具）：
> ```powershell
> $json = '{"query":"为这款智能手表制作一个展示视频，突出运动监测功能","platformType":"Amazon","languageType":"英语","videoModelType":"pro","aspectRatio":"16:9","needVoice":true,"duration":10,"resolution":"720p","videoTag":"主图视频"}'
> [System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))
> ```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateVideo" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary @payload_temp.json
```

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

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**示例 3：带参考图片生成视频**

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "query": "根据提供的产品图片生成展示视频",
  "platformType": "淘宝",
  "languageType": "中文简体",
  "videoModelType": "pro",
  "aspectRatio": "1:1",
  "needVoice": true,
  "duration": 8,
  "resolution": "1080p",
  "videoTag": "种草视频",
  "referenceImageStr": "https://example.com/product1.png,https://example.com/product2.png"
}
```
> 方式 B（无 Write 工具）：
> ```powershell
> $json = '{"query":"根据提供的产品图片生成展示视频","platformType":"淘宝","languageType":"中文简体","videoModelType":"pro","aspectRatio":"1:1","needVoice":true,"duration":8,"resolution":"1080p","videoTag":"种草视频","referenceImageStr":"https://example.com/product1.png,https://example.com/product2.png"}'
> [System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))
> ```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateVideo" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**示例 4：快速模式生成视频（fast模型）**

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "query": "制作一个简单的产品展示视频",
  "platformType": "Amazon",
  "languageType": "英语",
  "videoModelType": "fast",
  "aspectRatio": "9:16",
  "needVoice": false,
  "duration": 6,
  "resolution": "480p",
  "videoTag": "主图视频"
}
```
> 方式 B（无 Write 工具）：
> ```powershell
> $json = '{"query":"制作一个简单的产品展示视频","platformType":"Amazon","languageType":"英语","videoModelType":"fast","aspectRatio":"9:16","needVoice":false,"duration":6,"resolution":"480p","videoTag":"主图视频"}'
> [System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))
> ```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateVideo" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**示例 5：使用首帧和尾帧图片**

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "query": "制作一个产品宣传视频",
  "platformType": "京东",
  "languageType": "中文简体",
  "videoModelType": "pro",
  "aspectRatio": "16:9",
  "needVoice": true,
  "duration": 12,
  "resolution": "4K",
  "videoTag": "品牌宣传",
  "firstFrame": "https://example.com/first_frame.png",
  "lastFrame": "https://example.com/last_frame.png"
}
```
> 方式 B（无 Write 工具）：
> ```powershell
> $json = '{"query":"制作一个产品宣传视频","platformType":"京东","languageType":"中文简体","videoModelType":"pro","aspectRatio":"16:9","needVoice":true,"duration":12,"resolution":"4K","videoTag":"品牌宣传","firstFrame":"https://example.com/first_frame.png","lastFrame":"https://example.com/last_frame.png"}'
> [System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))
> ```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateVideo" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary @payload_temp.json
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
| query 超过1000字符 | 缩短需求描述内容 |
| duration 超出范围 | 视频时长需在4-15秒之间 |
| referenceImageStr 图片数量超限 | 最多6张图片，且图片+视频+音频总数不超过6个 |
| referenceVideoStr 视频数量/大小超限 | 最多3条视频，单个不超过50MB，总时长不超过15秒 |
| referenceVideoStr 包含人像 | 禁止人像视频，请更换参考视频 |
| referenceAudioStr 音频数量/大小超限 | 最多3条音频，单个不超过50MB，总时长不超过15秒 |
| firstFrame/lastFrame 与参考素材同时使用 | 首帧/尾帧不能与图片、视频、音频一起使用 |
| 服务繁忙（9999错误码） | 稍后重试 |
| taskStatus=3 生成失败 | 检查提示词质量和素材，尝试简化描述或更换素材 |
| 视频生成超时 | 视频生成耗时较长，增大超时时间并继续轮询 |

## 提示词处理
生成视频时，query 应指导AI：
- 明确描述视频的主题和内容
- 指定产品的关键特性和卖点
- 说明视频的风格和氛围
- 提及需要的视觉元素和效果

**示例query：**
- "为这款智能手表制作一个展示视频，突出运动监测功能"
- "制作一个产品宣传片，展示咖啡机的使用方法和特点"
- "生成一个创意视频，展示智能音箱的语音交互功能"
- "创建一个品牌宣传视频，强调产品的科技感和时尚感"

## 素材使用规则

### 参考素材（referenceImageStr / referenceVideoStr / referenceAudioStr）
- 三种素材总数不能超过6个
- 参考图片：最多6张，单个不超过10MB
- 参考视频：最多3条，单个不超过50MB，总时长不超过15秒，禁止人像
- 参考音频：最多3条，单个不超过50MB，总时长不超过15秒

### 首帧/尾帧图片（firstFrame / lastFrame）
- firstFrame：视频首帧图片，只能上传1张
- lastFrame：视频尾帧图片，只能上传1张
- **重要**：首帧/尾帧图片不能与参考素材（图片、视频、音频）同时使用

### 素材选择建议
- 需要产品参考时，使用 referenceImageStr 提供产品图片
- 需要风格参考时，使用 referenceVideoStr 提供参考视频（禁止人像）
- 需要背景音乐时，使用 referenceAudioStr 提供音频素材
- 需要指定视频开头/结尾画面时，使用 firstFrame 和 lastFrame

## 执行流程
1. 收集用户的视频生成需求并写入 `query`
2. 确定目标平台 `platformType` 和语言 `languageType`
3. 选择视频模型 `videoModelType`：pro（高质量）或 fast（快速生成）
4. 设置视频参数：分辨率、比例、时长、配音、业务标签等
5. 根据需求添加参考素材或首帧/尾帧图片
6. 在请求头中传入 `secretKey`
7. 调用 `generateVideo` 提交任务
8. 从响应中读取 `data.agentGenerateTaskId`
9. 调用 `queryTaskResult` 轮询任务结果
10. 将 `taskStatus=2` 的 `executeResult` 视频 URL 展示给用户