---
name: image-layer-decompose
description: >-
  通过 Flyelep 图层分离 API 把图片拆成 1 张底图和最多 16 个透明 PNG 图层，每个图层带叠放顺序、位置、名称和描述，便于二次编辑，同步返回，支持批量。
  当用户要求图层分离、图层拆分、拆图层、把海报拆成可编辑图层、分离文字/商品/人物元素时使用此技能。
---
# Flyelep 图层分离

将一张图片拆分为 1 张底图和最多 16 个透明 PNG 图层，每个图层附带叠放顺序、位置、名称和描述，便于二次编辑。同步返回，支持批量。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**

## API 接口信息

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/imageLayerDecompose`
- **Content-Type**: `application/json`
- **认证方式**: 在请求头中传入 `secretKey`（密钥需由用户在 Flyelep 开放平台申请：https://www.flyelep.cn/controlboard）
- **超时时间**: 处理耗时较长，建议 300 秒以上；批量时按张数相应放宽
- **计费**: 按实际产出的图层数计费

请求头示例：

```http
Content-Type: application/json
secretKey: 用户提供的API密钥
```

> **安全说明**：不要将真实密钥写入技能文件、示例代码仓库或持久化配置中，应在运行时由用户动态提供。

## 请求 Body

```json
{
  "imageUrlList": ["https://example.com/poster.png"],
  "prompt": "拆分出人物、商品和文字",
  "size": "auto"
}
```

## 响应格式

```json
{
  "code": 200,
  "data": ["https://xxx.com/base.png", "https://xxx.com/layer1.png"],
  "result": {
    "fileUrlList": ["https://xxx.com/base.png", "https://xxx.com/layer1.png"],
    "taskId": "2103020163319619586",
    "itemList": [
      {
        "sourceUrl": "https://example.com/poster.png",
        "success": true,
        "errorMsg": null,
        "layers": [
          {
            "url": "https://xxx.com/base.png",
            "width": 1536,
            "height": 2048,
            "base": true,
            "zindex": 0
          },
          {
            "url": "https://xxx.com/layer1.png",
            "width": 702,
            "height": 1625,
            "base": false,
            "zindex": 1,
            "name": "橙色户外背包商品",
            "description": "提取左侧橙色户外背包商品本体，保留原有纹理细节",
            "boundingBox": {
              "absolute": [0, 0, 702, 1625],
              "normalized": [0, 0, 456, 793]
            }
          }
        ]
      }
    ]
  }
}
```

- **优先读取 `result`**；`data` 与 `result.fileUrlList` 相同，仅为兼容老调用方保留
- `result.fileUrlList`：全部图层地址，按输入图顺序拉平
- `result.taskId`：整批共用的主任务 ID，可调用 `/poster/queryTaskResult` 查明细
- `result.itemList[]`：按输入图分组的明细，顺序与 `imageUrlList` 一致
  - `sourceUrl`：原图地址
  - `success` / `errorMsg`：该张是否拆分成功及失败原因。**批量时部分失败不影响其他图片，要逐项检查**
  - `layers[]`：按叠放顺序从下到上排列，**第一个为底图**（`base=true`、`zindex=0`）
    - `url`：图层图片地址（透明 PNG）
    - `width` / `height`：图层像素尺寸
    - `zindex`：叠放顺序，数值越大越靠上
    - `name` / `description`：图层名称与描述（底图没有）
    - `boundingBox.absolute`：图层在底图中的位置 `[x1, y1, x2, y2]`，像素，原点为左上角（底图没有）
    - `boundingBox.normalized`：同一位置归一化到 0~1000 的坐标

## 参数说明

> **重要**：必传参数必须通过询问用户获取，agent 不可自行填写。

| 字段 | 必传 | 默认值 | 说明 |
|------|------|--------|------|
| imageUrlList | 是 | - | 待拆分图片 URL 数组，1~20 张，每张各拆成一组图层 |
| size | 建议传 | `auto` | 输出分辨率档位：`auto` / `1K` / `1.5K` / `2K`。文档标为必填，用户未指定时传 `auto` |
| prompt | 否 | - | 拆分意图，如「拆出人物、商品和文字」；不传则自动识别图中所有主要元素，整批共用 |
| secretKey | 是 | - | API 密钥，放在请求头中 |

### 参数规则
- 单张图片分辨率需在 512×512 ~ 6000×6000 之间，过小或过大的图先缩放
- 用户只想拆某几类元素时写进 `prompt`；想全部拆开就不传
- 图片 URL 需公网可访问；本地文件先按「本地文件上传」章节换取公网直链

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

### 示例：拆分海报中的人物、商品和文字

**Windows/PowerShell**：

方式 A（使用 Write 工具）创建 `payload_temp.json`：
```json
{
  "imageUrlList": ["https://example.com/poster.png"],
  "prompt": "拆分出人物、商品和文字",
  "size": "auto"
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"imageUrlList":["https://example.com/poster.png"],"prompt":"拆分出人物、商品和文字","size":"auto"}', [System.Text.UTF8Encoding]::new($false))
```

执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/imageLayerDecompose" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 360 --data-binary @payload_temp.json
```

清理临时文件：`rm payload_temp.json`

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/imageLayerDecompose" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 360 --data-binary '{"imageUrlList":["https://example.com/poster.png"],"prompt":"拆分出人物、商品和文字","size":"auto"}'
```

## 常见错误及解决方案

| 错误 | 原因与解决 |
|------|-----------|
| `图片链接不能为空` | 未传 `imageUrlList` 或为空数组 |
| `itemList[].success=false` | 该张拆分失败，看 `errorMsg`；常见原因是分辨率不在 512×512 ~ 6000×6000 之间或图片无法访问 |
| HTTP 401 / `密钥无效!` | 密钥无效或已过期，向用户重新索取 |
| 请求超时 | 拆分耗时较长，超时调到 300 秒以上，批量时减少单次张数 |

## 执行流程

1. **向用户询问 `secretKey`**（API 密钥必须由用户提供，agent 不可自行填写）
2. 收集待拆分图片（1~20 张），本地文件先按「本地文件上传」章节换取公网直链
3. 确认是否有拆分要求写入 `prompt`；`size` 未指定时传 `auto`
4. 调用接口（超时 300 秒以上）
5. 逐项检查 `result.itemList[].success`：成功的按 `layers` 顺序（底图在前、`zindex` 由小到大）展示图层地址和名称；失败的告知 `errorMsg`
6. 用户要在画布中还原时，按 `boundingBox.absolute` 把图层贴回底图对应位置
