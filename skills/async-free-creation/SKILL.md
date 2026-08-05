---
name: async-free-creation
description: >-
  通过 Flyelep 异步自由创作接口调用 Image-2 模型生成产品图或创意图。
  当用户要求异步自由创作、自由创作、Image-2 自由创作、根据提示词和参考图生成多张图片时使用此技能。
---
# Flyelep 异步自由创作

通过 Flyelep Image-2 自由创作 API 异步生成图片。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**

## API 接口信息

### 创建任务

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/allAroundCreationAsync`
- **Content-Type**: `application/json`
- **认证方式**: 在请求头中传入 `secretKey`
- **超时时间**: 建议 120-300 秒

### 查询结果

- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult`
- **Content-Type**: `application/json`
- **认证方式**: 在请求头中传入 `secretKey`
- **说明**: 该接口是新版任务查询接口，优先使用；旧接口 `queryResult` 仍可作为兼容备选。

## 认证方式

在请求头中传入 `secretKey`。该密钥需由用户在 Flyelep 开放平台申请获得：https://www.flyelep.cn/controlboard 。

请求头示例：

```http
Content-Type: application/json
secretKey: 用户提供的API密钥
```

> **安全说明**：`secretKey` 必须放在请求头中，不要将真实密钥写入技能文件、示例代码仓库或持久化配置中，应在运行时由用户动态提供。

## 创建任务请求 Body

```json
{
  "query": "生成钢笔的产品图",
  "apiImgUrlList": [
    "https://example.com/product.png"
  ],
  "detailPictureNumber": 4,
  "aspectRatio": "1:1"
}
```

## 创建任务响应格式

成功：

```json
{
  "code": 200,
  "data": {
    "agentGenerateTaskId": "2054467932287070209"
  }
}
```

- `code=200` 表示创建任务成功
- `data.agentGenerateTaskId` 为异步任务 ID
- 创建任务后必须继续调用查询结果接口获取最终图片 URL

## 查询结果请求 Body

```json
{
  "agentGenerateTaskId": "2054467932287070209"
}
```

## 查询结果响应格式

```json
{
  "code": 200,
  "data": {
    "taskList": [
      {
        "taskStatus": 2,
        "executeResult": "https://example.com/result1.png"
      },
      {
        "taskStatus": 1
      }
    ]
  }
}
```

### taskStatus 状态

| 值 | 含义 |
|----|------|
| `0` | 待生成 |
| `1` | 生成中 |
| `2` | 生成成功 |
| `3` | 生成失败 |

当 `taskStatus=2` 时，读取对应项的 `executeResult` 图片 URL 并展示给用户。若仍有 `0` 或 `1`，等待后继续轮询；若出现 `3`，告知用户该任务项生成失败。

## 参数说明

### 必传参数

> **重要**：以下必传参数必须通过询问用户获取，agent 不可自行填写。调用本技能时，应先向用户列出必传参数与可选参数表格，由用户确认或提供后再执行。

| 字段 | 默认值 | 说明 |
|------|--------|------|
| query | - | 用户生成图片的具体需求描述，最多 1000 个字符 |
| detailPictureNumber | - | 需要生成的图片数量，支持 `1` 到 `4` |
| aspectRatio | 随机 | 图片比例；文档标注为必需，但为空时默认随机比例 |

### 可选参数

| 字段 | 默认值 | 说明 |
|------|--------|------|
| apiImgUrlList | - | 参考图片 URL 数组，最多 6 张，建议单张图片小于 10MB |

## 参数映射规则

### query

- 直接传入用户对图片的生成需求
- 保留用户原始创意意图，不要无故扩写成另一种产品或场景
- 如果用户只给产品名，可补充为简洁的产品图生成需求，例如：`生成钢笔的产品图`
- 最多 1000 个字符，超出时需要压缩描述

### apiImgUrlList

- 传入公网可访问的图片直链数组
- 最多 6 张
- 字段名是 `apiImgUrlList`，不是 `fileUrlList` 或 `imgUrlList`
- 用户未提供参考图时，不传此字段

### detailPictureNumber

- 支持 `1`、`2`、`3`、`4`
- 用户未指定数量时，默认传 `4`
- 用户要求超过 4 张时，告知该接口单次最多生成 4 张，可分多次调用

### aspectRatio

支持以下比例：

- `1:1`
- `3:2`
- `2:3`
- `3:4`
- `4:3`
- `4:5`
- `5:4`
- `9:16`
- `16:9`
- `21:9`

API 文档标注此参数为必需，但默认为空时接口随机选择比例。

默认规则：

- 用户明确指定比例时，原样传入
- 用户未指定比例时，可传空字符串或不传，让接口随机选择
- 横版优先推断为 `16:9`
- 方图优先推断为 `1:1`
- 竖版优先推断为 `9:16`

## 调用示例
- **重要**：在 Windows/PowerShell 环境下调用 API 时，必须采用以下流程：**先将请求体 JSON 写入当前工作目录下的临时文件 `payload_temp.json`，再通过 Shell 工具调用 `curl.exe --data-binary @payload_temp.json` 发送请求**。这是因为 PowerShell 使用 GBK 编码，而服务端使用 UTF-8 解析，直接在命令行中嵌入中文 JSON 会导致乱码。同时必须设置 `Content-Type: application/json; charset=utf-8` 请求头。
- **注意**：在 Windows 环境下使用 `curl.exe`（而非 `curl`，后者在 PowerShell 中是 `Invoke-WebRequest` 的别名）。必须在 `curl.exe` 后加 `--%` 停止 PowerShell 解析，否则 `@` 会被误判为 splatting 操作符导致报错。
- **文件创建方式**：根据可用工具选择其一（均需确保 UTF-8 **无 BOM** 编码，否则服务端 JSON 解析会在 position 0 报错）：
  - **方式 A（有 Write 工具）**：使用 Write 工具创建 `payload_temp.json`
  - **方式 B（无 Write 工具）**：使用 Shell 的 .NET API 创建文件（`Set-Content -Encoding UTF8` 会带 BOM，不可用）
- **清理**：API 返回结果后，务必删除 `payload_temp.json` 临时文件。

**示例 1：创建异步自由创作任务**

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "aspectRatio": "1:1",
  "query": "生成钢笔的产品图",
  "detailPictureNumber": 4,
  "apiImgUrlList": [
    "https://example.com/product.png"
  ]
}
```
> 方式 B（无 Write 工具）：
> ```powershell
> $json = '{"aspectRatio":"1:1","query":"生成钢笔的产品图","detailPictureNumber":4,"apiImgUrlList":["https://example.com/product.png"]}'
> [System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))
> ```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/allAroundCreationAsync" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

**示例 2：查询任务结果**

步骤 1：创建 `payload_temp.json`，内容如下：
```json
{
  "agentGenerateTaskId": "2054467932287070209"
}
```
> 方式 B（无 Write 工具）：
> ```powershell
> $json = '{"agentGenerateTaskId":"2054467932287070209"}'
> [System.IO.File]::WriteAllText("payload_temp.json", $json, [System.Text.UTF8Encoding]::new($false))
> ```

步骤 2：使用 Shell 工具执行：
```bash
curl.exe --% -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" -H "Content-Type: application/json; charset=utf-8" -H "secretKey: 你的密钥" --max-time 300 --data-binary @payload_temp.json
```

步骤 3：清理临时文件：
```bash
rm payload_temp.json
```

## 执行流程

1. 收集用户的生成需求并写入 `query`
2. 收集可选参考图并写入 `apiImgUrlList`
3. 确定 `detailPictureNumber`，未指定时默认 `4`
4. 确定 `aspectRatio`，未指定时可为空或不传
5. 在请求头中传入 `secretKey`
6. 调用 `allAroundCreationAsync` 创建任务
7. 从响应中读取 `data.agentGenerateTaskId`
8. 调用 `queryTaskResult` 轮询任务结果
9. 将 `taskStatus=2` 的 `executeResult` 图片 URL 逐个展示给用户

## 常见错误及解决方案

| 错误 | 原因与解决 |
|------|-----------|
| HTTP 401 / `code` 非 200 | `secretKey` 无效、缺失或已过期，确认请求头是否正确传入 |
| HTTP 405 Not Allowed | 请求方法错误，必须使用 `POST` |
| `agentGenerateTaskId` 为空 | 创建任务失败或响应结构异常，检查创建任务接口返回 |
| 查询结果一直是 `0` 或 `1` | 图片仍在排队或生成中，等待后继续轮询 |
| `taskStatus=3` | 对应图片生成失败，可简化提示词、减少参考图或重试 |
| `detailPictureNumber` 超出范围 | 该接口仅支持单次生成 `1-4` 张 |
| `apiImgUrlList` 无法访问 | 参考图 URL 不是公网直链、已过期，或源站限制访问 |
| 比例不支持 | `aspectRatio` 必须使用文档规定的比例枚举 |

## 提示词处理

该接口适合宽泛的创意图片生成，但仍应把提示词写得明确、短而有约束。

推荐提示词包含：

- 产品或主体
- 使用场景
- 风格与氛围
- 构图或比例要求
- 需要保留的参考图特征

不要在提示词中承诺接口不支持的细粒度编辑能力。若用户明确要求局部重绘、商品换色、商品替换、场景替换、图片翻译、抠图、放大或超清增强，应优先使用对应的专用 skill。
