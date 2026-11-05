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
- **超时时间**: 建议 300 秒以上（同步返回，批量图片处理耗时更长）

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
  "msg": null,
  "data": [
    "https://example.com/result1.png",
    "https://example.com/result2.png"
  ],
  "result": {
    "fileUrlList": [
      "https://example.com/result1.png",
      "https://example.com/result2.png"
    ],
    "taskId": "2103021460300001"
  }
}
```

- `code=200` 表示调用成功；业务失败时 HTTP 状态码仍是 200，`code` 非 200，原因在 `msg` 里
- **优先读取 `result.fileUrlList`**：延展后的图片 URL 数组，按 `imgUrlList` 的输入顺序排列
- `result.taskId`：本次请求的生图主任务 ID，可用 `/poster/queryTaskResult` 查询任务明细；按字符串保存，不要做数值运算
- `data`：兼容旧调用方保留，本接口为图片 URL 数组，与 `result.fileUrlList` 内容一致
- 返回结果应逐个展示给用户，不要回读图片内容

## 参数说明

### 必传参数

> **重要**：以下必传参数必须通过询问用户获取，agent 不可自行填写。调用本技能时，应先向用户列出必传参数与可选参数表格，由用户确认或提供后再执行。

| 字段 | 默认值 | 说明 |
|------|--------|------|
| imgUrlList | - | 源图片 URL 数组，1~20 张，单张不超过 10MB |
| ratio | - | 目标延展比例 |

### 可选参数

| 字段 | 默认值 | 说明 |
|------|--------|------|
| modelType | `"9"` | 模型类型，字符串：`"9"`=Flyelep Image 2（默认），`"2"`=Nano Banana 2，`"0"`=Nano Banana |

### 参数映射规则

**imgUrlList**：
- 接口要求传字符串数组，每个元素为一张图片 URL；字段名是 `imgUrlList`，不是 `imageUrlList`
- 单次 1~20 张，超过 20 张需分批调用
- 单张图片不超过 10MB，超过时服务端直接报错「图片大小不能超过10MB，请压缩后重试」
- 每个链接都应为公网可访问的图片直链，不要传网页地址
- 保留用户原始图片顺序，不要擅自重排 `imgUrlList`
- 如果用户提供本地文件路径，先按「本地文件上传」章节换取公网直链，再填入此参数

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

**modelType**：
- 该字段是**字符串**，不是数字，例如 `"9"`
- 只传 `"9"`、`"2"`、`"0"` 三个值之一；不传或传空串时服务端按 `"9"`（Flyelep Image 2）处理
- 用户没有指定模型时直接省略该字段

**计费**：
- 按提交的图片张数计费，与所选模型无关
- 只要有一张延展成功就按提交张数整单扣费；全部失败时报错、不扣费

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

### 示例 1：单张图片延展为横版 16:9

**前置步骤**：向用户索取图片路径或 URL。如用户提供本地文件，先按「本地文件上传」章节换取公网直链。

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

**前置步骤**：向用户索取图片路径或 URL。如用户提供本地文件，先按「本地文件上传」章节换取公网直链。

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
| `图片列表不能为空` | `imgUrlList` 未传或为空数组（注意字段名是 `imgUrlList`），至少传入一张图片 URL |
| `单次最多支持批量处理20张图片` | 超过 20 张，拆成多批调用 |
| `图片大小不能超过10MB，请压缩后重试` | 源图超过 10MB，压缩后重新上传 |
| 比例不支持 | `ratio` 只能使用接口支持的固定枚举值 |
| 积分不足 | 账户算力不足，充值后重试 |
| 处理超时 | 批量图片较多或源图较大时可适当增大超时时间（建议 300 秒以上） |
| 返回数组数量异常 | 先核对源图 URL 是否可访问，再重试请求 |

## 执行流程

1. **向用户询问 `secretKey`**（API 密钥必须由用户提供，agent 不可自行填写）
2. 收集用户提供的图片 URL 列表（1~20 张；如用户提供本地文件，先按「本地文件上传」章节换取公网直链）
3. 确定目标比例 `ratio`；用户指定模型时再传 `modelType`
4. 在请求头中传入 `secretKey`，调用接口
5. 读取 `result.fileUrlList`（缺失时回退 `data` 数组），按输入顺序展示返回的图片 URL，并记录 `result.taskId` 备查

该接口不接收自然语言提示词，不需要构造额外文案。保留用户原始图片顺序，不要擅自重排 `imgUrlList`。
