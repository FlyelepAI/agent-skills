---
name: partial-redrawing
description: >-
  通过 Flyelep AI 工具接口对图片局部区域进行重绘，可结合文本提示词和参考替换图生成新图。
  当用户要求局部修改图片、替换背景、替换某个区域内容、保留主体仅改局部时使用此技能。
---
# Flyelep 局部重绘
通过 Flyelep AI Tool API 对图片指定区域进行局部重绘，并返回重绘后的新图片 URL。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**

## API 接口信息
- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/partialRedrawing`
- **Content-Type**: `application/json`
- **认证方式**: 在请求头中传入 `secretKey`
- **超时时间**: 建议 120-300 秒

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
  "sourceUrl": "https://example.com/original.jpg",
  "textPrompt": "将背景替换为夏日海滩",
  "replaceImageUrl": "https://example.com/reference.jpg"
}
```

## 响应格式
统一响应结构：

```json
{
  "code": 200,
  "msg": "操作成功",
  "data": "https://example.com/redrawn.jpg"
}
```

- `code=200` 表示调用成功
- `msg` 为接口返回说明
- `data` 为重绘后的图片 URL

返回结果应直接展示给用户，不要回读图片内容。

## 参数说明
### 必传参数

> **重要**：以下必传参数必须通过询问用户获取，agent 不可自行填写。调用本技能时，应先向用户列出必传参数与可选参数表格，由用户确认或提供后再执行。

| 字段 | 默认值 | 说明 |
|------|--------|------|
| sourceUrl | - | 原图链接 |
| textPrompt | - | 用户提示词，用于描述重绘内容 |

### 可选参数
| 字段 | 默认值 | 说明 |
|------|--------|------|
| replaceImageUrl | - | 参考替换图片链接 |

## 参数映射规则
### sourceUrl
- 传入待重绘原图的公网可访问 URL
- 必须是图片直链，不要传网页地址

### textPrompt
- 直接描述要重绘的内容、目标风格和替换效果
- 应尽量明确“改哪里、改成什么、保留什么”
- 优先保留用户原始意图，不要无故扩写成完全不同的需求

推荐写法示例：

- `将背景替换为纯白背景，保留商品主体和阴影`
- `将杯子上的图案改为蓝色几何纹理，保留杯身材质`
- `把右上角的文字替换为英文促销文案，整体风格保持简洁`

### replaceImageUrl
- 当用户提供明确的参考替换图时再传入
- 适合用于“把某一区域替换成参考图风格或内容”的场景
- 用户未提供参考图时，不传此字段

> **说明**：文档描述中提到“基于掩码图对图片指定区域进行局部重绘”，但当前参数表仅列出 `sourceUrl`、`textPrompt`、`replaceImageUrl`，没有单独的掩码字段。因此本 skill 按文档可见参数执行，不额外构造 mask 参数。

## 调用示例
- **重要**：在 Windows/PowerShell 环境下调用 API 时，必须采用以下流程：**先将请求体 JSON 写入当前工作目录下的临时文件 `payload_temp.json`，再通过 Shell 工具调用 `curl.exe --data-binary @payload_temp.json` 发送请求**。这是因为 PowerShell 使用 GBK 编码，而服务端使用 UTF-8 解析，直接在命令行中嵌入中文 JSON 会导致乱码。同时必须设置 `Content-Type: application/json; charset=utf-8` 请求头。
- **注意**：在 Windows 环境下使用 `curl.exe`（而非 `curl`，后者在 PowerShell 中是 `Invoke-WebRequest` 的别名）。必须在 `curl.exe` 后加 `--%` 停止 PowerShell 解析，否则 `@` 会被误判为 splatting 操作符导致报错。
- **文件创建方式**：根据可用工具选择其一（均需确保 UTF-8 **无 BOM** 编码，否则服务端 JSON 解析会在 position 0 报错）：
  - **方式 A（有 Write 工具）**：使用 Write 工具创建 `payload_temp.json`
  - **方式 B（无 Write 工具）**：使用 Shell 的 .NET API 创建文件（`Set-Content -Encoding UTF8` 会带 BOM，不可用）
- **清理**：API 返回结果后，务必删除 `payload_temp.json` 临时文件。

**示例 1：仅通过文字提示进行局部重绘**

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "sourceUrl": "https://example.com/original.jpg",
  "textPrompt": "将背景替换为夏日海滩"
}
```
> 方式 B（无 Write 工具）：
> ```powershell
> $json = '{"sourceUrl":"https://example.com/original.jpg","textPrompt":"将背景替换为夏日海滩"}'
> [System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))
> ```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/partialRedrawing" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**示例 2：结合参考图进行局部替换**

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "sourceUrl": "https://example.com/product.jpg",
  "textPrompt": "将背景更换为更高级的木质桌面场景，保留产品主体不变",
  "replaceImageUrl": "https://example.com/wood-scene.jpg"
}
```
> 方式 B（无 Write 工具）：
> ```powershell
> $json = '{"sourceUrl":"https://example.com/product.jpg","textPrompt":"将背景更换为更高级的木质桌面场景，保留产品主体不变","replaceImageUrl":"https://example.com/wood-scene.jpg"}'
> [System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))
> ```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/partialRedrawing" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary @payload_temp.json
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
| `sourceUrl` 无法访问 | 原图 URL 不是公网直链、已过期，或源站限制访问 |
| `textPrompt` 过于模糊 | 提示词没有说明要修改的区域或目标效果，应补充“改哪里、改成什么” |
| `replaceImageUrl` 无法访问 | 参考图 URL 无效或不可公开访问，去掉该字段或更换可访问链接 |
| 重绘结果偏差较大 | 提示词不够具体，可补充材质、颜色、构图、保留元素等约束 |
| 请求超时 | 源图较大或处理复杂时，可适当增大超时时间 |

## 提示词处理
该接口接收自然语言提示词，`textPrompt` 的质量会直接影响结果。

执行时应遵循：

1. 明确保留项：主体、品牌标识、材质、构图等不应被误改的内容
2. 明确修改项：背景、局部文案、某个物体、某种颜色或纹理
3. 有参考图时再传 `replaceImageUrl`
4. 若用户只说“帮我改一下图”，应先补足最少必要信息，再调用接口

当用户目标是“小范围替换”时，提示词应避免写成整张图重做；当用户目标是“换背景”时，应在 `textPrompt` 中强调保留主体不变。
