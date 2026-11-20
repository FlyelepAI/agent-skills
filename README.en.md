<p align="center"><a href="https://www.flyelep.ai/"><img src="assets/logo.png" alt="Flyelep" width="96"></a></p>

# Flyelep AI Agent Skills

### E-commerce images · editing · video — just tell your AI Agent

**Plug Flyelep's e-commerce AI into Claude Code, OpenClaw, Cursor, Codex and other agents. Describe what you need in one sentence and get main images, image sets, detail pages, viral-style replicas and product videos.**

[中文](README.md) · **English**

[30-second Quickstart](#-30-second-quickstart) · [What It Does](#-what-it-does) · [Showcase](#-showcase) · [Tutorial](#-tutorial) · [All Skills](#-all-skills) · [ClawHub](https://clawhub.ai/flyelepai)

---

## ✨ Why Flyelep Skills

| Before | With Flyelep Skills |
|---|---|
| Wait days for a designer to deliver one set of listing images | One sentence to your agent, a full set of main + detail images in minutes |
| Jump between tools for matting, background swap, recoloring, upscaling | 20 skills in one agent, combined automatically as needed |
| Re-make every image for each marketplace language | Translate images into multiple languages at once, layout preserved |
| Hard to copy what makes a bestseller work | Send a viral image or video, replicate the style with your own product |
| Integrating an API means docs, code and polling logic | Parameters, polling and local file upload are built into each skill |

---

## 🚀 30-second Quickstart

### 1. Install

Pick one:

- **ClawHub (OpenClaw)**: open [clawhub.ai/flyelepai](https://clawhub.ai/flyelepai) and install the skills you need
- **skills CLI**:

  ```bash
  npx skills add FlyelepAI/agent-skills
  ```

- **Manual (Claude Code / Cursor / Codex, etc.)**: copy the skill folders under `skills/` into your agent's skills directory (e.g. `~/.claude/skills/` for Claude Code), or simply give the repo URL `https://github.com/FlyelepAI/agent-skills` to your agent and ask it to load the skills

### 2. Set your secret key

Sign in to the [Flyelep Open Platform](https://www.flyelep.ai/) → "API Key" to create a key (or contact Flyelep support), then tell your agent:

```
My Flyelep secretKey is sk-xxxx. Use it for all Flyelep skills.
```

> Keep the key in your local agent session or environment variables. Never commit it to a public repo.

### 3. Just ask

```
Here is my Bluetooth earbuds product photo (local file). Make an Amazon US main image set in English, 5 images.
```

The agent will upload the local file → call `generate-poster` → poll for the result → send you the image links.

---

## 🎯 What It Does

| Category | Use case | Skill |
|---|---|---|
| **Create** | Product image, white background, main + sub images, image sets, detail pages, promo images, Amazon A+ | `generate-poster` |
| | Free-form creative images in many aspect ratios | `async-free-creation` |
| | Replicate a viral image's style with your product | `gen-hot-image` |
| | Batch-add selling points and price tags to main images | `main-image-tagging` |
| **Edit** | Matting, scene swap, product swap, recolor | `ai-image-matting` `scene-replace` `product-replace` `product-color-change` |
| | Inpainting, edit text inside images | `partial-redrawing` `image-text-recognize` `text-edit` |
| | Enhance clarity, lossless upscale, outpaint to a new ratio | `image-clarity-enhance` `image-enlarge` `intelligent-extension` |
| | Fuse images into a long image, split into layers | `long-image-fusion` `image-layer-decompose` |
| | Translate images into multiple languages | `image-translate` |
| **Video** | Product main video, seeding video, brand promo | `generate-video` |
| | Replicate a viral video (creative / 1:1) | `gen-hot-video` |
| **Helpers** | Copywriting and prompt polishing | `ai-writing-assist` |
---

## 🖼 Showcase

### Case 1: A full Amazon listing in 30 minutes

> "Here is a white-background photo of my insulated bottle with key points (316 stainless steel, 12-hour insulation, 500ml). Make an Amazon US main image set and 3 A+ detail images, in English."

- **Skills used**: `generate-poster` (main set + detail images, A+ size `970x600`)
- **Output**: a main image set + 3 A+ images, English selling points rendered in the images
- **Follow up**: "Translate this set into German and Japanese" → `image-translate` delivers both languages at once

### Case 2: Copy what's already selling

> "This is a competitor's bestselling main image. Replicate the composition and mood with my lipstick photo, 4 images."
> "Now use this viral short video as a reference and make a 15-second creative replica with my product images."

- **Skills used**: `gen-hot-image` (viral image replica), `gen-hot-video` (viral video replica)
- **Output**: 4 on-style product images + 1 product video, with your product's real look preserved

### Case 3: From a quick phone snap to listing-ready assets

> "This is a quick photo of sneakers in my warehouse. Cut it out, put it on a city street, make a black version, then upscale 2x."

- **Skills used**: `ai-image-matting` → `scene-replace` → `product-color-change` → `image-enlarge`
- **Output**: transparent cutout, street scene, black colorway scene, upscaled HD image — chained automatically by the agent

### Case 4: Batch promo tags

> "Use this reference layout and add '50% OFF' and price $9.99 to all 20 product images."

- **Skills used**: `main-image-tagging`
- **Output**: 20 promo main images with a consistent layout

> More result images coming soon.

---

## 📘 Tutorial

### Things you can say

| You want | Say something like |
|---|---|
| A main image set | "Make a TikTok Shop main image set, 5 images, selling points: …" |
| White background | "Turn this product photo into a pure white background main image" |
| Marketplace and language | "For Amazon US, English, 1:1" |
| A specific model | "Generate with Flyelep Image 2.5" |
| A new aspect ratio | "Extend this image to 9:16 portrait" |
| Edit text in an image | "Change 'New Arrival' in the image to 'Black Friday'" |
| A video | "Make a 10-second main video from these three product images, 1080p" |
| Check balance | "How much compute do I have left on Flyelep?" |

### Tips

- **Send local files directly**: the agent uploads them automatically (URLs stay valid for 24 hours), then calls the right skill
- **Clear inputs, better results**: for scene swap, product swap and recolor, provide a sharp source image plus a clear text description
- **Check balance before big batches**: ask the agent to check remaining compute and concurrency limits first
- **Chain steps in one go**: describe multi-step needs (cutout → scene swap → upscale) and the agent runs them in order
- **Pricing**: see [Flyelep](https://www.flyelep.ai/) left menu "Compute > Pricing"

---

## 🧩 All Skills

| Skill | Description |
|------|------|
| [generate-poster](skills/generate-poster/SKILL.md) | E-commerce product, white background, main + sub, sets, detail and promo images |
| [async-free-creation](skills/async-free-creation/SKILL.md) | Async free-form creation, 1–20 creative images with selectable models |
| [intelligent-extension](skills/intelligent-extension/SKILL.md) | Smart outpainting with batch support and target aspect ratios |
| [image-translate](skills/image-translate/SKILL.md) | Recognize and translate text in images, multi-image and multi-language |
| [partial-redrawing](skills/partial-redrawing/SKILL.md) | Redraw a local area with text prompts and optional reference images |
| [image-enlarge](skills/image-enlarge/SKILL.md) | Lossless upscaling, single or batch |
| [ai-image-matting](skills/ai-image-matting/SKILL.md) | Remove backgrounds automatically, batch supported |
| [scene-replace](skills/scene-replace/SKILL.md) | Replace the background scene with reference images and text |
| [product-replace](skills/product-replace/SKILL.md) | Swap the product in an image while keeping the scene and lighting |
| [product-color-change](skills/product-color-change/SKILL.md) | Detect the product and change its color |
| [image-clarity-enhance](skills/image-clarity-enhance/SKILL.md) | AI clarity enhancement, batch supported |
| [long-image-fusion](skills/long-image-fusion/SKILL.md) | Fuse multiple images into one long image |
| [image-layer-decompose](skills/image-layer-decompose/SKILL.md) | Split an image into a base layer and up to 16 transparent layers |
| [main-image-tagging](skills/main-image-tagging/SKILL.md) | Batch-apply selling points and prices following a reference layout |
| [image-text-recognize](skills/image-text-recognize/SKILL.md) | OCR that returns each text line with coordinates |
| [text-edit](skills/text-edit/SKILL.md) | Replace text in an image while keeping the layout |
| [ai-writing-assist](skills/ai-writing-assist/SKILL.md) | AI copywriting and prompt polishing |
| [gen-hot-image](skills/gen-hot-image/SKILL.md) | Replicate a viral image's style for your product |
| [gen-hot-video](skills/gen-hot-video/SKILL.md) | Replicate a viral video, creative or 1:1 |
| [generate-video](skills/generate-video/SKILL.md) | Generate e-commerce videos from prompts and reference assets |
See [SKILL.md](SKILL.md) for an overview of every skill and its endpoints, and each skill's own `SKILL.md` for full parameters. Skill files are written in Chinese; agents read them fine in any language.

---

## 🔧 For Developers

- API docs: https://api.flyelep.cn/ , base URL `https://www.flyelep.cn/prod-api`
- Every skill passes `secretKey` in the request header (except the account balance endpoint, which takes it in the body)
- Async image tasks are queried via `/poster/queryTaskResult`; video tasks (`generate-video`, `gen-hot-video`) via `/poster/videoGen/queryResult`
- Sync AI-tool endpoints now return `result.fileUrlList` / `result.taskId`; new integrations should read `result` first
- Each skill's own `SKILL.md` is the source of truth; skills that take media inputs include an inline "local file upload" section, so a single skill installed alone can still upload files

## 🔗 Links

- Flyelep: https://www.flyelep.ai/
- ClawHub: https://clawhub.ai/flyelepai
- API docs: https://api.flyelep.cn/

## License

MIT
