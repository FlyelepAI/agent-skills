---
name: scene-replace
description: >-
  通过 Flyelep AI 工具接口将图片中的背景场景替换为指定场景，可结合参考图和文本提示词精准控制效果。
  当用户要求更换背景场景、替换商品展示环境、保留主体改场景时使用此技能。
---
# Flyelep 场景替换
通过 Flyelep AI Tool API 将图片中的背景场景替换为目标场景，并返回替换后的新图片 URL。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**

## API 接口信息

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/sceneReplace`
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
  "sourceUrl": "https://example.com/product.jpg",
  "replaceImageUrl": "https://example.com/scene.jpg",
  "modelType": 9,
  "textPrompt": "将场景替换成我上传的图片"
}
```

## 响应格式

```json
{
  "code": 200,
  "msg": "操作成功",
  "data": "https://example.com/scene_replaced.jpg"
}
```

- `code=200` 表示调用成功
- `msg` 为接口返回说明
- `data` 为场景替换后的图片 URL

返回结果应直接展示给用户，不要回读图片内容。

## 参数说明

### 必传参数

> **重要**：以下必传参数必须通过询问用户获取，agent 不可自行填写。调用本技能时，应先向用户列出必传参数与可选参数表格，由用户确认或提供后再执行。

| 字段 | 类型 | 是否必需 | 默认值 | 说明 |
|------|------|----------|--------|------|
| sourceUrl | String | 是 | - | 原图链接 |
| modelType | Integer | 是 | 9 | 当前仅支持传 `9`：Flyelep Image 2 |
| textPrompt | String | 是 | - | 用户提示词，描述目标场景 |
| replaceImageUrl | String | 否 | - | 场景参考图链接，暂时只支持单图 |

### 参数映射规则

**sourceUrl**：
- 传入待替换场景的原图公网 URL
- 必须是图片直链，不要传网页地址
- 如果用户提供本地文件路径，先调用 file-upload 技能上传文件获取公网链接，再填入此参数

**modelType**：
- 当前接口仅支持 `9`（Flyelep Image 2），必传
- 传其他值会报「无效的模型类型」

**textPrompt**：
- 用自然语言描述目标场景，如风格、环境、光线、氛围、陈列方式
- 有参考图时用于补充约束（如「将场景替换成我上传的图片」），无参考图时由文字单独控制目标场景
- 不传时接口会退化为默认提示词「替换场景」，效果不可控，因此实际调用应始终传入

**replaceImageUrl**：
- 可选参数，用于提供目标场景参考图
- 暂时只支持单图，传入多张（逗号分隔）会被接口拒绝，报「该操作类型最多只能上传1张图片」
- 不传时仅靠 `textPrompt` 生成目标场景
- 如果用户提供本地文件路径，先调用 file-upload 技能上传文件获取公网链接，再填入此参数

> **说明**：场景替换、商品替换、商品换色三个接口共用同一 DTO，由接口内部自动设置 `type` 字段，调用方无需传入 `type`。

## 调用示例

> **跨平台调用说明**：
> - 请求头必须包含 `Content-Type: application/json; charset=utf-8` 和 `secretKey`
> - **Windows/PowerShell**：因 GBK 编码问题，必须先将 JSON 写入临时文件 `payload_temp.json`（UTF-8 无 BOM），再用 `curl.exe --% --data-binary @payload_temp.json` 发送请求。使用 Write 工具创建文件，或用 .NET API `[System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))`。调用后用 `rm payload_temp.json` 清理。
> - **macOS/Linux**：bash/zsh 默认 UTF-8，可直接内联 JSON：`curl -X POST URL -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --data-binary 'JSON单行内容'`

### 示例：结合参考图与文本描述替换场景

**前置步骤**：向用户索取原图和场景参考图的路径或 URL。如用户提供本地文件，先调用 file-upload 技能上传获取公网链接。

**Windows/PowerShell**：

步骤 1：创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "sourceUrl": "https://example.com/product.jpg",
  "replaceImageUrl": "https://example.com/scene1.jpg",
  "modelType": 9,
  "textPrompt": "室内现代风格展厅，暖色灯光，突出高级陈列感"
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"sourceUrl":"https://example.com/product.jpg","replaceImageUrl":"https://example.com/scene1.jpg","modelType":9,"textPrompt":"室内现代风格展厅，暖色灯光，突出高级陈列感"}', [System.Text.UTF8Encoding]::new($false))
```

步骤 2：执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/sceneReplace" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/sceneReplace" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary '{"sourceUrl":"https://example.com/product.jpg","replaceImageUrl":"https://example.com/scene1.jpg","modelType":9,"textPrompt":"室内现代风格展厅，暖色灯光，突出高级陈列感"}'
```

## 常见错误及解决方案

| 错误 | 原因与解决 |
|------|-----------|
| HTTP 401 / `code` 非 200 | `secretKey` 无效、缺失或已过期，确认请求头是否正确传入 |
| HTTP 405 Not Allowed | 请求方法错误，必须使用 `POST` |
| `sourceUrl` 无法访问 | 原图 URL 不是公网直链、已过期，或源站限制访问 |
| `无效的模型类型` | `modelType` 不在支持范围，当前仅支持 `9` |
| `该操作类型最多只能上传1张图片` | `replaceImageUrl` 传了多张，场景替换只接受 1 张 |
| `textPrompt` 未传 | 提示词为必传参数，需要描述目标场景 |
| 场景效果不理想 | 文字描述过于模糊，可补充风格、光线、空间类型、氛围等信息 |
| 请求超时 | 原图较大、参考图较多或生成复杂时，可适当增大超时时间 |

## 执行流程

1. **向用户询问 `secretKey`**（API 密钥必须由用户提供，agent 不可自行填写）
2. 收集原图 URL，以及可选的场景参考图 URL（如用户提供本地文件，先调用 file-upload 技能上传获取公网链接）
3. 与用户确认目标场景需求，构造 `textPrompt`，`modelType` 固定填 `9`
4. 在请求头中传入 `secretKey`，调用接口
5. 将返回的场景替换结果图片 URL 直接展示给用户

该接口支持通过 `textPrompt` 控制目标场景，因此提示词质量很重要。执行时应遵循：

1. 明确保留项：主体商品、角度、构图、光影关系
2. 明确替换项：背景环境、风格、空间、色温、陈列方式
3. `textPrompt` 为必需；`replaceImageUrl` 可选，提供后与提示词共同控制生成效果

当用户要求"换背景场景但保留产品不变"时，提示词应明确写出"保留主体不变"；如果用户真正想改的是商品本身而不是背景，应改用商品替换类 skill。
