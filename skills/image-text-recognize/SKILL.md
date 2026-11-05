---
name: image-text-recognize
description: >-
  通过 Flyelep 文字识别 API 识别单张图片里的文字，返回每行文字内容和四角坐标，通常作为文字编辑（text-edit）的前一步。
  当用户要求识别图片文字、OCR、提取图片上的文案、查看图片里有哪些字、准备改图片上的字时使用此技能。
---
# Flyelep 图片文字识别

识别单张图片里的文字，返回每行文字的内容和四角坐标。它一般是文字编辑（`text-edit` 技能，`/aiTool/textEdit`）的前一步：先拿到原文和位置，再组装 `textReplaceList` 去改字。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**

## API 接口信息

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/recognizeImageText`
- **Content-Type**: `application/json; charset=utf-8`
- **认证方式**: 在请求头中传入 `secretKey`（密钥需由用户在 Flyelep 开放平台申请：https://www.flyelep.cn/controlboard）
- **超时时间**: 建议 60-120 秒
- **计费**: 识别成功每次扣 1 算力，识别失败不扣

请求头示例：

```http
Content-Type: application/json; charset=utf-8
secretKey: 用户提供的API密钥
```

> **安全说明**：不要将真实密钥写入技能文件、示例代码仓库或持久化配置中，应在运行时由用户动态提供。

## 请求 Body

```json
{
  "imageUrl": "https://example.com/poster.png"
}
```

## 响应格式

```json
{
  "code": 200,
  "data": {
    "imageUrl": "https://example.com/poster.png",
    "count": 2,
    "taskId": "2104876037654000000",
    "textList": [
      {
        "index": 0,
        "text": "HEAVY-DUTY",
        "rect": {
          "leftUp": { "x": 47, "y": 60 },
          "rightUp": { "x": 604, "y": 60 },
          "rightDown": { "x": 604, "y": 169 },
          "leftDown": { "x": 47, "y": 169 }
        }
      },
      {
        "index": 1,
        "text": "ROAD ROLLER",
        "rect": {
          "leftUp": { "x": 47, "y": 174 },
          "rightUp": { "x": 600, "y": 174 },
          "rightDown": { "x": 600, "y": 274 },
          "leftDown": { "x": 47, "y": 274 }
        }
      }
    ]
  }
}
```

- `data.imageUrl`：原图地址
- `data.count`：识别到的文字行数
- `data.taskId`：本次请求的主任务 ID
- `data.textList[]`：每行文字
  - `index`：行序号，从 0 开始
  - `text`：文字内容
  - `rect`：四角坐标 `leftUp` / `rightUp` / `rightDown` / `leftDown`，单位像素，原点在图片左上角

## 参数说明

| 字段 | 必传 | 说明 |
|------|------|------|
| imageUrl | 是 | 待识别图片的公网地址，只支持单张 |
| secretKey | 是 | API 密钥，放在请求头中 |

## 换算成文字编辑的入参

调用 `text-edit` 技能时，把每一行识别结果换算成 `textReplaceList` 的一项：

| textEdit 字段 | 取值 |
|---------------|------|
| `originalText` | `textList[i].text` |
| `index` | `textList[i].index` |
| `rect` | `[x, y, w, h]`：`x = min(leftUp.x, leftDown.x)`，`y = min(leftUp.y, rightUp.y)`，`w = max(rightUp.x, rightDown.x) - x`，`h = max(leftDown.y, rightDown.y) - y` |
| `replaceText` | 用户要改成的文字 |

以上面第一行为例：`rect = [47, 60, 604-47, 169-60] = [47, 60, 557, 109]`。

- 只把用户要改的行放进 `textReplaceList`，不改的行不要放
- 文字编辑必须使用**同一张图**的坐标，换图后要重新识别

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
> - **Windows/PowerShell**：先将 JSON 写入临时文件 `payload_temp.json`（UTF-8 无 BOM），再用 `curl.exe --% --data-binary @payload_temp.json` 发送请求，调用后用 `rm payload_temp.json` 清理。
> - **macOS/Linux**：bash/zsh 默认 UTF-8，可直接内联 JSON。

**Windows/PowerShell**：

```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"imageUrl":"https://example.com/poster.png"}', [System.Text.UTF8Encoding]::new($false))
```

```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/recognizeImageText" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary @payload_temp.json
```

清理临时文件：`rm payload_temp.json`

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/recognizeImageText" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary '{"imageUrl":"https://example.com/poster.png"}'
```

## 常见错误及解决方案

| 错误 | 原因与解决 |
|------|-----------|
| `图片链接不能为空` | 未传 `imageUrl` |
| `图片文字识别算力配置未配置` | 平台侧配置问题，重试无效，让用户联系平台 |
| HTTP 401 / `密钥无效!` | 密钥无效或已过期，向用户重新索取 |
| `count=0` | 图片中没有可识别文字，或文字过小、模糊 |

## 执行流程

1. **向用户询问 `secretKey`**（API 密钥必须由用户提供，agent 不可自行填写）
2. 收集 1 张图片，本地文件先按「本地文件上传」章节换取公网直链
3. 调用接口，按 `index` 顺序把识别到的文字列给用户
4. 用户要改字时，让用户指出要改的行和新文字，按「换算成文字编辑的入参」组装 `textReplaceList`，再调用 `text-edit` 技能
