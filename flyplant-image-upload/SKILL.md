---
name: flyplant-image-upload
description: 调用飞象 Agent 开放接口，把本地图片上传到云存储（腾讯云 COS）并拿到可访问的图片 URL。当需要上传图片、把本地图片转成图床链接、或为飞象其它开放接口（海报生成、AI 工具、图层拆分等）准备 imageUrl 入参时使用。
---

# 飞象图片上传

把本地图片文件传到飞象云存储，换回一个永久可访问的图片 URL。

## 接口

```
POST {BASE_URL}/poster-design/api/v1/file/upload
Content-Type: multipart/form-data
```

| 项 | 值 |
|---|---|
| 鉴权 | 请求头 `secretKey: <你的密钥>` |
| 文件字段 | `file`，图片二进制，multipart/form-data |
| 文件归类 | 固定为「飞象Agent」类型，落到 COS 的 `cos_ai_agent/` 目录 |

`BASE_URL` 是网关地址，`/poster-design` 这层前缀不能省。

## 响应

成功：

```json
{
  "code": 200,
  "msg": null,
  "data": {
    "relativePath": "cos_ai_agent/2026-08-11/3f2a9c1b7d84e6f5a012.png",
    "fullPath": "https://agent-1404002717.cos.ap-guangzhou.myqcloud.com/cos_ai_agent/2026-08-11/3f2a9c1b7d84e6f5a012.png",
    "serviceProvider": null
  }
}
```

**判断成功看 `code == 200`,不要只看 HTTP 状态码**——业务失败时 HTTP 仍是 200,`code` 会是 500,原因在 `msg` 里。

拿到之后用哪个字段：

- **`fullPath`** — 完整可访问 URL,永久有效、不带签名不会过期。要展示图片、要把图片喂给飞象其它开放接口的 `imageUrl` 参数,都用这个。
- **`relativePath`** — COS 对象 key,是存储侧的内部标识。除非对方接口明确要求 key,否则不要传它。

## 调用示例

curl：

```bash
curl -X POST "${BASE_URL}/poster-design/api/v1/file/upload" \
  -H "secretKey: ${SECRET_KEY}" \
  -F "file=@./product.png"
```

Python：

```python
import requests

with open("product.png", "rb") as f:
    resp = requests.post(
        f"{BASE_URL}/poster-design/api/v1/file/upload",
        headers={"secretKey": SECRET_KEY},
        files={"file": ("product.png", f, "image/png")},
        timeout=60,
    )

body = resp.json()
if body["code"] != 200:
    raise RuntimeError(f"上传失败: {body['msg']}")
image_url = body["data"]["fullPath"]
```

TypeScript / Node：

```ts
const form = new FormData();
form.append('file', new Blob([bytes], { type: 'image/png' }), 'product.png');

const resp = await fetch(`${BASE_URL}/poster-design/api/v1/file/upload`, {
  method: 'POST',
  headers: { secretKey: SECRET_KEY },
  body: form,
});

const body = await resp.json();
if (body.code !== 200) throw new Error(`上传失败: ${body.msg}`);
const imageUrl = body.data.fullPath;
```

不要手动设置 `Content-Type`,让 HTTP 客户端自己带上 multipart 的 boundary。

## 约束

**只支持 bmp、gif、jpg、jpeg、png**。后缀取自文件名,拿不到时回退到 Content-Type。其它格式（含 webp）会被直接拒绝——这些后缀绕不过内容审核,服务端不放行。

图片会先压到 1024×1024 走内容审核,再进 COS。审核不通过整个请求失败,不会有半成品文件。审核加上传通常几秒,超时设到 60 秒以上。

单次只能传一个文件。要传多张就并发调多次,每次拿一个独立 URL。

## 错误处理

| `msg` | 原因 | 处理 |
|---|---|---|
| 上传的图片不能为空 | 没带 `file` 字段,或文件是 0 字节 | 检查表单字段名是不是 `file` |
| 图片格式不支持，仅支持：... | 后缀不在白名单 | 先转成 png 或 jpg 再传 |
| 密钥不能为空! | 没带 `secretKey` 请求头 | 补上请求头 |
| 密钥无效! | 密钥错误或已失效 | 换一个有效密钥 |
| 图片违规 / 具体违规描述 | 内容审核没过 | 换图,重试没用 |

密钥问题和格式问题重试都不会好,别做自动重试。只有网络超时和 5xx 值得重试。
