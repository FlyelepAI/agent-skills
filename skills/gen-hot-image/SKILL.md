---
name: gen-hot-image
description: >-
  通过 Flyelep 爆款图片复刻 API，基于爆款图片风格生成产品复刻图。
  当用户要求复刻爆款图片、模仿参考图风格生成产品图时使用此技能。
---
# Flyelep 爆款图片复刻
通过 Flyelep AI Tool API 复刻爆款图片风格，将产品素材图融合到爆款参考图的视觉风格中。

**重要：这是一个 HTTP API 调用技能。必须通过 HTTP POST 请求调用 API 接口，禁止通过浏览器访问 Flyelep 网站。**
**注意：此接口为异步接口，只返回任务ID，需要通过 queryTaskResult 接口获取最终结果。**

## API 接口信息
- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/generateHotImage`
- **Content-Type**: `application/json`
- **超时时间**: 建议 60-120 秒（获取任务结果需额外轮询）

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
  "replaceUrl": "https://example.com/product1.png,https://example.com/product2.png",
  "sourceUrl": "https://example.com/hot_image1.png,https://example.com/hot_image2.png",
  "prompt": "突出产品卖点，增强视觉冲击力",
  "modelType": 9,
  "ratio": "1:1",
  "language": "中文简体"
}
```

## 响应格式
提交请求（异步）：
```json
{
  "code": 200,
  "data": {
    "agentGenerateTaskId": "2072922328536604674"
  }
}
```

查询任务结果：
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
        "taskStatus": 2,
        "executeResult": "https://example.com/result2.png"
      }
    ]
  }
}
```

- `code=200` 表示调用成功
- `agentGenerateTaskId` 为异步任务ID，用于后续查询结果
- `taskStatus`: 0-待生成，1-生成中，2-生成成功，3-生成失败
- `executeResult` 为生成的图片URL
- 将结果图片逐个展示给用户，不要回读图片内容

## 参数说明
### 必传参数
| 字段 | 默认值 | 说明 |
|------|--------|------|
| replaceUrl | - | 产品素材图地址，多张用英文逗号分隔，总大小在10MB以内 |
| sourceUrl | - | 爆款参考图地址，最多10张，多张用英文逗号分隔 |
| prompt | - | 提示词，最多1000个字符长度 |
| modelType | - | 模型类型：`2`=Flyelep Nano 2，`9`=Flyelep Image 2 |
| ratio | - | 生图比例 |
| language | - | 生成语言 |

## 参数映射规则
### modelType（模型类型）
- `2`：Flyelep Nano 2（轻量快速）
- `9`：Flyelep Image 2（高质量）

### ratio（图片比例）
- 支持：`1:1`、`3:2`、`2:3`、`3:4`、`4:3`、`4:5`、`5:4`、`16:9`、`9:16`、`21:9`

### language（生成语言）
- 中文简体、中文繁体、英文、俄语、日语、韩语、阿拉伯语、德语、西班牙语、法语、泰语、马来语、越南语、葡萄牙语、菲律宾语、印尼语、意大利语、荷兰语、波兰语、罗马尼亚语、匈牙利、保加利亚语

### replaceUrl（产品素材）
- 产品素材图地址，用于替换爆款图中的产品
- 多张图用英文逗号`,`分隔
- 总大小需在10MB以内

### sourceUrl（爆款参考）
- 爆款参考图片地址，用于提供风格参考
- 最多10张
- 多张图用英文逗号`,`分隔

## 异步任务查询
生成图片为异步流程，需要：
1. 调用 `generateHotImage` 提交任务，获取 `agentGenerateTaskId`
2. 轮询调用 `queryTaskResult` 查询任务状态
3. 当 `taskStatus=2` 时，表示生成成功，获取结果

### 查询任务结果接口
- **URL**: `POST https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult`
- **请求体**:
```json
{
  "agentGenerateTaskId": "任务ID"
}
```

- **轮询策略**：建议每5-10秒查询一次，超时时间不超过5分钟

## 调用示例
- 注意：调用 API 时必须设置 Content-Type 为 `application/json; charset=utf-8`，否则中文会乱码。
**提交爆款图片复刻任务：**
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/generateHotImage" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "secretKey: 你的密钥" \
  --max-time 120 \
  -d '{
    "replaceUrl": "https://example.com/product.png",
    "sourceUrl": "https://example.com/hot_image.png",
    "prompt": "突出产品卖点，增强视觉冲击力",
    "modelType": 9,
    "ratio": "1:1",
    "language": "中文简体"
  }'
```

**查询任务结果：**
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/queryTaskResult" \
  -H "Content-Type: application/json" \
  -H "secretKey: 你的密钥" \
  --max-time 30 \
  -d '{
    "agentGenerateTaskId": "2072922328536604674"
  }'
```

**多张产品素材复刻：**
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/aiTool/generateHotImage" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "secretKey: 你的密钥" \
  --max-time 120 \
  -d '{
    "replaceUrl": "https://example.com/product1.png,https://example.com/product2.png",
    "sourceUrl": "https://example.com/hot1.png,https://example.com/hot2.png",
    "prompt": "保持爆款图的配色和构图，将产品自然融入场景",
    "modelType": 2,
    "ratio": "3:4",
    "language": "英文"
  }'
```

## 常见错误及解决方案
| 错误 | 原因与解决 |
|------|-----------|
| HTTP 401 / `code` 非 200 | `secretKey` 无效、缺失或已过期，确认请求头是否正确传入 |
| HTTP 405 Not Allowed | 请求方法错误，必须使用 `POST` |
| replaceUrl/sourceUrl 格式错误 | 多个URL用英文逗号分隔，不是JSON数组 |
| 素材图超过10MB | 压缩产品素材图大小 |
| 参考图超过10张 | 减少sourceUrl中的图片数量 |
| prompt 超过1000字符 | 缩短提示词内容 |
| 服务繁忙（9999错误码） | 稍后重试 |
| taskStatus=3 生成失败 | 检查素材图质量，尝试更换图片或调整prompt |

## 提示词处理
复刻时，prompt 应指导AI：
- 保持爆款图的整体风格、配色和构图
- 将产品自然地融入到场景中
- 突出产品卖点和关键信息

**示例prompt：**
- "保持爆款图的配色和构图，将产品自然融入场景"
- "突出产品卖点，增强视觉冲击力"
- "延续原图的电商促销风格，产品主体突出"
- "保留原图的氛围感，让产品成为视觉焦点"
