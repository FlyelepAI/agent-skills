---
name: main-image-tagging
description: >-
  通过 Flyelep 主图打标 API 复刻一张参考图的版式，把同一组卖点和价格打到一批商品图上，每张商品图各产出一张打标图，同步返回，支持批量。
  当用户要求主图打标、给商品图加卖点和价格、按参考图版式批量做促销主图、加价格标签时使用此技能。
---
# Flyelep 主图打标

复刻一张参考图的版式，把同一组卖点和价格打到商品图上。`sourceUrlList` 里每张商品图各产出一张打标图，参考图、卖点、价格整批共用，生成比例以参考图为准。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**

## API 接口信息

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/mainImageTagging`
- **Content-Type**: `application/json`
- **认证方式**: 在请求头中传入 `secretKey`（密钥需由用户在 Flyelep 开放平台申请：https://www.flyelep.cn/controlboard）
- **超时时间**: 同步接口，处理完才返回，一次最多 20 张，建议 300 秒以上，张数多时再放宽
- **计费**: 固定走特惠通道，成功一张扣 8 算力，失败的不扣；提交前按商品图张数预校验余额，余额不足整批失败

请求头示例：

```http
Content-Type: application/json
secretKey: 用户提供的API密钥
```

> **安全说明**：不要将真实密钥写入技能文件、示例代码仓库或持久化配置中，应在运行时由用户动态提供。

## 请求 Body

```json
{
  "sourceUrlList": [
    "https://example.com/product1.png",
    "https://example.com/product2.png"
  ],
  "referenceImageUrl": "https://example.com/reference-layout.png",
  "sellingPoints": "轻薄透气，速干不粘身",
  "price": "¥799.9",
  "modelType": 9
}
```

## 响应格式

```json
{
  "code": 200,
  "data": [
    "https://xxx.com/tagged1.png",
    "https://xxx.com/tagged2.png"
  ],
  "result": {
    "fileUrlList": [
      "https://xxx.com/tagged1.png",
      "https://xxx.com/tagged2.png"
    ],
    "taskId": "2104856627422584833"
  }
}
```

- **优先读取 `result.fileUrlList`**；`data` 与它相同，仅为兼容老调用方保留
- **部分成功时 `code` 仍是 200**，结果只含成功的图片，失败项被去掉，**不能再按下标和 `sourceUrlList` 一一对应**
- 要知道哪张失败，用 `result.taskId` 调 `POST /prod-api/poster-design/api/v1/poster/queryTaskResult`（Body 传 `agentGenerateTaskId`）查明细
- 全部失败时接口报错，`msg` 为第一条失败原因

## 参数说明

> **重要**：必传参数必须通过询问用户获取，agent 不可自行填写。

| 字段 | 必传 | 说明 |
|------|------|------|
| sourceUrlList | 是 | 商品图地址数组，1~20 张，每张各产出一张打标图 |
| referenceImageUrl | 是 | 版式参考图，只能 1 张，整批共用；生成比例以它为准 |
| sellingPoints | 是 | 商品卖点，原样传给生成流程，整批共用 |
| price | 是 | 商品价格，保留原始展示格式，如 `¥99.9`、`$19.99`，整批共用 |
| modelType | 是 | 只能传整数 `9`（Flyelep Image 2），其他值直接拒绝且不扣算力 |
| secretKey | 是 | API 密钥，放在请求头中 |

### 参数规则
- 所有图片地址必须是服务端能访问的 http/https 链接，服务会读取参考图宽高来定生成比例
- 卖点、价格按用户原话传入，不要自行改写或换算币种
- 不同商品要用不同卖点或价格时，分多次调用
- 本地文件先按「本地文件上传」章节换取公网直链

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

### 示例：两张商品图按参考版式打标

**Windows/PowerShell**：

方式 A（使用 Write 工具）创建 `payload_temp.json`：
```json
{
  "sourceUrlList": ["https://example.com/product1.png", "https://example.com/product2.png"],
  "referenceImageUrl": "https://example.com/reference-layout.png",
  "sellingPoints": "轻薄透气，速干不粘身",
  "price": "¥799.9",
  "modelType": 9
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"sourceUrlList":["https://example.com/product1.png","https://example.com/product2.png"],"referenceImageUrl":"https://example.com/reference-layout.png","sellingPoints":"轻薄透气，速干不粘身","price":"¥799.9","modelType":9}', [System.Text.UTF8Encoding]::new($false))
```

执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/mainImageTagging" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 600 --data-binary @payload_temp.json
```

清理临时文件：`rm payload_temp.json`

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/mainImageTagging" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 600 --data-binary '{"sourceUrlList":["https://example.com/product1.png","https://example.com/product2.png"],"referenceImageUrl":"https://example.com/reference-layout.png","sellingPoints":"轻薄透气，速干不粘身","price":"¥799.9","modelType":9}'
```

## 常见错误及解决方案

| 错误 | 原因与解决 |
|------|-----------|
| `商品图不能为空` / `参考图不能为空` / `商品卖点不能为空` / `商品价格不能为空` / `模型类型不能为空` | 必传参数缺失 |
| `modelType 传入非法值，当前接口仅支持 9(Flyelep Image 2)` | `modelType` 改传 `9` |
| 余额不足 | 提交前按张数预校验，充值或减少张数 |
| 返回张数少于 `sourceUrlList` | 部分失败，用 `result.taskId` 查 `queryTaskResult` 明细，对失败的商品图单独重试 |
| 请求超时 | 同步处理较慢，调大超时或减少单次张数 |

## 执行流程

1. **向用户询问 `secretKey`**（API 密钥必须由用户提供，agent 不可自行填写）
2. 收集商品图（1~20 张）和 1 张版式参考图，本地文件先按「本地文件上传」章节换取公网直链
3. 收集卖点 `sellingPoints` 和价格 `price`，原样传入；`modelType` 固定传 `9`
4. 调用接口（超时 300 秒以上），读取 `result.fileUrlList` 展示给用户
5. 返回张数少于提交张数时，用 `result.taskId` 查明细并告知用户哪些失败
