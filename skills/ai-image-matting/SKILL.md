---
name: ai-image-matting
description: >-
  通过 Flyelep AI 工具接口自动去除图片背景，支持单张或批量处理。
  当用户要求抠图、去背景、提取商品主体、生成透明底素材时使用此技能。
---
# Flyelep AI抠图
通过 Flyelep AI Tool API 自动去除图片背景，并返回抠图后的新图片 URL。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**

## API 接口信息
- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/aiImageMatting`
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
  "imgUrls": "https://example.com/img1.jpg,https://example.com/img2.jpg"
}
```

## 响应格式
统一响应结构：

```json
{
  "code": 200,
  "msg": "操作成功",
  "data": "https://example.com/matted1.png,https://example.com/matted2.png"
}
```

- `code=200` 表示调用成功
- `msg` 为接口返回说明
- `data` 为抠图后图片地址
- 多张图片时，`data` 中多个 URL 以英文逗号 `,` 分隔

返回结果应按逗号拆分后逐个展示给用户，不要回读图片内容。

## 参数说明
### 必传参数

> **重要**：以下必传参数必须通过询问用户获取，agent 不可自行填写。调用本技能时，应先向用户列出必传参数与可选参数表格，由用户确认或提供后再执行。
| 字段 | 默认值 | 说明 |
|------|--------|------|
| imgUrls | - | 图片链接字符串，多张时使用英文逗号分隔 |

## 参数映射规则
### imgUrls
- 接口要求传字符串，不是数组
- 单张图片时直接传一个 URL 字符串
- 多张图片时，用英文逗号 `,` 按顺序拼接
- 每个链接都应为公网可访问的图片直链，不要传网页地址

### 结果格式
- 抠图结果通常更适合返回透明背景图片
- 文档示例中返回的是 `.png`，因此应优先预期结果为支持透明底的图片格式
- 若接口实际返回其他格式，按接口返回值原样展示即可

## 调用示例
- **重要**：在 Windows/PowerShell 环境下调用 API 时，必须采用以下流程：**先将请求体 JSON 写入当前工作目录下的临时文件 `payload_temp.json`，再通过 Shell 工具调用 `curl.exe --data-binary @payload_temp.json` 发送请求**。这是因为 PowerShell 使用 GBK 编码，而服务端使用 UTF-8 解析，直接在命令行中嵌入中文 JSON 会导致乱码。同时必须设置 `Content-Type: application/json; charset=utf-8` 请求头。
- **注意**：在 Windows 环境下使用 `curl.exe`（而非 `curl`，后者在 PowerShell 中是 `Invoke-WebRequest` 的别名）。必须在 `curl.exe` 后加 `--%` 停止 PowerShell 解析，否则 `@` 会被误判为 splatting 操作符导致报错。
- **文件创建方式**：根据可用工具选择其一（均需确保 UTF-8 **无 BOM** 编码，否则服务端 JSON 解析会在 position 0 报错）：
  - **方式 A（有 Write 工具）**：使用 Write 工具创建 `payload_temp.json`
  - **方式 B（无 Write 工具）**：使用 Shell 的 .NET API 创建文件（`Set-Content -Encoding UTF8` 会带 BOM，不可用）
- **清理**：API 返回结果后，务必删除 `payload_temp.json` 临时文件。

**示例 1：单张图片抠图**

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "imgUrls": "https://example.com/img1.jpg"
}
```
> 方式 B（无 Write 工具）：
> ```powershell
> $json = '{"imgUrls":"https://example.com/img1.jpg"}'
> [System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))
> ```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/aiImageMatting" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**示例 2：批量图片抠图**

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "imgUrls": "https://example.com/img1.jpg,https://example.com/img2.jpg"
}
```
> 方式 B（无 Write 工具）：
> ```powershell
> $json = '{"imgUrls":"https://example.com/img1.jpg,https://example.com/img2.jpg"}'
> [System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))
> ```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/aiTool/aiImageMatting" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary @payload_temp.json
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
| `imgUrls` 格式错误 | 该字段必须是字符串，多张图用英文逗号分隔，不是 JSON 数组 |
| 图片 URL 无法访问 | 传入的链接不是公网直链、已过期，或源站限制访问 |
| 抠图边缘效果不理想 | 原图主体与背景对比不足、边缘过于复杂，可换更清晰或背景更干净的源图 |
| 请求超时 | 批量图片较多或源图较大时，可适当增大超时时间 |

## 提示词处理
该接口不接收自然语言提示词，不需要构造额外文案。

执行时只需要：

1. 收集一张或多张图片 URL
2. 将多张 URL 用英文逗号拼接为 `imgUrls`
3. 在请求头中传入 `secretKey`
4. 调用接口并将返回的结果按逗号拆分后逐个展示

当用户明确要求“透明底”“抠出主体”“去掉背景”时，优先使用此技能；如果用户真正想要的是“换背景”而不是“去背景”，则更适合使用场景替换或局部重绘类技能。
