---
name: long-image-fusion
description: >-
  通过 Flyelep 多图融合 API 把多张图片拼接融合为一张长图，并对拼接处做自然过渡处理，同步返回结果。
  当用户要求多图融合、拼长图、把几张图拼成详情页长图、图片拼接融合时使用此技能。
---
# Flyelep 多图融合

将多张图片融合为一张长图，并把拼接处进行融合处理，同步返回结果图地址。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**

## API 接口信息

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/longImageFusion`
- **Content-Type**: `application/json`
- **认证方式**: 在请求头中传入 `secretKey`（密钥需由用户在 Flyelep 开放平台申请：https://www.flyelep.cn/controlboard）
- **超时时间**: 处理耗时较长，建议 300 秒以上
- **计费**: 按成功生成的张数计费

请求头示例：

```http
Content-Type: application/json
secretKey: 用户提供的API密钥
```

> **安全说明**：不要将真实密钥写入技能文件、示例代码仓库或持久化配置中，应在运行时由用户动态提供。

## 请求 Body

```json
{
  "imageUrlList": [
    "https://example.com/part1.png",
    "https://example.com/part2.png"
  ]
}
```

## 响应格式

```json
{
  "code": 200,
  "msg": null,
  "data": ["https://xxx.com/fusion-result.png"],
  "result": {
    "fileUrlList": ["https://xxx.com/fusion-result.png"],
    "taskId": "2103017888991000000"
  }
}
```

- `code=200` 表示成功，失败时原因在 `msg` 里
- **优先读取 `result.fileUrlList`**：融合结果图地址数组
- `result.taskId`：生图主任务 ID，可调用 `POST /prod-api/poster-design/api/v1/poster/queryTaskResult`（Body 传 `agentGenerateTaskId`）查明细
- `data` 与 `result.fileUrlList` 内容相同，仅为兼容老调用方保留

## 参数说明

> **重要**：必传参数必须通过询问用户获取，agent 不可自行填写。

| 字段 | 必传 | 说明 |
|------|------|------|
| imageUrlList | 是 | 待融合图片 URL 数组，**至少 2 张**，按数组顺序从上到下拼接融合成一张长图 |
| secretKey | 是 | API 密钥，放在请求头中 |

### 参数规则
- 图片 URL 需公网可访问
- 数组顺序就是拼接顺序，先问清用户希望的上下顺序
- 只有 1 张图时不要调用，接口会返回 `图片链接至少需要2张`
- 如果用户提供本地文件路径，先按「本地文件上传」章节换取公网直链

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

### 示例：融合 3 张详情图为一张长图

**Windows/PowerShell**：

方式 A（使用 Write 工具）创建 `payload_temp.json`：
```json
{
  "imageUrlList": [
    "https://example.com/part1.png",
    "https://example.com/part2.png",
    "https://example.com/part3.png"
  ]
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"imageUrlList":["https://example.com/part1.png","https://example.com/part2.png","https://example.com/part3.png"]}', [System.Text.UTF8Encoding]::new($false))
```

执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/longImageFusion" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 360 --data-binary @payload_temp.json
```

清理临时文件：`rm payload_temp.json`

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/longImageFusion" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 360 --data-binary '{"imageUrlList":["https://example.com/part1.png","https://example.com/part2.png","https://example.com/part3.png"]}'
```

## 常见错误及解决方案

| 错误 | 原因与解决 |
|------|-----------|
| `图片链接至少需要2张` | `imageUrlList` 少于 2 张，补充图片 |
| HTTP 401 / `密钥无效!` | 密钥无效或已过期，向用户重新索取 |
| 请求超时 | 融合耗时较长，把超时调到 300 秒以上 |
| 图片无法访问 | URL 不是公网直链或已过期（上传直链 24 小时有效），重新上传 |

## 执行流程

1. **向用户询问 `secretKey`**（API 密钥必须由用户提供，agent 不可自行填写）
2. 收集至少 2 张待融合图片，确认拼接顺序；本地文件先按「本地文件上传」章节换取公网直链
3. 按顺序写入 `imageUrlList`，在请求头中传入 `secretKey`
4. 调用接口（超时 300 秒以上），读取 `result.fileUrlList` 展示给用户
