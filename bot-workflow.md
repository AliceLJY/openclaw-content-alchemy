# Bot Article Workflow

> Complete workflow for an OpenClaw bot to write, illustrate, and publish WeChat articles via Claude Code.
>
> OpenClaw bot 写文章的完整流程：写稿 → 确认 → 配图 → 发布。

## Prerequisites

- [openclaw-worker](https://github.com/AliceLJY/openclaw-worker) Task API running on `host.docker.internal:3456`
- [Content Alchemy](https://github.com/AliceLJY/content-alchemy) skill installed in Claude Code
- Chrome running with `--remote-debugging-port=9222` for WeChat publishing
- Image generation script at `workspace/scripts/generate-image.mjs`

## API Call Format

```
POST http://host.docker.internal:3456/claude
Authorization: Bearer <your-token>
```

```json
{
  "prompt": "...",
  "timeout": 600000,
  "callbackChannel": "<discord-channel-id>",
  "callbackContainer": "<your-bot-container-name>"
}
```

> `callbackContainer` must match your bot's Docker container name. Without it, callbacks go to the default container (wrong bot gets the response).

---

## Step 1: CC Writes Article

> CC 执行 Content Alchemy Stage 1-5 写稿，不生成图片文件，但预留占位符。

**API request:**
- `timeout`: 600000
- `callbackChannel`: current channel ID
- `callbackContainer`: your bot container name

**Prompt template** (replace `{topic}` with actual topic):

```
/content-alchemy 话题：{topic}

【Bot 模式指令】
1. 执行 Stage 1-5（话题挖掘→素材提取→分析验证→精炼润色→排版成文）
2. 不要调用 gemini-image-gen.ts 生成图片文件
3. 但必须在文章正文的合适位置保留图片占位标记：封面用 {{IMAGE_COVER}}，正文插图用 {{IMAGE_1}} {{IMAGE_2}} 等
4. 每个占位标记旁边用 HTML 注释写明图片描述，例如 {{IMAGE_1}} <!-- description -->
5. 文章必须完整排版（标题、小节、签名块都要有），不要只输出纯文本
6. 文章输出路径必须在 ~/Desktop/bot-articles/{slug}/ 目录下
7. 任务结束前输出一个 JSON 代码块，格式：
   {"slug": "dir-name", "article_path": "full-path", "images": [{"placeholder": "IMAGE_COVER", "prompt_en": "English description", "position": "cover"}, ...]}
```

**After callback returns**, extract:
- `slug`, `article_path`, `images` list from JSON
- `sessionId` (needed for Step 3)

> If CC fails, **tell user and stop**. Do not auto-retry.

---

## Step 1.5: User Confirms Angle/Framework

> 用户可能在外面用手机看文章，等用户确认角度再继续。

After CC finishes, tell the user:

```
文章写好了，放在 ~/Desktop/bot-articles/{slug}/article.md。
你手机看一下，角度和框架 OK 吗？OK 我就开始配图了。
```

**Wait for user confirmation before proceeding.** If user has edits, call CC to revise.

---

## Step 2: Bot Generates Illustrations (Auto Style Rotation + nano Enhancement)

> Bot 自动选风格 + nano 增强 prompt、生图。56种风格自动轮换，nano-banana-pro 强制一起用，不问用户。

### Style Selection Logic

1. Read `workspace/images/style-catalog.md`, match 4-5 candidates to article tone (single or combo)
2. Read `workspace/images/style-history.txt`, exclude styles used in the last 5 articles
3. Pick one from remaining candidates
4. Announce selection: "这篇文章调性是{tone}，我用「{style}」风格配图"
5. Proceed without waiting for reply (user can override anytime)
6. After generation, append to `style-history.txt`: `{date} {slug} {style-name}`
7. Keep only the last 20 entries in history

### nano-banana-pro Prompt Enhancement (mandatory)

> 风格选完后，**必须**用 nano-banana-pro 参考库增强 prompt，不可跳过。

1. Extract visual keywords from the article (scene, mood, objects)
2. Search `nano-banana-pro-prompts-recommend-skill/references/` for matching reference prompts (11 categories: social-media-post, poster-flyer, comic-storyboard, etc.)
3. Borrow structure from reference prompts (scene description, lighting, composition, camera angle)
4. Fuse with selected style suffix into final prompt

**Final prompt format:**

```
[主题场景描述], [56风格英文prompt后缀], [nano参考库的增强元素(灯光/构图/镜头等)]
```

### Image Generation

```bash
node /home/node/.openclaw/workspace/scripts/generate-image.mjs \
  --prompt "{final enhanced prompt}" \
  --output "/home/node/.openclaw/workspace/images/{slug}/{placeholder}.png"
```

- Send Discord preview after each image
- Ask user if satisfied after all images are done

---

## Step 3: CC Embeds Images + Publishes

> 发布前先检查微信登录状态，没登录就停。

### Login Check (mandatory before publish)

**API request:**
- `timeout`: 60000

**Prompt:**
```
检查微信公众号登录状态：用 Playwright 连接 Chrome CDP（localhost:9222），打开 https://mp.weixin.qq.com/ ，截图看是否在登录页还是已登录首页。只返回结果：LOGGED_IN 或 NOT_LOGGED_IN。
```

- `NOT_LOGGED_IN` → Tell user "微信未登录，需要到电脑上扫码登录后再发布", then **stop**
- `LOGGED_IN` → Proceed with publish

### Publish Request

**API request:**
- `timeout`: 600000
- `sessionId`: from Step 1

**Prompt template:**
```
继续 Content Alchemy。

图片已生成，Mac 本地路径：
- ~/.openclaw-<botname>/workspace/images/{slug}/IMAGE_COVER.png
- ~/.openclaw-<botname>/workspace/images/{slug}/IMAGE_1.png
（list all images）

请执行：
1. 将图片复制到文章目录 {article_path} 的 images/ 子目录
2. 把文章中的 {{IMAGE_COVER}} {{IMAGE_1}} 等占位符替换为 ![](images/filename.png)
3. 执行 Stage 6 发布到微信公众号
4. 发布完成后执行 Stage 7 清理（mv 到 Trash）
```

---

## Iron Rules

| # | Rule | Why |
|---|------|-----|
| 1 | **Bot is dispatcher** | Articles are written by CC, not the bot |
| 2 | **Confirm angle first** | Step 1.5 — wait for user mobile confirmation |
| 3 | **Auto-rotate styles + nano** | Style rotation and nano prompt enhancement are bundled — always both, never skip nano |
| 4 | **Check login before publish** | No login = stop immediately, save quota |
| 5 | **No auto-retry on failure** | Tell user, let them decide |
| 6 | **Always set callbackContainer** | Prevents callbacks going to wrong bot |
| 7 | **Preserve sessionId** | Step 3 must carry Step 1's sessionId |
| 8 | **Articles on Desktop** | `~/Desktop/bot-articles/` — user's phone can access |
