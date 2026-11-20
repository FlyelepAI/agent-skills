---
name: flyelep-ai-skills
description: >-
  Flyelep AI Agent 技能集合，可通过仓库 URL 被 OpenClaw、Claude Code 等 AI 工具加载。
  当前仓库中的技能全部基于 Flyelep API 接口文档（https://api.flyelep.cn/）整理，覆盖电商海报生成、AI 图片工具、视频生成三大类能力。
---
## 可用技能

| 技能 | 描述 |
|------|------|
| [generate-poster](skills/generate-poster/SKILL.md) | 生成电商产品单图、白底图、主副图、套图、详情图、宣传图 |
| [async-free-creation](skills/async-free-creation/SKILL.md) | 异步自由创作，可选模型生成 1-20 张创意图片 |
| [intelligent-extension](skills/intelligent-extension/SKILL.md) | 智能延展图片，支持批量处理和目标比例适配 |
| [image-translate](skills/image-translate/SKILL.md) | 识别并翻译图片中的文字，支持多图、多语言 |
| [partial-redrawing](skills/partial-redrawing/SKILL.md) | 对图片局部区域进行重绘，可结合文本提示词和参考图 |
| [image-enlarge](skills/image-enlarge/SKILL.md) | 无损放大图片，支持单张或批量 |
| [ai-image-matting](skills/ai-image-matting/SKILL.md) | 自动去除图片背景，支持批量抠图 |
| [scene-replace](skills/scene-replace/SKILL.md) | 替换图片背景场景，可结合参考图和文本描述 |
| [product-replace](skills/product-replace/SKILL.md) | 替换图片中的商品主体，保留背景和光影效果 |
| [product-color-change](skills/product-color-change/SKILL.md) | 智能识别商品并进行换色处理 |
| [image-clarity-enhance](skills/image-clarity-enhance/SKILL.md) | AI 超清增强图片清晰度，支持批量处理 |
| [long-image-fusion](skills/long-image-fusion/SKILL.md) | 多图融合，把多张图拼接融合成一张长图 |
| [image-layer-decompose](skills/image-layer-decompose/SKILL.md) | 图层分离，把图片拆成底图和最多 16 个透明图层 |
| [main-image-tagging](skills/main-image-tagging/SKILL.md) | 主图打标，按参考图版式把卖点和价格批量打到商品图上 |
| [image-text-recognize](skills/image-text-recognize/SKILL.md) | 文字识别，返回图片中每行文字和坐标 |
| [text-edit](skills/text-edit/SKILL.md) | 文字编辑，按原文→新文替换图片上的文字并保持版式 |
| [ai-writing-assist](skills/ai-writing-assist/SKILL.md) | AI 帮写，辅助生成创意文案和优化提示词 |
| [gen-hot-image](skills/gen-hot-image/SKILL.md) | 爆款图片复刻，基于爆款风格生成产品复刻图 |
| [gen-hot-video](skills/gen-hot-video/SKILL.md) | 爆款视频复刻，支持创意复刻与 1:1 复刻 |
| [generate-video](skills/generate-video/SKILL.md) | 电商视频生成，根据提示词和参考素材生成产品视频 |
| [file-upload](skills/file-upload/SKILL.md) | 上传本地图片、视频、音频，返回 24 小时内可访问的直链 |

## 环境要求

- 用户需联系 Flyelep 客服获取 API 密钥 `secretKey`
- Flyelep 平台地址：https://www.flyelep.cn，接口文档：https://api.flyelep.cn/
- 技能发布页：GitHub https://github.com/FlyelepAI/agent-skills ，ClawHub https://clawhub.ai/flyelepai
- 接口基础地址：`https://www.flyelep.cn/prod-api`
- 本仓库中的技能统一要求在请求头中传入 `secretKey`（查询账户算力接口除外，见下文）
- 计费标准见 Flyelep 官网：首页左侧「算力 > 功能定价」

## 通用约定

- **任务 ID 一律按字符串处理**：19 位长整数，转成数字会丢精度
- **图片类异步任务**用 `POST /prod-api/poster-design/api/v1/poster/queryTaskResult` 查结果，返回 `data.status`（1 进行中 / 2 全部成功 / 3 全部失败 / 4 部分成功）和 `data.taskList[]`（`taskStatus` 0 待生成 / 1 生成中 / 2 成功 / 3 失败，`executeResult` 为结果地址或失败原因）；建议每 3~5 秒查询一次，`status` 不为 1 即停止
- **视频类异步任务**（`generate-video`、`gen-hot-video`）用 `POST /prod-api/poster-design/api/v1/poster/videoGen/queryResult`，返回格式同上（同样带 `data.status`）
- **AI 工具同步接口**的响应在 `code` / `msg` / `data` 之外新增了 `result`：`result.fileUrlList` 是结果地址数组，`result.taskId` 是主任务 ID。**新接入优先读 `result.fileUrlList`**，`data` 只为兼容老调用方保留

### 查询账户剩余算力

- `POST /prod-api/poster-design/api/v1/poster/queryApiCustomerPoint`
- 本接口的 `secretKey` 放在**请求体**中：`{"secretKey": "您的密钥"}`
- 返回 `data.remainingPoint`（剩余算力）、`data.usedPoint`（已用算力）、`data.maxConcurrency`（最大并发数）
- 批量生成前可先查一次余额和并发上限

## 技能列表

### generate-poster

生成电商产品图海报，适用于产品图、电商海报、Amazon 商品图、A+ 图、详情页图片、白底图等场景。

主要能力：

- `generateType`：100 产品单图、101 白底图、102 主副图、103 产品套图、200 详情图、300 宣传图
- 支持跨境电商与中文电商，语种由平台推导：中文平台默认 `中文简体`，跨境平台默认 `英文`
- 模型 `modelEdition`：`9` Flyelep Image 2（默认）、`13` Flyelep Image 2.5、`14` Flyelep Nano 2.1（Flyelep Nano、Nano 2 已下架）
- 通道 `channel`：特惠 `promotion`（默认，8 算力/张，支持以上三个模型）/ 尊享 `premium`
- 张数 1~20；参考图异步最多 10 张、同步最多 6 张
- 支持通用比例与亚马逊 A+ 尺寸 `970x600` / `1464x600` / `600x450`（仅限详情图 + Amazon）

接口入口：

- 异步（推荐）：`POST /prod-api/poster-design/api/v1/poster/generateAsync` + `POST /prod-api/poster-design/api/v1/poster/queryTaskResult`
- 同步：`POST /prod-api/poster-design/api/v1/poster/generate`（最长挂 15 分钟，`data` 为 `;` 拼接的图片地址，不返回任务 ID）

白底图用上述入口传 `generateType=101`；也可用同步专属入口 `POST /prod-api/poster-design/api/v1/poster/whiteBgMainImgGen`（Body 同 `/generate`，服务端固定 `generateType=101`）。

详细参数和映射规则请查看 [skills/generate-poster/SKILL.md](skills/generate-poster/SKILL.md)。

### async-free-creation

异步自由创作，根据提示词和可选参考图生成产品图或创意图片。

主要能力：

- 单次生成 1~20 张，参考图 `fileUrlList` 最多 6 张（旧字段 `apiImgUrlList` 仅兼容）
- 支持 `1:1`、`16:9`、`9:16` 等 10 种比例
- 模型与海报接口一致：`9` Flyelep Image 2（默认）、`13` Flyelep Image 2.5、`14` Flyelep Nano 2.1
- 默认走特惠通道 `promotion`

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/allAroundCreationAsync`
- `POST /prod-api/poster-design/api/v1/poster/queryTaskResult`

详细参数和轮询规则请查看 [skills/async-free-creation/SKILL.md](skills/async-free-creation/SKILL.md)。

### intelligent-extension

对图片进行智能延展（扩图、补边），支持批量处理（1~20 张），并可指定 `1:1`、`16:9`、`9:16` 等目标比例。

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/aiTool/intelligentExtension`

详细参数和比例规则请查看 [skills/intelligent-extension/SKILL.md](skills/intelligent-extension/SKILL.md)。

### image-translate

识别并翻译图片中的文字，保持原有版式重新生成图片。支持多图、多语言（`toList`），图片数 × 语言数不超过 20。

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/aiTool/translate`

详细语言枚举和调用方式请查看 [skills/image-translate/SKILL.md](skills/image-translate/SKILL.md)。

### partial-redrawing

对图片局部区域进行重绘，可通过文本提示词控制生成结果，也可结合参考替换图。需要改图中文字时改用 `text-edit`。

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/aiTool/partialRedrawing`

详细参数和提示词建议请查看 [skills/partial-redrawing/SKILL.md](skills/partial-redrawing/SKILL.md)。

### image-enlarge

对图片按倍数进行无损放大，支持单张或批量（最多 20 张）。放大倍率 `scalingRatio` 支持 `2`、`4`、`8`，未指定时用 `2`。

> 只想“变清晰”而不放大尺寸时，应改用 `image-clarity-enhance`。

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/aiTool/enlarge`

详细调用方式请查看 [skills/image-enlarge/SKILL.md](skills/image-enlarge/SKILL.md)。

### ai-image-matting

自动去除图片背景、保留主体，输出透明底 PNG。图片入参 `imageUrlList` 数组，最多 20 张。

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/aiTool/aiImageMatting`

详细参数和批量处理方式请查看 [skills/ai-image-matting/SKILL.md](skills/ai-image-matting/SKILL.md)。

### scene-replace

保留商品主体，将背景替换为指定场景。必传 `sourceUrl`、`modelType`（仅支持 `9` Flyelep Image 2），场景参考图 `replaceImageUrl` 可选。

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/aiTool/sceneReplace`

详细参数说明请查看 [skills/scene-replace/SKILL.md](skills/scene-replace/SKILL.md)。

### product-replace

将场景图中的原商品替换为指定商品，保持场景和光影不变。`modelType` 仅支持 `9`（Flyelep Image 2）。

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/aiTool/productReplace`

详细参数说明请查看 [skills/product-replace/SKILL.md](skills/product-replace/SKILL.md)。

### product-color-change

将商品更换为指定颜色，保持材质、结构和背景不变。`modelType` 仅支持 `9`（Flyelep Image 2）。

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/aiTool/productColorChange`

详细提示词建议请查看 [skills/product-color-change/SKILL.md](skills/product-color-change/SKILL.md)。

### image-clarity-enhance

提升图片清晰度与细节，支持批量。图片入参 `imageUrlList` 数组；增强强度 `enhanceStrength`：`light`（默认）/ `standard` / `strong`，其他值报错。

图片规格：长边不超过 `2048px`、短边不少于 `256px`、文件不超过 `10MB`。

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/aiTool/imageClarityEnhance`

详细参数说明请查看 [skills/image-clarity-enhance/SKILL.md](skills/image-clarity-enhance/SKILL.md)。

### long-image-fusion

将多张图片（至少 2 张）按顺序融合为一张长图，并对拼接处做融合处理。同步返回，按成功张数计费。

接口入口：

- `POST /prod-api/poster-design/api/v1/aiTool/longImageFusion`

详细参数说明请查看 [skills/long-image-fusion/SKILL.md](skills/long-image-fusion/SKILL.md)。

### image-layer-decompose

将图片拆分为 1 张底图和最多 16 个透明 PNG 图层，每个图层附带叠放顺序、位置、名称和描述。同步返回，支持 1~20 张批量；单张分辨率需在 512×512 ~ 6000×6000 之间，按产出图层数计费。

接口入口：

- `POST /prod-api/poster-design/api/v1/aiTool/imageLayerDecompose`

详细参数和返回结构请查看 [skills/image-layer-decompose/SKILL.md](skills/image-layer-decompose/SKILL.md)。

### main-image-tagging

复刻一张参考图的版式，把同一组卖点和价格打到 1~20 张商品图上，每张商品图各产出一张打标图。`modelType` 仅支持 `9`，固定特惠通道，成功一张扣 8 算力。

接口入口：

- `POST /prod-api/poster-design/api/v1/aiTool/mainImageTagging`

详细参数说明请查看 [skills/main-image-tagging/SKILL.md](skills/main-image-tagging/SKILL.md)。

### image-text-recognize

识别单张图片里的文字，返回每行文字内容和四角坐标，每次识别成功扣 1 算力。通常作为 `text-edit` 的前一步。

接口入口：

- `POST /prod-api/poster-design/api/v1/aiTool/recognizeImageText`

详细参数和坐标换算请查看 [skills/image-text-recognize/SKILL.md](skills/image-text-recognize/SKILL.md)。

### text-edit

在单张原图上按 `textReplaceList`（原文、新文、`rect` 坐标）逐条替换文字，并尽量保持原有版式。一次只处理一张图，`modelType` 为字符串 `"9"`，成功扣 8 算力。

接口入口：

- `POST /prod-api/poster-design/api/v1/aiTool/textEdit`

详细参数说明请查看 [skills/text-edit/SKILL.md](skills/text-edit/SKILL.md)。

### ai-writing-assist

AI 帮写，根据描述（可附带参考图，最多 6 张）生成若干条创意文案，可用于优化提示词或获取创意灵感。同步接口，最长约 5 分钟；每天 10 次免费，超出每次 2 积分。

接口入口：

- `POST /prod-api/poster-design/api/v1/aiTool/assistedGeneration`

详细参数和文案结构说明请查看 [skills/ai-writing-assist/SKILL.md](skills/ai-writing-assist/SKILL.md)。

### gen-hot-image

爆款图片复刻，参考爆款图片，用商品图复刻生成同风格图片。`modelType` 支持 `9`（Flyelep Image 2，推荐）和 `2`（Flyelep Nano 2）。异步接口。

接口入口：

- `POST /prod-api/poster-design/api/v1/aiTool/generateHotImage`
- `POST /prod-api/poster-design/api/v1/poster/queryTaskResult`

详细参数和轮询规则请查看 [skills/gen-hot-image/SKILL.md](skills/gen-hot-image/SKILL.md)。

### gen-hot-video

爆款视频复刻，参考爆款视频，用商品素材复刻生成新视频。

主要能力：

- 复刻模式 `replicaMode`：创意复刻（默认）与 1:1 复刻
- 素材 `replaceUrl` 可混传图片、视频、音频
- 支持 `pro` / `fast` / `ultra` 三种模型档位
- 异步接口，成功后按秒数扣费

接口入口：

- `POST /prod-api/poster-design/api/v1/aiTool/generateHotVideo`
- `POST /prod-api/poster-design/api/v1/poster/videoGen/queryResult`

详细参数、模型类型和轮询规则请查看 [skills/gen-hot-video/SKILL.md](skills/gen-hot-video/SKILL.md)。

### generate-video

电商视频生成，根据提示词和参考素材生成产品展示、种草、品牌宣传等视频。

主要能力：

- 时长 4~30 秒（超过单段上限会自动拆段拼接）
- 分辨率 `480p` / `720p` / `1080p` / `4K`，`fast` 和 `ultra` 只支持 `480p` / `720p`
- 模型：`pro` Flyelep Dance 2.0（默认）、`fast` Flyelep Dance 2.0 Fast、`ultra` Flyelep Dance 2.5
- 参考图、参考视频、首帧、尾帧至少传一项（不支持纯文生视频），可附加参考音频
- 支持生成或关闭配音音频

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/generateVideo`
- `POST /prod-api/poster-design/api/v1/poster/videoGen/queryResult`

视频业务标签：`主图视频`、`种草视频`、`品牌宣传`。

详细参数说明、素材规则和轮询策略请查看 [skills/generate-video/SKILL.md](skills/generate-video/SKILL.md)。

### file-upload

把本地图片、视频、音频文件上传到云存储，返回可公网访问的直链，用于为其它技能准备素材入参。

主要能力：

- 上传本地图片、视频、音频，换回公网可访问的 URL
- 返回的直链不带签名，**只保证 24 小时内可访问**，应尽快用于后续生成任务
- 图片在上传前自动执行内容审核，视频和音频不审核；进桶的是原始文件，不会被压缩降质
- 上传不消耗算力，对象会重命名为 UUID，原文件名不进 URL，中文名可直接上传

接口入口：

- `POST /prod-api/poster-design/api/v1/file/upload`

请求方式为 `multipart/form-data`，文件字段名为 `file`，单次只能上传一个文件。

支持格式：

- 图片：`bmp`、`gif`、`jpg`、`jpeg`、`png`
- 视频：`mp4`、`mov`、`m4v`、`webm`、`avi`、`mkv`
- 音频：`mp3`、`wav`、`m4a`、`aac`、`ogg`、`flac`

详细参数说明和错误处理请查看 [skills/file-upload/SKILL.md](skills/file-upload/SKILL.md)。

## 使用建议

- 所有技能均以各自目录中的 `SKILL.md` 为准
- 对于需要高稳定性的调用，优先使用可公网访问、格式规范的图片直链
- 用户提供的是本地文件而非直链时，先用 `file-upload` 上传换取直链，再调用其它技能
- 涉及素材入参的技能都在自己的 `SKILL.md` 里内联了「本地文件上传」章节作为兜底，只单独安装某一个技能也能完成上传；**上传接口若有变动，需同步更新这些章节**
- 对于场景替换、商品替换、商品换色这类生成型接口，建议尽量同时提供清晰原图和明确的文本描述
- 改图中文字：先用 `image-text-recognize` 拿到原文和坐标，再用 `text-edit` 替换

## License

MIT
