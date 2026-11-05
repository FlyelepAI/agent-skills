---
name: product-replace
description: >-
  通过 Flyelep AI 工具接口将图片中的商品替换为指定商品图，同时保留原图背景及光影效果。
  当用户要求替换商品主体、保留原场景换产品、保持背景不变更换展示商品时使用此技能。
---
# Flyelep 商品替换
通过 Flyelep AI Tool API 将图片中的商品替换为目标商品，并返回替换后的新图片 URL。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**

## API 接口信息

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/productReplace`
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
  "sourceUrl": "https://example.com/scene_with_old_product.jpg",
  "replaceImageUrl": "https://example.com/new_product.jpg",
  "modelType": 9,
  "textPrompt": "保留背景和光影，将主体商品替换为新的白色保温杯"
}
```

## 响应格式

```json
{
  "code": 200,
  "data": "https://example.com/product_replaced.jpg",
  "result": {
    "fileUrlList": ["https://example.com/product_replaced.jpg"],
    "taskId": "2103029456521xxxx"
  }
}
```

- `code=200` 表示调用成功；失败时 `code` 非 200，原因在 `msg` 里（成功时 `msg` 通常不返回）
- `result.fileUrlList`：替换结果图片 URL 数组，**优先读取这个字段**（单张也是长度为 1 的数组）
- `result.taskId`：本次生图的主任务 ID（字符串），可用 `/poster/queryTaskResult` 查询任务明细
- `data`：兼容旧版的字段，值与 `fileUrlList` 一致，是单个 URL 字符串（多张时英文逗号分隔）；新接入请读 `result.fileUrlList`

返回结果应直接展示给用户，不要回读图片内容。

## 参数说明

### 必传参数

> **重要**：以下必传参数必须通过询问用户获取，agent 不可自行填写。调用本技能时，应先向用户列出必传参数与可选参数表格，由用户确认或提供后再执行。

| 字段 | 默认值 | 说明 |
|------|--------|------|
| sourceUrl | - | 原图链接，包含原始商品的场景图，只传 1 张 |
| replaceImageUrl | - | 目标商品图链接，最多 3 张，多张用英文逗号分隔 |
| textPrompt | - | 用户提示词 |
| modelType | 9 | 模型类型：当前仅支持传 `9`（Flyelep Image 2） |

### 参数映射规则

**sourceUrl**：
- 传入待替换商品的原图公网 URL
- 必须是图片直链，不要传网页地址
- 原图中应清楚包含待替换商品和原背景环境
- 只传 1 张原图，不要用逗号拼接多张；输出比例按原图宽高自动匹配
- 不传或传空会报「原图URL不能为空」
- 如果用户提供本地文件路径，先按「本地文件上传」章节换取公网直链，再填入此参数

**replaceImageUrl**：
- 用于提供目标商品图，必传（接口不会拦截空值，但不传时模型拿不到目标商品，结果不可控）
- 最多 3 张，多张用英文逗号 `,` 拼接；超过 3 张接口报「产品替换最多只能上传3张图片」
- 同一商品的多角度图一起传，有助于模型还原商品细节
- 当用户明确说"把原商品换成另一件商品"时，优先传入该字段
- 如果用户提供本地文件路径，先按「本地文件上传」章节换取公网直链，再填入此参数

**textPrompt**：
- 用于补充替换要求，例如材质、颜色、角度、尺寸观感、保留方式
- 可用于强调"保留原场景、保留光影、保留构图"
- 不传时接口会退化为默认提示词「替换商品」，实际调用应始终传入

**modelType**：
- 当前接口仅支持 `9`（Flyelep Image 2），必传
- 不传或传其他值会报「modelType 传入非法值，当前接口仅支持 9(Flyelep Image 2)」

**计费**：生成成功后才扣算力，每次调用按 1 张计费；失败不扣。

推荐写法示例：

- `保留背景和桌面反光，将商品替换为黑色蓝牙耳机`
- `保持原场景与阴影效果，将主体换成白色保温杯`
- `保留背景展台不变，将中间产品替换为新的香水瓶，风格保持高级简洁`

> **说明**：场景替换、商品替换、商品换色三个接口共用同一 DTO，由接口内部自动设置 `type` 字段，调用方无需传入 `type`。

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

### 示例：结合目标商品图与文本约束替换商品

**前置步骤**：向用户索取原图和目标商品图的路径或 URL。如用户提供本地文件，先按「本地文件上传」章节换取公网直链。

**Windows/PowerShell**：

步骤 1：创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "sourceUrl": "https://example.com/scene_with_old_product.jpg",
  "replaceImageUrl": "https://example.com/new_product_front.jpg,https://example.com/new_product_side.jpg",
  "modelType": 9,
  "textPrompt": "将商品替换为我上传的图片，颜色为红色"
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"sourceUrl":"https://example.com/scene_with_old_product.jpg","replaceImageUrl":"https://example.com/new_product_front.jpg,https://example.com/new_product_side.jpg","modelType":9,"textPrompt":"将商品替换为我上传的图片，颜色为红色"}', [System.Text.UTF8Encoding]::new($false))
```

步骤 2：执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/productReplace" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/productReplace" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary '{"sourceUrl":"https://example.com/scene_with_old_product.jpg","replaceImageUrl":"https://example.com/new_product_front.jpg,https://example.com/new_product_side.jpg","modelType":9,"textPrompt":"将商品替换为我上传的图片，颜色为红色"}'
```

## 常见错误及解决方案

| 错误 | 原因与解决 |
|------|-----------|
| HTTP 401 / `code` 非 200 | `secretKey` 无效、缺失或已过期，确认请求头是否正确传入 |
| HTTP 405 Not Allowed | 请求方法错误，必须使用 `POST` |
| `原图URL不能为空` | 漏传 `sourceUrl` 或传了空串 |
| `sourceUrl` 无法访问 | 原图 URL 不是公网直链、已过期，或源站限制访问 |
| `replaceImageUrl` 无法访问 | 目标商品图 URL 无效、不可公开访问，或链接格式不正确 |
| `modelType 传入非法值，当前接口仅支持 9(Flyelep Image 2)` | `modelType` 漏传或不是 `9`，固定传 `9` |
| `产品替换最多只能上传3张图片` | `replaceImageUrl` 传了超过 3 张，删减后重试 |
| `积分不足，请充值或联系管理员` 等算力不足提示 | 账户算力不够，充值后重试 |
| `…处理失败` | 生成失败（不扣算力），可换更清晰的图或调整提示词后重试 |
| 替换结果不像目标商品 | 目标商品图不够清晰或角度不足，可增加参考图（最多 3 张）并补充 `textPrompt` |
| 商品替换后背景不协调 | 提示词未强调保留原背景和光影，可在 `textPrompt` 中补充说明 |
| 请求超时 | 原图较大、参考商品图较多或生成复杂时，可适当增大超时时间 |

## 执行流程

1. **向用户询问 `secretKey`**（API 密钥必须由用户提供，agent 不可自行填写）
2. 收集原图 URL 和目标商品图 URL（如用户提供本地文件，先按「本地文件上传」章节换取公网直链）
3. 与用户确认替换需求，构造 `textPrompt`，`modelType` 固定填 `9`
4. 在请求头中传入 `secretKey`，调用接口
5. 读取 `result.fileUrlList` 中的结果图片 URL 直接展示给用户（旧字段 `data` 仅作兼容）

该接口支持 `textPrompt`，但在商品替换场景下，目标商品图通常比纯文字更关键。执行时应遵循：

1. 优先保证 `sourceUrl` 清晰展示原场景和原商品
2. 优先提供 `replaceImageUrl`，帮助模型准确识别目标商品
3. 通过 `textPrompt` 强调保留项：背景、光影、角度、构图、摆放位置
4. 通过 `textPrompt` 补充目标商品要求：颜色、材质、风格、展示方式

当用户真正想改的是"背景场景"而不是"商品主体"时，应改用场景替换 skill；当用户只是想换颜色而不是换商品，应改用商品换色类 skill。
