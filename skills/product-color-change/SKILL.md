---
name: product-color-change
description: >-
  通过 Flyelep AI 工具接口智能识别图片中的商品并进行换色处理。
  当用户要求修改商品颜色、保持商品不变只换配色、生成同款不同颜色展示图时使用此技能。
---
# Flyelep 商品换色
通过 Flyelep AI Tool API 对图片中的商品进行换色处理，并返回换色后的新图片 URL。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**

## API 接口信息

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/productColorChange`
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
  "sourceUrl": "https://example.com/product_red.jpg",
  "textPrompt": "将商品颜色改为深蓝色",
  "modelType": 0
}
```

## 响应格式

```json
{
  "code": 200,
  "msg": "操作成功",
  "data": "https://example.com/product_blue.jpg"
}
```

- `code=200` 表示调用成功
- `msg` 为接口返回说明
- `data` 为换色后的图片 URL

返回结果应直接展示给用户，不要回读图片内容。

## 参数说明

### 必传参数

> **重要**：以下必传参数必须通过询问用户获取，agent 不可自行填写。调用本技能时，应先向用户列出必传参数与可选参数表格，由用户确认或提供后再执行。

| 字段 | 默认值 | 说明 |
|------|--------|------|
| sourceUrl | - | 原图链接 |
| textPrompt | - | 换色提示词，如"将商品颜色改为深蓝色" |
| modelType | - | 模型类型：无论任何情况，都默认填 `0` |

### 参数映射规则

**sourceUrl**：
- 传入待换色商品的原图公网 URL
- 必须是图片直链，不要传网页地址
- 原图应尽量清晰展示商品主体和原始颜色
- 如果用户提供本地文件路径，先调用 image-upload 技能上传文件获取公网链接，再填入此参数

**textPrompt**：
- 文档将其标为必需
- 直接描述目标颜色及必要约束
- 应尽量明确"将什么改成什么颜色"

推荐写法示例：

- `将商品颜色改为深蓝色`
- `把包包主体颜色改为奶油白，保留金属扣件颜色不变`
- `将耳机外壳换成哑光黑色，保持材质质感与光影不变`
- `把杯身改为浅绿色，保留品牌标识和背景不变`

**提示词边界**：
- 优先描述颜色，不要把换色需求扩写成换材质或换商品
- 如果用户只是想"更换商品"，应改用商品替换 skill
- 如果用户想"换背景"，应改用场景替换 skill

> **说明**：场景替换、商品替换、商品换色三个接口共用同一 DTO，由接口内部自动设置 `type` 字段，调用方无需传入 `type`。

## 调用示例

> **跨平台调用说明**：
> - 请求头必须包含 `Content-Type: application/json; charset=utf-8` 和 `secretKey`
> - **Windows/PowerShell**：因 GBK 编码问题，必须先将 JSON 写入临时文件 `payload_temp.json`（UTF-8 无 BOM），再用 `curl.exe --% --data-binary @payload_temp.json` 发送请求。使用 Write 工具创建文件，或用 .NET API `[System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))`。调用后用 `rm payload_temp.json` 清理。
> - **macOS/Linux**：bash/zsh 默认 UTF-8，可直接内联 JSON：`curl -X POST URL -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --data-binary 'JSON单行内容'`

### 示例 1：基础商品换色

**前置步骤**：向用户索取图片路径或 URL。如用户提供本地文件，先调用 image-upload 技能上传获取公网链接。

**Windows/PowerShell**：

步骤 1：创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "sourceUrl": "https://example.com/product_red.jpg",
  "textPrompt": "将商品颜色改为深蓝色",
  "modelType": 0
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"sourceUrl":"https://example.com/product_red.jpg","textPrompt":"将商品颜色改为深蓝色","modelType":0}', [System.Text.UTF8Encoding]::new($false))
```

步骤 2：执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/productColorChange" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/productColorChange" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary '{"sourceUrl":"https://example.com/product_red.jpg","textPrompt":"将商品颜色改为深蓝色","modelType":0}'
```

### 示例 2：强调保留材质与光影的换色

**前置步骤**：向用户索取图片路径或 URL。如用户提供本地文件，先调用 image-upload 技能上传获取公网链接。

**Windows/PowerShell**：

步骤 1：创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "sourceUrl": "https://example.com/product_watch.jpg",
  "textPrompt": "将表带改为深棕色皮革观感，保留金属表盘和整体光影不变",
  "modelType": 0
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"sourceUrl":"https://example.com/product_watch.jpg","textPrompt":"将表带改为深棕色皮革观感，保留金属表盘和整体光影不变","modelType":0}', [System.Text.UTF8Encoding]::new($false))
```

步骤 2：执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/productColorChange" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/productColorChange" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary '{"sourceUrl":"https://example.com/product_watch.jpg","textPrompt":"将表带改为深棕色皮革观感，保留金属表盘和整体光影不变","modelType":0}'
```

## 常见错误及解决方案

| 错误 | 原因与解决 |
|------|-----------|
| HTTP 401 / `code` 非 200 | `secretKey` 无效、缺失或已过期，确认请求头是否正确传入 |
| HTTP 405 Not Allowed | 请求方法错误，必须使用 `POST` |
| `sourceUrl` 无法访问 | 原图 URL 不是公网直链、已过期，或源站限制访问 |
| `modelType` 非 0 | 模型类型只支持 `0` |
| 换色结果偏差较大 | `textPrompt` 过于模糊，可补充目标颜色、材质观感和保留项 |
| 局部也被错误换色 | 原图主体边界不清晰，可换更干净的源图或在提示词里强调保留范围 |
| 请求超时 | 图片较大或处理复杂时，可适当增大超时时间 |

## 执行流程

1. **向用户询问 `secretKey`**（API 密钥必须由用户提供，agent 不可自行填写）
2. 收集原图 URL（如用户提供本地文件，先调用 image-upload 技能上传获取公网链接）
3. 与用户确认换色需求，构造 `textPrompt`，`modelType` 默认填 `0`
4. 在请求头中传入 `secretKey`，调用接口
5. 将返回的换色结果图片 URL 直接展示给用户

该接口支持 `textPrompt`，商品换色的结果高度依赖提示词描述质量。执行时应遵循：

1. 明确目标颜色
2. 明确保留项：材质、品牌标识、背景、光影、构图
3. 避免把"换色"写成"换商品"或"换背景"
4. 对多部件商品可明确指定仅修改哪个部位

当用户要求"同款不同色""把红色改成蓝色"时，优先使用此技能；如果用户想替换为完全不同的商品，应改用商品替换 skill。
