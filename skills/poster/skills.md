---
name: flyelep-poster-gen
description: >-
  通过 Flyelep API 生成电商产品主图和详情图海报。
  当用户要求生成产品图、电商海报、Amazon 商品图、详情页图片时使用此技能。
metadata: {"clawdbot":{"emoji":"🎨","requires":{"envVars":["FLYELEP_SECRET_KEY"],"anyBins":["python3"]},"os":["linux","darwin","win32"]}}
---

# Flyelep 电商海报生图

通过 Flyelep AI API 生成电商产品主图和详情图海报。

## 使用方法

使用绝对路径运行脚本（不要先 cd 到技能目录）：

**生成产品详情图（默认）：**

```bash
python3 ~/skills/poster/scripts/generate_poster.py \
  --query "根据上传的图片生成对应的产品图" \
  --file-urls "https://example.com/product.png"
```

**生成产品主图：**

```bash
python3 ~/skills/poster/scripts/generate_poster.py \
  --query "为这个蓝牙耳机生成一张白底产品主图" \
  --generate-type 100 \
  --detail-number 1 \
  --file-urls "https://example.com/earphone.png"
```

**使用测试环境：**

```bash
python3 ~/skills/poster/scripts/generate_poster.py \
  --query "生成产品主图" \
  --base-url "https://testclient.flyelep.cn"
```

## 认证方式

脚本按以下优先级获取凭证：

1. `--secret-key` 命令行参数（用户在对话中提供时使用）
2. `FLYELEP_SECRET_KEY` 环境变量

用户需在 Flyelep 平台获取 `secretKey`。

## 参数说明

### 必传参数

| 参数 | 命令行标志 | 默认值 | 说明 |
|------|-----------|--------|------|
| query | `--query` | - | 生图需求描述，最多1000个字符 |
| generateType | `--generate-type` | 200 | 100=产品单图，200=产品详情图 |
| posterType | `--poster-type` | 5 | 5=跨境电商，6=中文电商 |
| platformType | `--platform-type` | amazon | 电商平台（见下方映射表） |
| languageType | `--language-type` | 英文 | 生成图片上的文案语种 |
| detailPictureNumber | `--detail-number` | 10 | 产品单图限1张；详情图可选5、10、15张 |
| modelEdition | `--model-edition` | 3 | 2=Flyelep 2.0，3=Flyelep 3.0 |
| needText | `--need-text` | true | 图片上是否包含文案 |
| secretKey | `--secret-key` | 环境变量 | API 密钥 |

### 可选参数

| 参数 | 命令行标志 | 默认值 | 说明 |
|------|-----------|--------|------|
| fileUrlList | `--file-urls` | - | 参考图片URL，逗号分隔，最多6张 |
| aspectRatio | `--aspect-ratio` | 随机 | 图片比例：1:1、3:2、2:3、3:4、4:3、4:5、5:4、9:16、16:9、21:9 |
| baseUrl | `--base-url` | https://www.flyelep.cn | API 基础地址（或设置 FLYELEP_BASE_URL 环境变量），测试环境：https://testclient.flyelep.cn |

## 参数映射规则

### platformType（电商平台）

根据用户的目标市场选择：

- 跨境电商：`amazon`、`Temu`、`Shopee`、`TikTok Shop`、`AliExpress`、`OZON`
- 中文电商：`淘宝`、`京东`、`拼多多`、`1688`、`小红书`

### languageType（文案语种）

根据用户的语言偏好选择：

- 跨境：英文、俄语、日语、韩语、阿拉伯语、德语、西班牙语、法语、泰语、马来语、越南语、葡萄牙语、菲律宾语、印尼语、意大利语、荷兰语、波兰语、中文繁体
- 中文：中文简体

### posterType + modelEdition（海报类型 + 模型版本）

先根据市场区域选择 posterType，再选择模型版本：

- 跨境电商（`posterType=5`）：Flyelep 2.0（`modelEdition=2`）或 3.0（`modelEdition=3`）
- 中文电商（`posterType=6`）：Flyelep 2.0（`modelEdition=2`）或 3.0（`modelEdition=3`）

### aspectRatio（图片比例）

根据用户的版式需求选择：

- 正方形：`1:1`
- 横版：`3:2`、`4:3`、`16:9`、`21:9`
- 竖版：`2:3`、`3:4`、`4:5`、`5:4`、`9:16`
- 未提及比例 → 不传此参数（API 自动随机选择比例）

### detailPictureNumber（生成图片数量）

- 产品单图（`generateType=100`）：固定为 `1`
- 产品详情图（`generateType=200`）：`5`、`10` 或 `15`

## 环境检查与常见错误

运行前检查：

- `command -v python3`（必须存在）
- `test -n "$FLYELEP_SECRET_KEY"`（或传 `--secret-key`）

常见错误及解决方案：

- `错误：未提供密钥` → 设置 `FLYELEP_SECRET_KEY` 环境变量或传 `--secret-key`
- `错误：HTTP 401` → 密钥无效或已过期，请在 Flyelep 平台重新生成
- `错误：请求超时` → 生图可能需要较长时间（最多5分钟），可增加 `--timeout` 值
- `API 返回成功但未生成图片` → 生图任务在排队中，可稍后重试或使用异步接口
- `错误：API 返回 500` → 服务端异常，请稍后重试
- `错误：描述超过1000字符` → 请缩短 query 内容

## 输出说明

- 脚本将生成的图片 URL 输出到 stdout（每行一个）
- 状态信息输出到 stderr
- API 原始响应中 URL 以分号分隔，脚本会自动拆分
- 不要回读图片内容，直接告知用户 URL 即可

## 提示词处理

**基于参考图生成：** 将用户的产品描述直接传给 `--query`，通过 `--file-urls` 附上参考图片。仅在描述明显不足时才进行优化。

**无参考图生成：** 在 `--query` 中描述产品、风格和构图。

保留用户的创意意图。当用户描述模糊时，根据上下文（平台、语言等）推断合理的默认值。

## 使用示例

**跨境详情图 + 参考图（Amazon，英文，Flyelep 3.0）：**

```bash
python3 ~/skills/poster/scripts/generate_poster.py \
  --query "根据上传的图片生成对应的产品图" \
  --generate-type 200 \
  --poster-type 5 \
  --platform-type amazon \
  --language-type 英文 \
  --detail-number 5 \
  --model-edition 3 \
  --aspect-ratio "1:1" \
  --need-text true \
  --file-urls "https://example.com/product1.png,https://example.com/product2.png"
```

**中文电商主图（淘宝，中文简体）：**

```bash
python3 ~/skills/poster/scripts/generate_poster.py \
  --query "为这款智能手表生成一张电商主图，突出科技感" \
  --generate-type 100 \
  --poster-type 6 \
  --platform-type "淘宝" \
  --language-type "中文简体" \
  --detail-number 1 \
  --model-edition 3 \
  --aspect-ratio "1:1"
```

**使用测试环境快速生成：**

```bash
python3 ~/skills/poster/scripts/generate_poster.py \
  --query "为一款无线鼠标生成 Amazon 产品详情图" \
  --base-url "https://testclient.flyelep.cn"
```
