---
name: generate-poster
description: >-
  通过 Flyelep API 生成电商产品主图和详情图海报。
  当用户要求生成产品图、电商海报、Amazon 商品图、详情页图片时使用此技能。
---
# Flyelep 电商海报生图
通过 Flyelep AI API 异步生成电商产品主图和详情图海报。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**

## API 接口信息

### 创建任务

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateAsync`
- **Content-Type**: `application/json`
- **认证方式**: 在请求 JSON body 中传入 `secretKey`
- **超时时间**: 建议 120 秒（创建任务为瞬时操作）

### 查询结果

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult`
- **Content-Type**: `application/json`
- **认证方式**: 在请求 Header 中传入 `secretKey`
- **说明**: 异步接口，需轮询获取最终图片 URL（生图耗时较长，可能需要数分钟）

## 认证方式

在请求 JSON body 的 `secretKey` 字段中传入 API 密钥。用户需在 Flyelep 平台（https://www.flyelep.cn）获取密钥。

> **安全说明**：将 `secretKey` 放在 JSON body 中是 Flyelep API 的设计要求。API 密钥仅在请求时传递给 Flyelep 服务器，不存储在技能代码中。请勿将真实的密钥直接写在示例代码中，运行时由用户动态提供。

## 请求 Body

```json
{
  "query": "生图需求描述，最多1000个字符",
  "generateType": 200,
  "posterType": 5,
  "platformType": "amazon",
  "languageType": "英文",
  "detailPictureNumber": 10,
  "modelEdition": 3,
  "needText": true,
  "secretKey": "用户提供的API密钥",
  "fileUrlList": ["https://example.com/product.png"],
  "aspectRatio": "1:1"
}
```

## 响应格式

### 创建任务响应

```json
{
  "code": 200,
  "data": "20374084144913*****"
}
```

`data` 为异步任务 ID，用于后续轮询获取结果。

### 查询结果响应

```json
{
  "code": 200,
  "data": {
    "taskList": [
      {
        "taskStatus": 2,
        "executeResult": "https://xxx.com/image1.png"
      },
      {
        "taskStatus": 2,
        "executeResult": "https://xxx.com/image2.png"
      }
    ]
  }
}
```

- `taskStatus`: 0-待生成，1-生成中，2-生成成功，3-生成失败
- `executeResult` 为生成的图片 URL
- 将成功的图片逐个展示给用户，不要回读图片内容

## 参数说明

### 必传参数

> **重要**：以下必传参数必须通过询问用户获取，agent 不可自行填写。调用本技能时，应先向用户列出必传参数与可选参数表格，由用户确认或提供后再执行。

| 字段 | 默认值 | 说明 |
|------|--------|------|
| query | - | 生图需求描述，最多1000个字符 |
| generateType | 200 | 100=产品单图，200=产品详情图 |
| posterType | 5 | 5=跨境电商，6=中文电商 |
| platformType | amazon | 电商平台（见下方映射表） |
| languageType | 英文 | 生成图片上的文案语种 |
| detailPictureNumber | 10 | 产品单图固定为1；详情图可选5、10、15 |
| modelEdition | 3 | 2=Flyelep 2.0，3=Flyelep 3.0，9=Flyelep Image 2 |
| needText | true | 图片上是否包含文案 |
| secretKey | - | API 密钥 |

### 可选参数

| 字段 | 默认值 | 说明 |
|------|--------|------|
| fileUrlList | - | 参考图片 URL 数组，最多6张 |
| aspectRatio | 随机 | 图片比例：1:1、3:2、2:3、3:4、4:3、4:5、5:4、9:16、16:9、21:9 |

## 参数映射规则

### platformType（电商平台）
- 跨境电商：`amazon`、`Temu`、`Shopee`、`TikTok Shop`、`AliExpress`、`OZON`
- 中文电商：`淘宝`、`京东`、`拼多多`、`1688`、`小红书`

### languageType（文案语种）
- 跨境：英文、俄语、日语、韩语、阿拉伯语、德语、西班牙语、法语、泰语、马来语、越南语、葡萄牙语、菲律宾语、印尼语、意大利语、荷兰语、波兰语、中文繁体
- 中文：中文简体

### posterType（海报类型）
根据市场区域选择，再搭配 modelEdition：
- 跨境电商 → `posterType=5`
- 中文电商 → `posterType=6`

### aspectRatio（图片比例）
- 正方形：`1:1`
- 横版：`3:2`、`4:3`、`16:9`、`21:9`
- 竖版：`2:3`、`3:4`、`4:5`、`5:4`、`9:16`
- 未提及比例 → 不传此字段（API 自动随机选择）

### detailPictureNumber（图片数量）
- 产品单图（`generateType=100`）：固定为 `1`
- 产品详情图（`generateType=200`）：`5`、`10` 或 `15`

## 异步任务流程

生成图片为异步流程，需要：
1. 调用 `generateAsync` 提交任务，获取任务 ID
2. 轮询调用 `queryTaskResult` 查询任务状态（建议每 10 秒查询一次）
3. 当 `taskStatus=2` 时，表示生成成功，获取结果图片 URL

### 查询任务结果接口
- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult`
- **Header**: `secretKey: 用户提供的API密钥`
- **请求体**:
```json
{
  "agentGenerateTaskId": "任务ID"
}
```

- **轮询策略**：建议每5-10秒查询一次，视频生成耗时较长，超时时间建议设置为20分钟

## 调用示例

- **重要**：调用 API 时，必须设置 `Content-Type: application/json; charset=utf-8` 请求头。以下分平台说明：
- **Windows/PowerShell 环境**：
  - 必须采用以下流程：**先将请求体 JSON 写入当前工作目录下的临时文件 `payload_temp.json`，再通过 Shell 工具调用 `curl.exe --data-binary @payload_temp.json` 发送请求**。这是因为 PowerShell 使用 GBK 编码，而服务端使用 UTF-8 解析，直接在命令行中嵌入中文 JSON 会导致乱码。
  - 使用 `curl.exe`（而非 `curl`，后者在 PowerShell 中是 `Invoke-WebRequest` 的别名）。必须在 `curl.exe` 后加 `--%` 停止 PowerShell 解析，否则 `@` 会被误判为 splatting 操作符导致报错。
  - **文件创建方式**：根据可用工具选择其一（均需确保 UTF-8 **无 BOM** 编码，否则服务端 JSON 解析会在 position 0 报错）：
    - **方式 A（有 Write 工具）**：使用 Write 工具创建 `payload_temp.json`
    - **方式 B（无 Write 工具）**：使用 Shell 的 .NET API 创建文件（`Set-Content -Encoding UTF8` 会带 BOM，不可用）
- **macOS/Linux 环境**：
  - bash/zsh 默认使用 UTF-8 编码，可直接内联中文 JSON，无需临时文件。命令中使用 `curl`（无需 `.exe`，无需 `--%`）。
  - 推荐内联写法：`curl -X POST URL -H "..." -H "..." --data-binary 'JSON单行内容'`，一步完成。
  - 也可使用临时文件方式：`curl --data-binary @payload_temp.json`。
- **清理**：API 返回结果后，务必删除 `payload_temp.json` 临时文件（如使用了临时文件）。

### 示例 1：生成产品主图（跨境电商，Amazon）

#### 步骤 1：创建任务

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "query": "为这个蓝牙耳机生成一张白底产品主图",
  "generateType": 100,
  "posterType": 5,
  "platformType": "Amazon",
  "languageType": "英文",
  "detailPictureNumber": 1,
  "modelEdition": 3,
  "needText": true,
  "secretKey": "你的密钥"
}
```
> 方式 B（无 Write 工具）：
> ```powershell
> $json = '{"query":"为这个蓝牙耳机生成一张白底产品主图","generateType":100,"posterType":5,"platformType":"Amazon","languageType":"英文","detailPictureNumber":1,"modelEdition":3,"needText":true,"secretKey":"你的密钥"}'
> [System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))
> ```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateAsync" -H "Content-Type: application/json; charset=utf-8" --max-time 120 --data-binary @payload_temp.json
```

> **macOS/Linux 内联写法**（无需临时文件）：
> ```bash
> curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateAsync" -H "Content-Type: application/json; charset=utf-8" --max-time 120 --data-binary '{"query":"为这个蓝牙耳机生成一张白底产品主图","generateType":100,"posterType":5,"platformType":"Amazon","languageType":"英文","detailPictureNumber":1,"modelEdition":3,"needText":true,"secretKey":"你的密钥"}'
> ```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

#### 步骤 2：查询任务结果（轮询）

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "agentGenerateTaskId": "上一步返回的任务ID"
}
```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary @payload_temp.json
```

> **macOS/Linux 内联写法**：
> ```bash
> curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary '{"agentGenerateTaskId":"上一步返回的任务ID"}'
> ```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

#### 步骤 3：处理结果

轮询查询直到 `taskStatus=2`（生成成功），提取 `executeResult` 中的图片 URL 展示给用户。

### 示例 2：生成产品详情图（带参考图）

#### 步骤 1：创建任务

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "query": "根据上传的图片生成对应的产品图",
  "generateType": 200,
  "posterType": 5,
  "platformType": "Amazon",
  "languageType": "英文",
  "detailPictureNumber": 5,
  "modelEdition": 3,
  "needText": true,
  "secretKey": "你的密钥",
  "fileUrlList": ["https://example.com/product1.png", "https://example.com/product2.png"],
  "aspectRatio": "1:1"
}
```
> 方式 B（无 Write 工具）：
> ```powershell
> $json = '{"query":"根据上传的图片生成对应的产品图","generateType":200,"posterType":5,"platformType":"Amazon","languageType":"英文","detailPictureNumber":5,"modelEdition":3,"needText":true,"secretKey":"你的密钥","fileUrlList":["https://example.com/product1.png","https://example.com/product2.png"],"aspectRatio":"1:1"}'
> [System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))
> ```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateAsync" -H "Content-Type: application/json; charset=utf-8" --max-time 120 --data-binary @payload_temp.json
```

> **macOS/Linux 内联写法**（无需临时文件）：
> ```bash
> curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateAsync" -H "Content-Type: application/json; charset=utf-8" --max-time 120 --data-binary '{"query":"根据上传的图片生成对应的产品图","generateType":200,"posterType":5,"platformType":"Amazon","languageType":"英文","detailPictureNumber":5,"modelEdition":3,"needText":true,"secretKey":"你的密钥","fileUrlList":["https://example.com/product1.png","https://example.com/product2.png"],"aspectRatio":"1:1"}'
> ```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

#### 步骤 2：查询任务结果（轮询）

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "agentGenerateTaskId": "上一步返回的任务ID"
}
```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary @payload_temp.json
```

> **macOS/Linux 内联写法**：
> ```bash
> curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary '{"agentGenerateTaskId":"上一步返回的任务ID"}'
> ```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

### 示例 3：中文电商主图（淘宝，中文简体）

#### 步骤 1：创建任务

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "query": "为这款智能手表生成一张电商主图，突出科技感",
  "generateType": 100,
  "posterType": 6,
  "platformType": "淘宝",
  "languageType": "中文简体",
  "detailPictureNumber": 1,
  "modelEdition": 3,
  "needText": true,
  "secretKey": "你的密钥",
  "aspectRatio": "1:1"
}
```
> 方式 B（无 Write 工具）：
> ```powershell
> $json = '{"query":"为这款智能手表生成一张电商主图，突出科技感","generateType":100,"posterType":6,"platformType":"淘宝","languageType":"中文简体","detailPictureNumber":1,"modelEdition":3,"needText":true,"secretKey":"你的密钥","aspectRatio":"1:1"}'
> [System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))
> ```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateAsync" -H "Content-Type: application/json; charset=utf-8" --max-time 120 --data-binary @payload_temp.json
```

> **macOS/Linux 内联写法**（无需临时文件）：
> ```bash
> curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateAsync" -H "Content-Type: application/json; charset=utf-8" --max-time 120 --data-binary '{"query":"为这款智能手表生成一张电商主图，突出科技感","generateType":100,"posterType":6,"platformType":"淘宝","languageType":"中文简体","detailPictureNumber":1,"modelEdition":3,"needText":true,"secretKey":"你的密钥","aspectRatio":"1:1"}'
> ```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

#### 步骤 2：查询任务结果（轮询）

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "agentGenerateTaskId": "上一步返回的任务ID"
}
```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary @payload_temp.json
```

> **macOS/Linux 内联写法**：
> ```bash
> curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary '{"agentGenerateTaskId":"上一步返回的任务ID"}'
> ```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

## 常见错误及解决方案

| 错误 | 原因与解决 |
|------|-----------|
| HTTP 405 Not Allowed | URL 路径错误，确保包含 `/prod-api` 前缀 |
| code 500 "当前并发请求过多" | 服务繁忙，稍后重试 |
| HTTP 401 | 密钥无效或已过期，在 Flyelep 平台重新生成 |
| 查询结果 taskStatus 一直为 1 | 生图任务仍在进行中，继续轮询或稍后重试 |
| 查询结果 taskStatus 为 3 | 生图失败，检查参数是否正确或稍后重试 |
| 描述超过1000字符 | 缩短 query 内容 |

## 提示词处理

**基于参考图生成：** 将用户的产品描述传入 `query`，通过 `fileUrlList` 附上参考图片 URL。仅在描述明显不足时才优化。
**无参考图生成：** 在 `query` 中描述产品、风格和构图。
保留用户的创意意图。当用户描述模糊时，根据上下文（平台、语言等）推断合理的默认值。