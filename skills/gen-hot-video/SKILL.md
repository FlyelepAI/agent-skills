---
name: gen-hot-video
description: >-
  通过 Flyelep 爆款视频复刻 API，基于爆款视频风格生成产品复刻视频。
  当用户要求复刻爆款视频、模仿参考视频风格生成产品视频时使用此技能。
---
# Flyelep 爆款视频复刻
通过 Flyelep AI Tool API 复刻爆款视频，把商品素材（图片 / 视频 / 音频）融合到爆款参考视频中。支持两种模式：创意复刻（默认，参考爆款风格重新创作）与 1:1 复刻（严格照搬原视频镜头与节奏，只替换指定对象）。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**
**注意：此接口为异步接口，只返回任务ID，需要通过视频结果查询接口 `videoGen/queryResult` 获取最终结果（不要用图片类的 `queryTaskResult`）。**

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

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/videoGen/queryResult`
- **超时时间**: 建议 30 秒
- **说明**: 视频类任务专用查询接口，返回格式与图片类 `queryTaskResult` 相同

## 请求 Body

创建任务（创意复刻，默认模式）：
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

创建任务（1:1 复刻）：
```json
{
  "replicaMode": "oneToOne",
  "replaceType": "product",
  "replaceUrl": "https://example.com/product.png",
  "sourceUrl": "https://example.com/hot_video.mp4",
  "modelType": "pro",
  "resolution": "720p"
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

- `code=200` 表示调用成功；业务失败（参数校验、算力不足等）时 HTTP 状态码仍是 200，`code` 非 200，原因在 `msg` 里
- `data.agentGenerateTaskId` 为异步任务ID（字符串，避免长整型精度丢失），原样用于后续查询
- `data.taskList[].taskStatus`: 0-待生成，1-生成中，2-生成成功，3-生成失败；每次只生成 1 条，看 `taskList[0]` 即可
- `data.taskList[].executeResult`: 成功时为视频 URL，失败时为失败原因
- `data.status`（整体状态，1 进行中 / 2 全部成功 / 3 全部失败 / 4 部分成功）：**不为 1 即可停止轮询**
- 将结果视频展示给用户

## 参数说明

### 必传参数

> **重要**：以下必传参数必须通过询问用户获取，agent 不可自行填写。调用本技能时，应先向用户列出必传参数与可选参数表格，由用户确认或提供后再执行。

| 字段 | 默认值 | 说明 |
|------|--------|------|
| replaceUrl | - | 商品素材地址（http/https），英文逗号分隔，支持图片（≤9 张）、视频（≤3 个）、音频（≤3 个） |
| sourceUrl | - | 爆款参考视频地址（http/https），建议 4-15 秒 |
| modelType | - | 模型类型：`pro`=Flyelep Dance 2.0，`fast`=Flyelep Dance 2.0 Fast，`ultra`=Flyelep Dance 2.5 |
| resolution | - | 分辨率，线上必传（不传会报「分辨率(resolution)仅支持…」），推荐 `720p` |
| prompt | - | 提示词，最多1000个字符长度；**创意复刻必传**，1:1 复刻可不传 |
| language | - | 生成语言；**创意复刻必传**，1:1 复刻不使用 |
| duration | - | 视频时长（秒），4-15；**创意复刻必传**，1:1 复刻不传时按原视频时长成片 |

### 可选参数

| 字段 | 默认值 | 说明 |
|------|--------|------|
| replicaMode | creative | 复刻模式：`creative` 创意复刻 / `oneToOne` 1:1 复刻 |
| additionalPrompt | - | 补充提示词（可选，追加到最终提示词） |
| ratio | - | 视频生成比例；1:1 复刻不传时按原视频宽高自动匹配 |
| subtitle | - | 是否添加字幕，`true`/`false` |
| replaceType | product | 仅 1:1 复刻生效：替换对象，见下方说明 |
| productSubject | - | 仅 1:1 复刻生效：商品主体名称，最多 10 个字，如「上衣」 |
| keepMusicRhythm | true | 仅 1:1 复刻生效：是否保留原视频音乐与节奏 |
| oralBroadcast | false | 仅 1:1 复刻生效：是否口播视频（人物对镜讲解、口型与语音同步） |

### 参数映射规则

**replicaMode**（复刻模式）：
- `creative`（默认，不传即此模式）：创意复刻，参考爆款视频的风格与剧本重新创作，`prompt`、`language`、`duration` 必传
- `oneToOne`：1:1 复刻，严格照搬原视频的镜头、动作与节奏，只把 `replaceType` 指定的对象换成商品素材；`prompt`、`language`、`duration` 都可不传
- 大小写不敏感，传其它值接口直接拒绝

**replaceType**（替换对象，仅 1:1 复刻生效）：
- `product`（默认）：商品，参考图最多 1 张
- `preciseProduct`：精准商品（逐帧严格一致），参考图最多 3 张（可传多角度图）
- `model`：模特（整个人替换），参考图最多 1 张
- `precisePerson`：精准人物，参考图最多 1 张
- `face`：人脸，参考图最多 1 张
- `clothing`：服饰，参考图最多 2 张（正反面）
- `background`：背景，参考图最多 1 张
- 参考图超过该类型上限时报「XX类型的参考图片(replaceUrl)最多支持N张」

**modelType**（模型类型）：
- `pro`：Flyelep Dance 2.0（高质量）
- `fast`：Flyelep Dance 2.0 Fast（快速生成）
- `ultra`：Flyelep Dance 2.5（最高画质）
- 只接受这三个取值，大小写不敏感；传其它值接口直接拒绝

**resolution**（分辨率）：
- 支持：`480p`、`720p`、`1080p`（大小写不敏感）
- `fast` 仅支持 `480p`、`720p`；`ultra` 建议也只用 `480p`、`720p`
- 线上实际为必传，不传会报「分辨率(resolution)仅支持 480p / 720p / 1080p / 2K / 4K」

**ratio**（视频比例）：
- 支持：`1:1`、`3:4`、`4:3`、`16:9`、`9:16`、`21:9`
- 1:1 复刻不传时，按原视频宽高自动匹配最接近的比例（探测失败按 `9:16`）

**duration**（视频时长）：
- 范围：4-15 秒，传了就校验，超出报「视频时长(duration)仅支持 4-15 秒」
- 创意复刻必传，按该时长成片
- 1:1 复刻可不传，服务端按原视频真实时长成片（`pro`/`fast` 最长 15 秒，`ultra` 最长 30 秒）

**计费**：生成成功后按成片时长（秒）扣除算力，单价随分辨率与模型档位不同，见官网功能定价；提交时会预检算力是否足够

**language**（生成语言）：
- 仅支持：中文简体、中文繁体、英文、马来语、葡萄牙语、韩语、日语、西班牙语、俄语
- 必须原样使用上面的中文标签（例如是 `英文` 而不是 `英语`），传其它写法会被拒绝

**replaceUrl**（商品素材）：
- 商品素材地址，用于替换爆款视频中的产品/人物等，必须是 http/https 链接
- 多个用英文逗号分隔，按后缀自动识别类型：图片最多 9 张、视频最多 3 个、音频最多 3 个（1:1 复刻的图片张数另受 `replaceType` 限制）
- 地址必须带可识别的文件后缀（如 `.png`、`.mp4`、`.mp3`），否则报「replaceUrl包含无法识别的参考素材格式」
- 如果用户提供本地文件路径，先按「本地文件上传」章节换取公网直链，再填入此参数

**sourceUrl**（爆款参考）：
- 爆款参考视频地址，必须是 http/https 链接
- 建议时长 4-15 秒
- 1:1 复刻时，原视频与 `replaceUrl` 中的视频合计不超过 15 秒（`ultra` 30 秒），单条不短于 2 秒，否则提交时直接报错
- 如果用户提供本地文件路径，先按「本地文件上传」章节换取公网直链，再填入此参数

**productSubject / keepMusicRhythm / oralBroadcast**（仅 1:1 复刻生效）：
- `productSubject`：商品主体名称，最多 10 个字；传了就直接使用，不再识图判断主体
- `keepMusicRhythm`：默认 `true`，舞蹈、卡点、BGM 类视频建议保留
- `oralBroadcast`：默认 `false`，口播类视频传 `true`

**subtitle**（字幕控制）：
- `true`：添加字幕
- `false`：不添加字幕
- 通过追加提示词可控制字幕内容

## 异步任务流程

> **重要**：本接口为异步接口，必须**先调用主接口获取 `agentGenerateTaskId`，然后调用 `videoGen/queryResult` 接口轮询任务结果**，不能省略轮询步骤。

1. 调用主接口（`generateHotVideo`）提交任务，从响应 `data.agentGenerateTaskId` 获取任务ID
2. 使用 `agentGenerateTaskId` 调用 `videoGen/queryResult` 接口轮询任务结果（视频耗时较长，建议每 10 秒查询一次）
3. `taskStatus` 为 0 或 1（或 `data.status=1`）时继续轮询
4. 当 `taskStatus=2` 时，表示生成成功，`executeResult` 为视频 URL
5. 当 `taskStatus=3` 时，表示生成失败，`executeResult` 为失败原因，停止轮询

> **轮询策略**：视频生成耗时较长，整体超时建议设置为 20 分钟

## 本地文件上传

用户提供的是本地文件路径而不是公网直链时，先把文件上传换取直链，再调用本接口。已安装 `file-upload` 技能时以该技能为准；未安装时按下面的说明直接调用上传接口。

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/file/upload`
- **请求方式**: `multipart/form-data`，文件字段名固定为 `file`，单次只能上传一个文件，多个文件并发调用多次
- **认证方式**: 请求头传 `secretKey`，与本技能使用同一个密钥
- **超时时间**: 视频体积大、上传耗时长，建议 300 秒
- **不要手动设置 `Content-Type` 请求头**，让 HTTP 客户端自动生成带 boundary 的值，手写会导致服务端解析失败
- 视频仅支持 `mp4`、`mov`、`m4v`、`webm`、`avi`、`mkv`。**文件名必须带正确后缀**，服务端靠后缀判断格式，`mov`、`webm`、`mkv`、`m4v` 缺后缀时无法靠 Content-Type 回退，会被判为格式不支持
- 视频不做内容审核，直接入库，不要上传来源不明的内容
- 原文件名不会出现在 URL 里，中文名、空格、特殊字符都能直接上传，不需要提前改名
- 上传不消耗算力，但服务端不做去重：同一个文件在一次任务里只上传一次，记下 `fullPath` 复用

成功响应取 `data.fullPath` 作为公网直链，不带签名，**只保证 24 小时内可访问**，拿到后应尽快用于生成任务：

```json
{
  "code": 200,
  "msg": null,
  "data": {
    "relativePath": "cos_ai_agent_h_24/2026-11-05/3f2a9c1b7d84e6f5a012.mp4",
    "fullPath": "https://agent-1404002717.cos.ap-guangzhou.myqcloud.com/cos_ai_agent_h_24/2026-11-05/3f2a9c1b7d84e6f5a012.mp4",
    "serviceProvider": null
  }
}
```

判断成功只看 `code`，业务失败时 HTTP 状态码仍是 200，`code` 为 500 或 9999，原因在 `msg` 里。

```bash
# Windows/PowerShell（用 curl.exe，PowerShell 里的 curl 是 Invoke-WebRequest 的别名）
curl.exe -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/file/upload" -H "secretKey: 你的密钥" --max-time 300 -F "file=@C:/path/to/reference.mp4"

# macOS/Linux
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/file/upload" -H "secretKey: 你的密钥" --max-time 300 -F "file=@./reference.mp4"
```

拿到 `code=9999`、`msg` 为 `服务繁忙，请稍后再试` 时，先自查三项：是否漏了 `secretKey` 请求头、表单字段名是否为 `file`、文件是否超出服务端体积上限（超限只会返回这条通用错误，此时应先压缩或裁剪再重试，不要原样重传）。密钥、格式、体积类错误重试无效，只有网络超时、5xx 和存储类异常值得重试。

## 调用示例

> **跨平台调用说明**：
> - 请求头必须包含 `Content-Type: application/json; charset=utf-8` 和 `secretKey`
> - **Windows/PowerShell**：因 GBK 编码问题，必须先将 JSON 写入临时文件 `payload_temp.json`（UTF-8 无 BOM），再用 `curl.exe --% --data-binary @payload_temp.json` 发送请求。使用 Write 工具创建文件，或用 .NET API `[System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))`。调用后用 `rm payload_temp.json` 清理。
> - **macOS/Linux**：bash/zsh 默认 UTF-8，可直接内联 JSON：`curl -X POST URL -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --data-binary 'JSON单行内容'`

### 示例 1：完整流程 - 提交任务并查询结果

**前置步骤**：向用户索取产品素材视频和爆款参考视频的路径或 URL。如用户提供本地文件，先按「本地文件上传」章节换取公网直链。

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

步骤 5：执行查询请求（每 10 秒左右轮询，直到 `taskStatus` 为 2 或 3）：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/videoGen/queryResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary @payload_temp.json
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

查询结果（每 10 秒左右轮询，直到 `taskStatus` 为 2 或 3）：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/videoGen/queryResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary '{"agentGenerateTaskId":"2072923591164715009"}'
```

### 示例 2：快速模式复刻（fast模型）

**前置步骤**：向用户索取产品素材视频和爆款参考视频的路径或 URL。如用户提供本地文件，先按「本地文件上传」章节换取公网直链。

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

### 示例 3：1:1 复刻（替换商品）

**前置步骤**：向用户索取商品图和爆款参考视频的路径或 URL，确认替换对象 `replaceType`。如用户提供本地文件，先按「本地文件上传」章节换取公网直链。1:1 复刻不需要 `prompt`、`language`、`duration`，成片时长跟随原视频。

**Windows/PowerShell**：

步骤 1：创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "replicaMode": "oneToOne",
  "replaceType": "product",
  "productSubject": "保温杯",
  "replaceUrl": "https://example.com/product.png",
  "sourceUrl": "https://example.com/hot_video.mp4",
  "modelType": "pro",
  "resolution": "720p",
  "keepMusicRhythm": true
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"replicaMode":"oneToOne","replaceType":"product","productSubject":"保温杯","replaceUrl":"https://example.com/product.png","sourceUrl":"https://example.com/hot_video.mp4","modelType":"pro","resolution":"720p","keepMusicRhythm":true}', [System.Text.UTF8Encoding]::new($false))
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
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/generateHotVideo" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary '{"replicaMode":"oneToOne","replaceType":"product","productSubject":"保温杯","replaceUrl":"https://example.com/product.png","sourceUrl":"https://example.com/hot_video.mp4","modelType":"pro","resolution":"720p","keepMusicRhythm":true}'
```

查询结果（参考示例 1 的查询结果命令，替换 `agentGenerateTaskId`）

## 常见错误及解决方案

| 错误 | 原因与解决 |
|------|-----------|
| HTTP 401 / `code` 非 200 | `secretKey` 无效、缺失或已过期，确认请求头是否正确传入 |
| HTTP 405 Not Allowed | 请求方法错误，必须使用 `POST` |
| `参考素材不能为空` / `原视频不能为空` | `replaceUrl`、`sourceUrl` 必传 |
| `…地址格式不合法，必须为 http/https 链接` | `replaceUrl`、`sourceUrl` 必须是 http/https 公网链接，多个用英文逗号分隔 |
| `replaceUrl包含无法识别的参考素材格式` | 素材链接需带 `.png`/`.jpg`/`.mp4`/`.mp3` 等可识别后缀 |
| 参考图片/视频/音频数量超限 | 图片≤9、视频≤3、音频≤3；1:1 复刻图片张数另受 `replaceType` 限制 |
| `模型类型不能为空` 或模型类型不合法 | `modelType` 必传，只能是 `pro`、`fast`、`ultra` |
| `分辨率(resolution)仅支持 …` | `resolution` 必传，使用 `480p`、`720p` 或 `1080p`（fast 仅 480p/720p） |
| `视频时长(duration)仅支持 4-15 秒` / `视频时长(duration)必须大于0` | 创意复刻必须传 4-15 秒的 `duration` |
| 提示词或语言为空 | 创意复刻模式下 `prompt`、`language` 必传 |
| 语言不支持 | `language` 必须原样使用支持列表中的中文标签，如 `英文` 而不是 `英语` |
| `1:1复刻的参考视频总时长不能超过15秒` / `不能短于2秒` | 裁剪原视频（ultra 上限 30 秒），或更换素材 |
| prompt 超过1000字符 | 缩短提示词内容 |
| 服务繁忙（9999错误码） | 稍后重试 |
| taskStatus=3 生成失败 | 失败原因在 `executeResult` 里；检查视频素材质量，尝试更换素材或调整prompt |
| 视频生成超时 | 视频生成耗时较长，增大超时时间并继续轮询 |
| 查询一直不出结果 | 确认查询的是 `videoGen/queryResult` 而不是图片类 `queryTaskResult`，`agentGenerateTaskId` 按字符串原样传入 |

## 执行流程

1. **向用户询问 `secretKey`**（API 密钥必须由用户提供，agent 不可自行填写）
2. 收集商品素材 URL（图片/视频/音频）和爆款参考视频 URL（如用户提供本地文件，先按「本地文件上传」章节换取公网直链）
3. 与用户确认复刻模式 `replicaMode`：创意复刻（默认）或 1:1 复刻
4. 创意复刻：构造 `prompt` 和 `additionalPrompt`，选择 `modelType`、`resolution`、`ratio`、`duration`、`language`；1:1 复刻：选择 `replaceType`、`modelType`、`resolution`，按需填 `productSubject`、`keepMusicRhythm`、`oralBroadcast`、`ratio`
5. 在请求头中传入 `secretKey`，调用创建任务接口，获取 `data.agentGenerateTaskId`
6. 使用 `agentGenerateTaskId` 轮询 `videoGen/queryResult`（每 10 秒左右一次），`taskStatus=2` 时取视频 URL，`taskStatus=3` 时停止并告知失败原因
7. 将返回的结果视频展示给用户

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
