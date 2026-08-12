---
name: generate-poster
description: >-
  通过 Flyelep API 生成电商产品主图、详情图海报和白底主图，支持异步（提交后轮询）与同步（一次请求出图）两种模式。
  当用户要求生成产品图、电商海报、Amazon 商品图、详情页图片、白底图、白底主图、纯白背景产品图时使用此技能。
---
# Flyelep 电商海报生图
通过 Flyelep AI API 生成电商产品主图、详情图海报和白底主图，支持异步与同步两种模式。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**

## 两种调用模式

| 模式 | 入口 | 返回 | 适用场景 |
|------|------|------|----------|
| 异步（推荐） | `/generateAsync` | 立即返回任务 ID，再轮询 `/queryTaskResult` | 绝大多数场景，尤其是多张详情图 |
| 同步 | `/generate`、`/whiteBgMainImgGen` | 一次请求内挂住连接直到出图，直接返回图片结果 | 只出 1 张图、且调用方不方便实现轮询时 |

两种模式的请求参数完全一致，共用同一套排队与计费逻辑，差别只在于结果怎么拿。

**默认用异步模式**。同步模式的连接要挂最长 15 分钟，容易被中间的网关、代理或客户端超时切断，而且它**不返回任务 ID**，一旦连接断开就无法再查回这次生成的结果。只有用户明确要求"一次请求拿到图"时才用同步。

## API 接口信息

### 创建任务（异步，推荐）

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateAsync`
- **Content-Type**: `application/json`
- **认证方式**: `secretKey` 可放在请求 Header 或 Body 中，两者都传时以 **Header 为准**；所有 `generateType` 的规则一致
- **超时时间**: 建议 120 秒（创建任务为瞬时操作）

请求头示例：

```http
Content-Type: application/json
secretKey: 用户提供的API密钥
```

查询结果接口同样支持 Header 传 `secretKey`，推荐统一放在 Header 里。

> **安全说明**：API 密钥仅在请求时传递给 Flyelep 服务器，不存储在技能代码中。请勿将真实的密钥直接写在示例代码中，运行时由用户动态提供。

### 查询结果（仅异步模式需要）

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult`
- **Content-Type**: `application/json`
- **认证方式**: 在请求 Header 中传入 `secretKey`
- **说明**: 异步接口，需轮询获取最终图片 URL（生图耗时较长，可能需要数分钟）

### 创建任务（同步）

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generate`
- **Content-Type**: `application/json`
- **认证方式**: 与异步模式一致，`secretKey` 放 Header 或 Body 均可，Header 优先
- **超时时间**: 服务端最长挂 **15 分钟**（900 秒）后返回超时失败，客户端 `--max-time` 要设到 900 秒以上，否则会自己先断开
- **说明**: 请求会一直挂住直到生成完成，响应里直接带图片结果，**不返回任务 ID**

白底主图另有一个语义化的同步入口：

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/whiteBgMainImgGen`
- **说明**: 等价于 `/generate` + `generateType=101`，服务端会强制把 `generateType` 置为 `101`，因此 body 里传不传 `generateType` 都一样。其余参数、鉴权、超时与 `/generate` 完全相同

## 请求 Body

以下 Body 结构对异步（`/generateAsync`）和同步（`/generate`、`/whiteBgMainImgGen`）都适用。

### generateType=100/200（产品单图/详情图）

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

### generateType=101（白底主图）

> **注意**：白底主图不需要 `platformType`、`languageType`、`needText` 参数。

```json
{
  "query": "生成白底主图的需求描述，最多1000个字符",
  "generateType": 101,
  "posterType": 5,
  "detailPictureNumber": 1,
  "modelEdition": 3,
  "secretKey": "用户提供的API密钥",
  "fileUrlList": ["https://example.com/product.png"],
  "aspectRatio": "1:1"
}
```

### 查询结果

```json
{
  "agentGenerateTaskId": "创建任务返回的任务ID"
}
```

## 响应格式

### 创建任务响应（异步模式）

```json
{
  "code": 200,
  "data": "20374084144913*****"
}
```

`data` 为异步任务 ID，用于后续轮询获取结果。

### 创建任务响应（同步模式）

```json
{
  "code": 200,
  "data": "https://xxx.com/image1.png;https://xxx.com/image2.png;"
}
```

- 同步模式的 `data` 是**一个字符串**，不是任务 ID，也不是数组
- 服务端把工作流逐段输出的内容用英文分号 `;` 拼接后返回，末尾可能带一个多余的 `;`
- 拿到后按 `;` 拆分、丢掉空片段，再从片段中取出图片 URL 展示给用户
- 内容由工作流决定，可能夹带非 URL 的文本片段，因此要按"是否以 `http` 开头"过滤，不要假设每一段都是图片
- 失败时 `code` 非 200，原因在 `msg` 里，例如 `当前并发请求过多，请稍后重试`、`任务执行超时，请稍后重试`、`任务执行失败: xxx`

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
| generateType | 200 | 100=产品单图，101=白底主图，200=产品详情图 |
| posterType | 5 | 5=跨境电商，6=中文电商 |
| platformType | amazon | 电商平台（见下方映射表）。**generateType=101 时不需要** |
| languageType | 英文 | 生成图片上的文案语种。**generateType=101 时不需要** |
| detailPictureNumber | 10 | 产品单图限1张；详情图可选5、10、15张 |
| modelEdition | 3 | 模型类型：2=Flyelep 2.0，3=Flyelep 3.0（默认），9=Flyelep Image 2 |
| needText | true | 图片上是否包含文案。**generateType=101 时不需要** |
| secretKey | - | API 密钥，放在 Header 或 Body 中均可，Header 优先 |

### 可选参数

| 字段 | 默认值 | 说明 |
|------|--------|------|
| fileUrlList | - | 参考图片 URL 数组，最多6张 |
| aspectRatio | 随机 | 图片比例：1:1、3:2、2:3、3:4、4:3、4:5、5:4、9:16、16:9、21:9 |
| channel | `premium` | 尊享通道（`premium`，默认）/特惠通道（`promotion`）。特惠通道仅支持 `modelEdition` 为 `3` 或 `9` |

### 参数映射规则

#### platformType（电商平台）
- 跨境电商：`amazon`、`Temu`、`Shopee`、`TikTok Shop`、`AliExpress`、`OZON`
- 中文电商：`淘宝`、`京东`、`拼多多`、`1688`、`小红书`

#### languageType（文案语种）
- 跨境：英文、俄语、日语、韩语、阿拉伯语、德语、西班牙语、法语、泰语、马来语、越南语、葡萄牙语、菲律宾语、印尼语、意大利语、荷兰语、波兰语、中文繁体
- 中文：中文简体

#### posterType（海报类型）
根据市场区域选择，再搭配 modelEdition：
- 跨境电商 → `posterType=5`
- 中文电商 → `posterType=6`

#### aspectRatio（图片比例）
- 正方形：`1:1`
- 横版：`3:2`、`4:3`、`16:9`、`21:9`
- 竖版：`2:3`、`3:4`、`4:5`、`5:4`、`9:16`
- 未提及比例 → 不传此字段（API 自动随机选择）

#### detailPictureNumber（图片数量）
- 产品单图（`generateType=100`）：固定为 `1`
- 白底主图（`generateType=101`）：白底单图固定为 `1`，白底详情图可选 `5`、`10` 或 `15`
- 产品详情图（`generateType=200`）：`5`、`10` 或 `15`

#### fileUrlList（参考图片）
- 数组格式，最多6张
- 每个链接都应为公网可访问的图片直链
- 如果用户提供本地文件路径，先调用 file-upload 技能上传文件获取公网链接，再填入此参数

#### modelEdition（模型类型）
- `2`：Flyelep 2.0（`posterType=5` 时为 gemini-2.5，`posterType=6` 时为 doubao-seedream）
- `3`：Flyelep 3.0（gemini-3.1），不传时的默认值
- `9`：Flyelep Image 2
- 用户未指定模型时不传此字段即可，服务端按 `3` 处理

#### channel（通道）
- `premium`：尊享通道，不传时的默认值，支持全部模型
- `promotion`：特惠通道，**仅支持 `modelEdition` 为 `3`（Flyelep 3.0）或 `9`（Flyelep Image 2）**，与其它模型组合会被接口拒绝
- 用户未指定通道时不传此字段

## 异步任务流程

> **重要**：走 `generateAsync` 时必须**先调用主接口获取 `agentGenerateTaskId`，然后调用 `queryTaskResult` 接口轮询任务结果**，不能省略轮询步骤。

1. 调用主接口（`generateAsync`）提交任务，从响应中获取 `agentGenerateTaskId`
2. 使用 `agentGenerateTaskId` 调用 `queryTaskResult` 接口轮询任务结果（建议每 5-10 秒查询一次）
3. 当 `taskStatus=2` 时，表示生成成功，获取 `executeResult` 结果
4. 当 `taskStatus=3` 时，表示生成失败

## 同步任务流程

1. 调用 `/generate`（白底主图可用 `/whiteBgMainImgGen`），Body 与异步模式完全一致
2. 把客户端超时设到 900 秒以上，请求会一直挂住等出图，中途没有任何进度反馈
3. 响应返回后先看 `code`：非 200 时按 `msg` 处理，不要把 `msg` 当成图片结果
4. `code=200` 时把 `data` 按英文分号 `;` 拆分，过滤掉空片段和非 `http` 开头的片段，剩下的就是图片 URL
5. 逐个展示图片 URL 给用户

> **连接断开就拿不回结果**：同步接口不返回任务 ID，客户端超时、网关断连或进程退出后，这次生成的结果无法再查询（算力照常扣除）。需要可靠拿到结果时改用异步模式。

## 调用示例

> **跨平台调用说明**：
> - 请求头必须包含 `Content-Type: application/json; charset=utf-8` 和 `secretKey`
> - **Windows/PowerShell**：因 GBK 编码问题，必须先将 JSON 写入临时文件 `payload_temp.json`（UTF-8 无 BOM），再用 `curl.exe --% --data-binary @payload_temp.json` 发送请求。使用 Write 工具创建文件，或用 .NET API `[System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))`。调用后用 `rm payload_temp.json` 清理。
> - **macOS/Linux**：bash/zsh 默认 UTF-8，可直接内联 JSON：`curl -X POST URL -H "Content-Type: application/json; charset=utf-8" --data-binary 'JSON单行内容'`（按需追加 `-H "secretKey: 你的密钥"`）

### 示例 1：生成产品主图（跨境电商，Amazon）

#### 步骤 1：创建任务

**Windows/PowerShell**：

创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "query": "为这个蓝牙耳机生成一张产品主图",
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

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"query":"为这个蓝牙耳机生成一张白底产品主图","generateType":100,"posterType":5,"platformType":"Amazon","languageType":"英文","detailPictureNumber":1,"modelEdition":3,"needText":true,"secretKey":"你的密钥"}', [System.Text.UTF8Encoding]::new($false))
```

执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateAsync" -H "Content-Type: application/json; charset=utf-8" --max-time 120 --data-binary @payload_temp.json
```

清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateAsync" -H "Content-Type: application/json; charset=utf-8" --max-time 120 --data-binary '{"query":"为这个蓝牙耳机生成一张白底产品主图","generateType":100,"posterType":5,"platformType":"Amazon","languageType":"英文","detailPictureNumber":1,"modelEdition":3,"needText":true,"secretKey":"你的密钥"}'
```

#### 步骤 2：查询任务结果

**Windows/PowerShell**：

创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "agentGenerateTaskId": "上一步返回的任务ID"
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"agentGenerateTaskId":"上一步返回的任务ID"}', [System.Text.UTF8Encoding]::new($false))
```

执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary @payload_temp.json
```

清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary '{"agentGenerateTaskId":"上一步返回的任务ID"}'
```

#### 步骤 3：处理结果

轮询查询直到 `taskStatus=2`（生成成功），提取 `executeResult` 中的图片 URL 展示给用户。

### 示例 2：生成产品详情图（带参考图）

**前置步骤**：向用户索取图片路径或 URL。如用户提供本地文件，先调用 file-upload 技能上传获取公网链接。

#### 步骤 1：创建任务

**Windows/PowerShell**：

创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
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

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"query":"根据上传的图片生成对应的产品图","generateType":200,"posterType":5,"platformType":"Amazon","languageType":"英文","detailPictureNumber":5,"modelEdition":3,"needText":true,"secretKey":"你的密钥","fileUrlList":["https://example.com/product1.png","https://example.com/product2.png"],"aspectRatio":"1:1"}', [System.Text.UTF8Encoding]::new($false))
```

执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateAsync" -H "Content-Type: application/json; charset=utf-8" --max-time 120 --data-binary @payload_temp.json
```

清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateAsync" -H "Content-Type: application/json; charset=utf-8" --max-time 120 --data-binary '{"query":"根据上传的图片生成对应的产品图","generateType":200,"posterType":5,"platformType":"Amazon","languageType":"英文","detailPictureNumber":5,"modelEdition":3,"needText":true,"secretKey":"你的密钥","fileUrlList":["https://example.com/product1.png","https://example.com/product2.png"],"aspectRatio":"1:1"}'
```

#### 步骤 2：查询任务结果

**Windows/PowerShell**：

创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "agentGenerateTaskId": "上一步返回的任务ID"
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"agentGenerateTaskId":"上一步返回的任务ID"}', [System.Text.UTF8Encoding]::new($false))
```

执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary @payload_temp.json
```

清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary '{"agentGenerateTaskId":"上一步返回的任务ID"}'
```

#### 步骤 3：处理结果

轮询查询直到 `taskStatus=2`（生成成功），提取 `executeResult` 中的图片 URL 展示给用户。

### 示例 3：中文电商主图（淘宝，中文简体）

#### 步骤 1：创建任务

**Windows/PowerShell**：

创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
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

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"query":"为这款智能手表生成一张电商主图，突出科技感","generateType":100,"posterType":6,"platformType":"淘宝","languageType":"中文简体","detailPictureNumber":1,"modelEdition":3,"needText":true,"secretKey":"你的密钥","aspectRatio":"1:1"}', [System.Text.UTF8Encoding]::new($false))
```

执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateAsync" -H "Content-Type: application/json; charset=utf-8" --max-time 120 --data-binary @payload_temp.json
```

清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateAsync" -H "Content-Type: application/json; charset=utf-8" --max-time 120 --data-binary '{"query":"为这款智能手表生成一张电商主图，突出科技感","generateType":100,"posterType":6,"platformType":"淘宝","languageType":"中文简体","detailPictureNumber":1,"modelEdition":3,"needText":true,"secretKey":"你的密钥","aspectRatio":"1:1"}'
```

#### 步骤 2：查询任务结果

**Windows/PowerShell**：

创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "agentGenerateTaskId": "上一步返回的任务ID"
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"agentGenerateTaskId":"上一步返回的任务ID"}', [System.Text.UTF8Encoding]::new($false))
```

执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary @payload_temp.json
```

清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary '{"agentGenerateTaskId":"上一步返回的任务ID"}'
```

#### 步骤 3：处理结果

轮询查询直到 `taskStatus=2`（生成成功），提取 `executeResult` 中的图片 URL 展示给用户。

### 示例 4：生成白底主图（generateType=101）

> **注意**：白底主图不需要 `platformType`、`languageType`、`needText` 参数。

**前置步骤**：向用户索取图片路径或 URL。如用户提供本地文件，先调用 file-upload 技能上传获取公网链接。

#### 步骤 1：创建任务

**Windows/PowerShell**：

创建 `payload_temp.json`（无 platformType/languageType/needText，两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "query": "生成一张蓝牙耳机白底产品主图，产品居中，背景纯白",
  "generateType": 101,
  "posterType": 5,
  "detailPictureNumber": 1,
  "modelEdition": 3,
  "secretKey": "你的密钥",
  "aspectRatio": "1:1"
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"query":"生成一张蓝牙耳机白底产品主图，产品居中，背景纯白","generateType":101,"posterType":5,"detailPictureNumber":1,"modelEdition":3,"secretKey":"你的密钥","aspectRatio":"1:1"}', [System.Text.UTF8Encoding]::new($false))
```

执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateAsync" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary @payload_temp.json
```

清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generateAsync" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 120 --data-binary '{"query":"生成一张蓝牙耳机白底产品主图，产品居中，背景纯白","generateType":101,"posterType":5,"detailPictureNumber":1,"modelEdition":3,"secretKey":"你的密钥","aspectRatio":"1:1"}'
```

#### 步骤 2：查询任务结果

**Windows/PowerShell**：

创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "agentGenerateTaskId": "上一步返回的任务ID"
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"agentGenerateTaskId":"上一步返回的任务ID"}', [System.Text.UTF8Encoding]::new($false))
```

执行请求（secretKey 在 Header 中）：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary @payload_temp.json
```

清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 30 --data-binary '{"agentGenerateTaskId":"上一步返回的任务ID"}'
```

#### 步骤 3：处理结果

轮询查询直到 `taskStatus=2`（生成成功），提取 `executeResult` 中的图片 URL 展示给用户。

### 示例 5：同步模式生成产品主图（一次请求拿到图）

> **注意**：同步模式一个请求就返回图片，不需要查询步骤，但必须把超时设到 900 秒以上。

**Windows/PowerShell**：

创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "query": "为这个蓝牙耳机生成一张产品主图",
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

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"query":"为这个蓝牙耳机生成一张产品主图","generateType":100,"posterType":5,"platformType":"Amazon","languageType":"英文","detailPictureNumber":1,"modelEdition":3,"needText":true,"secretKey":"你的密钥"}', [System.Text.UTF8Encoding]::new($false))
```

执行请求（`--max-time 960` 留出比服务端 900 秒更长的余量）：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generate" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 960 --data-binary @payload_temp.json
```

清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generate" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 960 --data-binary '{"query":"为这个蓝牙耳机生成一张产品主图","generateType":100,"posterType":5,"platformType":"Amazon","languageType":"英文","detailPictureNumber":1,"modelEdition":3,"needText":true,"secretKey":"你的密钥"}'
```

处理结果：把 `data` 按 `;` 拆分，取出以 `http` 开头的片段作为图片 URL 展示给用户。

### 示例 6：同步模式生成白底主图（whiteBgMainImgGen）

> **注意**：该入口会强制 `generateType=101`，Body 中不必再传；同样不需要 `platformType`、`languageType`、`needText`。

**Windows/PowerShell**：

创建 `payload_temp.json`（两种方式任选其一）：

方式 A（使用 Write 工具）：
```json
{
  "query": "生成一张蓝牙耳机白底产品主图，产品居中，背景纯白",
  "posterType": 5,
  "detailPictureNumber": 1,
  "modelEdition": 3,
  "secretKey": "你的密钥",
  "aspectRatio": "1:1"
}
```

方式 B（无 Write 工具，PowerShell 执行）：
```powershell
[System.IO.File]::WriteAllText("payload_temp.json", '{"query":"生成一张蓝牙耳机白底产品主图，产品居中，背景纯白","posterType":5,"detailPictureNumber":1,"modelEdition":3,"secretKey":"你的密钥","aspectRatio":"1:1"}', [System.Text.UTF8Encoding]::new($false))
```

执行请求：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/whiteBgMainImgGen" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 960 --data-binary @payload_temp.json
```

清理临时文件：
```bash
rm payload_temp.json
```

**macOS/Linux**：
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/whiteBgMainImgGen" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 960 --data-binary '{"query":"生成一张蓝牙耳机白底产品主图，产品居中，背景纯白","posterType":5,"detailPictureNumber":1,"modelEdition":3,"secretKey":"你的密钥","aspectRatio":"1:1"}'
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
| 同步模式 `msg` 为 `任务执行超时，请稍后重试` | 生成超过服务端 15 分钟上限，改用异步模式或减少 `detailPictureNumber` |
| 同步模式 `msg` 为 `任务执行失败: xxx` | 工作流执行报错，按 `xxx` 排查参数或稍后重试 |
| 同步模式客户端先超时断开 | `--max-time` 小于服务端 900 秒上限，调大超时；结果无法找回，需重新生成 |
| 同步模式 `data` 拆分后没有图片 URL | 工作流本次未输出图片，改用异步模式提交并用 `queryTaskResult` 确认任务状态 |
| `生图类型不能为空` | `/generate` 与 `/generateAsync` 必须传 `generateType`；用 `/whiteBgMainImgGen` 时可不传 |

## 执行流程

1. **向用户询问 `secretKey`**（API 密钥必须由用户提供，agent 不可自行填写）
2. 收集用户的生成需求并写入 `query`
3. 确定目标平台 `platformType`、语言 `languageType` 和海报类型 `posterType`
4. 选择 `generateType`：100=产品单图，101=白底主图，200=产品详情图
5. 设置参数：`detailPictureNumber`、`modelEdition`、`needText`、`aspectRatio` 等
6. 如用户提供参考图，收集图片 URL 写入 `fileUrlList`（本地文件先调用 file-upload 技能上传）
7. 在请求头中传入 `secretKey`
8. 选择调用模式：默认异步；用户明确要求"一次请求出图"时才用同步
9. 异步模式：调用 `/generateAsync` 读取任务 ID → 轮询 `/queryTaskResult` 直到 `taskStatus=2` → 取 `executeResult`
10. 同步模式：调用 `/generate`（白底主图可用 `/whiteBgMainImgGen`），超时设 900 秒以上 → 把 `data` 按 `;` 拆分取出图片 URL
11. 将生成的图片逐个展示给用户

**提示词处理：**
- 基于参考图生成：将用户的产品描述传入 `query`，通过 `fileUrlList` 附上参考图片 URL。仅在描述明显不足时才优化
- 无参考图生成：在 `query` 中描述产品、风格和构图
- 保留用户的创意意图。当用户描述模糊时，根据上下文（平台、语言等）推断合理的默认值
