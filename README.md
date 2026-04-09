# Agent Skills

AI Agent 海报技能集合，可通过 GitHub URL 被 OpenClaw、Claude Code 等 AI 工具调用。

## 可用技能

| 技能 | 描述 |
|------|------|
| [poster](skills/poster/SKILL.md) | 生成电商产品主图和详情图海报 |

## 安装方式

### OpenClaw / Cursor

在设置中添加技能仓库 URL：

```
https://github.com/FlyelepAI/agent-skills
```

### Claude Code

使用 `/install-skill` 命令安装：

```
/install-skill https://github.com/FlyelepAI/agent-skills/skills/poster
```

## 技能列表

### poster

生成电商产品主图和详情图海报，支持：

- 跨境电商：Amazon、Temu、Shopee、TikTok Shop、AliExpress、OZON
- 中文电商：淘宝、京东、拼多多、1688、小红书
- 多语言文案：英文、中文、日语、韩语等 18 种语言

**环境要求：**
- 用户需从 Flyelep API 开放平台获取 API 密钥（https://www.flyelep.cn）

**使用示例：**

**生成产品主图（跨境电商，Amazon）：**
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generate" \
  -H "Content-Type: application/json" \
  --max-time 600 \
  -d '{
    "query": "为这个蓝牙耳机生成一张白底产品主图",
    "generateType": 100,
    "posterType": 5,
    "platformType": "amazon",
    "languageType": "英文",
    "detailPictureNumber": 1,
    "modelEdition": 3,
    "needText": true,
    "secretKey": "你的密钥"
  }'
```

**生成产品详情图（带参考图）：**
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generate" \
  -H "Content-Type: application/json" \
  --max-time 600 \
  -d '{
    "query": "根据上传的图片生成对应的产品图",
    "generateType": 200,
    "posterType": 5,
    "platformType": "amazon",
    "languageType": "英文",
    "detailPictureNumber": 5,
    "modelEdition": 3,
    "needText": true,
    "secretKey": "你的密钥",
    "fileUrlList": ["https://example.com/product1.png", "https://example.com/product2.png"],
    "aspectRatio": "1:1"
  }'
```

**中文电商主图（淘宝，中文简体）：**
```bash
curl -X POST "https://www.flyelep.cn/prod-api/poster-design/api/v1/poster/generate" \
  -H "Content-Type: application/json" \
  --max-time 600 \
  -d '{
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
  }'
```

详细参数说明请查看 [skills/poster/SKILL.md](skills/poster/SKILL.md)。

## License

MIT
