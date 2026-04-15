# Agent Skills

Flyelep AI Agent 技能集合，可通过仓库 URL 被 OpenClaw、Claude Code 等 AI 工具加载。

当前仓库中的技能全部基于 Flyelep API 接口文档整理，主要覆盖电商海报生成与 AI 图片工具两大类能力。

## 可用技能

| 技能 | 描述 |
|------|------|
| [poster](skills/poster/SKILL.md) | 生成电商产品主图和详情图海报 |
| [intelligent-extension](skills/intelligent-extension/SKILL.md) | 智能延展图片，支持批量处理和目标比例适配 |
| [image-translate](skills/image-translate/SKILL.md) | 识别并翻译图片中的文字，返回翻译后的新图片 |
| [partial-redrawing](skills/partial-redrawing/SKILL.md) | 对图片局部区域进行重绘，可结合文本提示词和参考图 |
| [image-enlarge](skills/image-enlarge/SKILL.md) | 无损放大图片，支持单张或批量增强 |
| [ai-image-matting](skills/ai-image-matting/SKILL.md) | 自动去除图片背景，支持批量抠图 |
| [scene-replace](skills/scene-replace/SKILL.md) | 替换图片背景场景，可结合参考图和文本描述 |
| [product-replace](skills/product-replace/SKILL.md) | 替换图片中的商品主体，保留背景和光影效果 |
| [product-color-change](skills/product-color-change/SKILL.md) | 智能识别商品并进行换色处理 |
| [image-clarity-enhance](skills/image-clarity-enhance/SKILL.md) | AI 超清增强图片清晰度，支持批量处理 |

## 安装方式

### OpenClaw

将本仓库加入 OpenClaw 的技能来源，或将单个技能目录安装到本地 workspace。

如果使用仓库地址方式接入：

```text
https://github.com/FlyelepAI/agent-skills
```

如果使用本地 OpenClaw skills 目录方式安装，技能通常位于：

```text
~/.openclaw/workspace/skills/<skill-name>/
```

### Claude Code

使用 `/install-skill` 命令安装单个技能：

```bash
/install-skill https://github.com/FlyelepAI/agent-skills/skills/poster
```

将 `poster` 替换为对应技能目录名即可安装其他技能。

## 环境要求

- 用户需从 Flyelep 开放平台获取 API 密钥
- Flyelep 平台地址：https://www.flyelep.cn
- 除 `poster` 外，本仓库中的 AI 工具技能统一要求在请求头中传入 `secretKey`
- `poster` 技能按接口要求在请求 body 中传入 `secretKey`

## 技能列表

### poster

生成电商产品主图和详情图海报，适用于产品图、电商海报、Amazon 商品图、详情页图片等场景。

主要能力：

- 生成产品单图和产品详情图
- 支持跨境电商与中文电商
- 支持多平台、多语言文案
- 支持参考图输入和比例控制

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/generate`

详细参数和映射规则请查看 [skills/poster/SKILL.md](skills/poster/SKILL.md)。

### intelligent-extension

对图片进行智能延展，支持批量处理，并可指定目标比例。

主要能力：

- 扩图、补边、延展画布
- 适配 `1:1`、`16:9`、`9:16` 等比例
- 支持多张图片批量处理

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/aiTool/intelligentExtension`

详细参数和比例规则请查看 [skills/intelligent-extension/SKILL.md](skills/intelligent-extension/SKILL.md)。

### image-translate

识别并翻译图片中的文字，返回翻译后的新图片地址。

主要能力：

- 图片文字识别与翻译
- 支持自动识别源语言
- 支持整数枚举目标语言映射

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/aiTool/translate`

详细语言枚举和调用方式请查看 [skills/image-translate/SKILL.md](skills/image-translate/SKILL.md)。

### partial-redrawing

对图片局部区域进行重绘，可通过文本提示词控制生成结果，也可结合参考替换图。

主要能力：

- 局部替换
- 局部重绘
- 替换背景、文案或局部元素

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/aiTool/partialRedrawing`

详细参数和提示词建议请查看 [skills/partial-redrawing/SKILL.md](skills/partial-redrawing/SKILL.md)。

### image-enlarge

对图片进行无损放大，支持单张或批量增强。

主要能力：

- 提高清晰度
- 放大图片尺寸
- 批量增强商品图

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/aiTool/enlarge`

增强强度为：

- `1`：轻度增强
- `2`：标准增强
- `3`：强力增强

详细调用方式请查看 [skills/image-enlarge/SKILL.md](skills/image-enlarge/SKILL.md)。

### ai-image-matting

自动去除图片背景，适合商品抠图、透明底素材生成等场景。

主要能力：

- 去背景
- 抠出主体
- 批量抠图

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/aiTool/aiImageMatting`

详细参数和批量处理方式请查看 [skills/ai-image-matting/SKILL.md](skills/ai-image-matting/SKILL.md)。

### scene-replace

将图片中的背景场景替换为指定场景，可结合参考图和文本提示词控制效果。

主要能力：

- 更换背景场景
- 替换商品展示环境
- 保留主体不变，仅替换环境

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/aiTool/sceneReplace`

模型类型：

- `0`：`gemini-2.5`
- `1`：`gemini-3-pro`

详细参数说明请查看 [skills/scene-replace/SKILL.md](skills/scene-replace/SKILL.md)。

### product-replace

将图片中的商品替换为指定商品图，同时尽量保留原图背景及光影效果。

主要能力：

- 更换商品主体
- 保留原场景和展示环境
- 支持目标商品图和文本补充约束

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/aiTool/productReplace`

模型类型：

- `0`：`gemini-2.5`
- `1`：`gemini-3-pro`

详细参数说明请查看 [skills/product-replace/SKILL.md](skills/product-replace/SKILL.md)。

### product-color-change

对图片中的商品进行智能换色处理，适合制作同款不同颜色展示图。

主要能力：

- 修改商品颜色
- 保留商品主体和背景
- 支持通过提示词约束换色范围和效果

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/aiTool/productColorChange`

模型类型：

- `0`：`gemini-2.5`
- `1`：`gemini-3-pro`

详细提示词建议请查看 [skills/product-color-change/SKILL.md](skills/product-color-change/SKILL.md)。

### image-clarity-enhance

增强图片清晰度，支持单张或批量处理，适合做 AI 超清增强。

主要能力：

- 清晰度增强
- 批量超清处理
- 根据强度控制增强程度

接口入口：

- `POST /prod-api/poster-design/api/v1/poster/aiTool/imageClarityEnhance`

增强强度：

- `light`
- `standard`
- `strong`

图片规格限制：

- 仅支持 `JPG`、`PNG`、`BMP`
- 最短边不少于 `10px`
- 最长边不超过 `5000px`
- 长宽比不超过 `4:1`
- 文件大小不超过 `8MB`

详细参数说明请查看 [skills/image-clarity-enhance/SKILL.md](skills/image-clarity-enhance/SKILL.md)。

## 使用建议

- 所有技能均以各自目录中的 `SKILL.md` 为准
- 当 Flyelep 接口文档更新时，应同步更新对应 skill
- 对于需要高稳定性的调用，优先使用可公网访问、格式规范的图片直链
- 对于场景替换、商品替换、商品换色这类生成型接口，建议尽量同时提供清晰原图和明确的文本描述

## License

MIT
