# Agent Skills

AI Agent 技能集合，可通过 GitHub URL 被 OpenClaw、Claude Code 等 AI 工具调用。

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
- Python 3
- 环境变量 `FLYELEP_SECRET_KEY`（从 Flyelep API开放平台获取：https://www.flyelep.cn）

**使用示例：**

```bash
# 生成 Amazon 产品详情图
python3 scripts/generate_poster.py \
  --query "为这款蓝牙耳机生成产品详情图" \
  --file-urls "https://example.com/product.png"
```

详细参数说明请查看 [skills/poster/skills.md](skills/poster/SKILL.md)。

## 添加新技能

在 `skills/` 目录下创建新的技能文件夹，包含 `skills.md` 文件。格式参考现有技能。

## License

MIT
