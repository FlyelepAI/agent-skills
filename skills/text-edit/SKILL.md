---
name: text-edit
description: >-
  通过 Flyelep 文字编辑 API 在单张图片上按原文→新文逐条替换文字，尽量保持原有字体、颜色和版式，同步返回结果图。
  当用户要求改图片上的字、替换海报文案、修改图片文字、把图上的英文换成别的内容时使用此技能。
---
# Flyelep 文字编辑

在单张原图上按 `textReplaceList` 逐条替换文字，并尽量保持原有版式。一次只处理一张图，不支持批量。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**

## API 接口信息

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/textEdit`
- **Content-Type**: `application/json`
- **认证方式**: 在请求头中传入 `secretKey`（密钥需由用户在 Flyelep 开放平台申请：https://www.flyelep.cn/controlboard）
- **超时时间**: 同步接口，建议 300 秒以上
- **计费**: 固定走特惠通道，成功返回结果图后扣 8 算力，失败不扣

请求头示例：

```http
Content-Type: application/json
secretKey: 用户提供的API密钥
```

> **安全说明**：不要将真实密钥写入技能文件、示例代码仓库或持久化配置中，应在运行时由用户动态提供。

## 请求 Body

```json
{
  "sourceUrl": "https://example.com/poster.png",
  "textReplaceList": [
    {
      "index": 0,
      "originalText": "HEAVY-DUTY",
      "replaceText": "HELLO",
      "rect": [47, 60, 557, 109]
    },
    {
      "index": 1,
      "originalText": "ROAD ROLLER",
      "replaceText": "VERY GOOD",
      "rect": [47, 174, 553, 100]
    }
  ],
  "modelType": "9"
}
```

## 响应格式

```json
{
  "code": 200,
  "data": "https://xxx.com/text-edited.png",
  "result": {
    "fileUrlList": ["https://xxx.com/text-edited.png"],
    "taskId": "2104876037654000000"
  }
}
```

- **优先读取 `result.fileUrlList`**（长度为 1 的数组）；`data` 是同一张结果图的地址字符串，为兼容老调用方保留
- `result.taskId`：主任务 ID，可调用 `POST /prod-api/poster-design/api/v1/poster/queryTaskResult`（Body 传 `agentGenerateTaskId`）复查

## 参数说明

> **重要**：必传参数必须通过询问用户获取，agent 不可自行填写。

| 字段 | 必传 | 默认值 | 说明 |
|------|------|--------|------|
| sourceUrl | 是 | - | 待编辑原图，只能 1 张，需服务端能访问的 http/https 链接 |
| textReplaceList | 是 | - | 原文与替换文列表，至少 1 条 |
| textReplaceList[].originalText | 是 | - | 原图上的原文 |
| textReplaceList[].replaceText | 是 | - | 要换成的文字 |
| textReplaceList[].rect | 是 | - | 原文位置 `[x, y, w, h]`：左上角 x、左上角 y、宽、高，单位像素，原点在图片左上角 |
| textReplaceList[].index | 否 | - | 同一张图里原文重复时用于区分，建议从 0 开始 |
| modelType | 否 | `"9"` | **字符串**，只支持 `"9"`（Flyelep Image 2），建议固定传 `"9"` |
| secretKey | 是 | - | API 密钥，放在请求头中 |

### 参数规则
- `rect` 是这一张图上的像素坐标，多张图的坐标不能共用；换图必须重新获取坐标
- 原文和坐标最可靠的来源是 `image-text-recognize` 技能（`/aiTool/recognizeImageText`）：识别结果 `textList[i]` 的 `text` 作 `originalText`、`index` 作 `index`，四角坐标换算为 `rect = [leftUp.x, leftUp.y, rightDown.x - leftUp.x, rightDown.y - leftUp.y]`（文字倾斜时取四角的最小/最大值）
- 只放需要改的行；`replaceText` 与 `originalText` 相同的行没有意义，不要放
- 服务会读取原图宽高来定生成比例
- 请求里可以带 `languageType`，但当前生成流程不使用它，不需要传

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
> - **macOS/Linux**：bash/zsh 默认 UTF-8，可直接内联 JSON。

### 示例：先识别再改字

**步骤 1**：按 `image-text-recognize` 技能识别原图，拿到 `textList`。

**步骤 2**：用户确认把 `HEAVY-DUTY` 改成 `HELLO` 后，组装请求。

**Windows/PowerShell**：

方式 A（使用 Write 工具）创建 `payload_temp.json`：
```json
{
  "sourceUrl": "https://example.com/poster.png",
  "textReplaceList": [
    { "index": 0, "originalText": "HEAVY-DUTY", "replaceText": "HELLO", "rect": [47, 60, 557, 109] }
  ],
  "modelType": "9"
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"sourceUrl":"https://example.com/poster.png","textReplaceList":[{"index":0,"originalText":"HEAVY-DUTY","replaceText":"HELLO","rect":[47,60,557,109]}],"modelType":"9"}', [System.Text.UTF8Encoding]::new($false))
```

执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/textEdit" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 360 --data-binary @payload_temp.json
```

清理临时文件：`rm payload_temp.json`

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/textEdit" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 360 --data-binary '{"sourceUrl":"https://example.com/poster.png","textReplaceList":[{"index":0,"originalText":"HEAVY-DUTY","replaceText":"HELLO","rect":[47,60,557,109]}],"modelType":"9"}'
```

## 常见错误及解决方案

| 错误 | 原因与解决 |
|------|-----------|
| `原图不能为空` | 未传 `sourceUrl` |
| `原文和替换文的映射列表不能为空` | `textReplaceList` 为空 |
| `modelType 不受支持` | `modelType` 只能传 `"9"` 或不传 |
| 改错位置或没改动 | `rect` 不准或来自另一张图，重新调用文字识别获取坐标 |
| 请求超时 | 同步处理较慢，把超时调到 300 秒以上 |

## 执行流程

1. **向用户询问 `secretKey`**（API 密钥必须由用户提供，agent 不可自行填写）
2. 收集原图（1 张），本地文件先按「本地文件上传」章节换取公网直链
3. 用户没给坐标时，先调用 `image-text-recognize` 技能识别文字，把识别结果列给用户
4. 让用户确认要改哪几行、改成什么，组装 `textReplaceList`（`originalText`、`replaceText`、`rect`、`index`）
5. 调用接口（超时 300 秒以上），读取 `result.fileUrlList[0]` 展示给用户
