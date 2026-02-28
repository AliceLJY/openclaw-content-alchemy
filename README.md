# OpenClaw Content Alchemy

> Turn your OpenClaw bot into a content publishing pipeline — from idea to WeChat article, fully automated.
>
> 让你的 OpenClaw bot 变成内容发布流水线——从想法到微信公众号文章，全自动。

## What Is This?

A ready-to-use configuration kit for [OpenClaw](https://github.com/openclaw/openclaw) bots that want to write, illustrate, and publish articles. Supports three publishing modes — from full Claude Code integration to **fully CC-Free** operation.

> 一套开箱即用的 OpenClaw bot 配置方案，让 bot 能写文章、配图、发布到微信公众号。支持三种发布模式——从完整 CC 集成到**完全不依赖 CC**。

## Architecture — Three Publishing Modes

### Mode A: Full CC (Bot + Claude Code)

```
User → Bot → CC writes article → User confirms → Bot generates images → CC embeds + publishes
```

> Bot 调度 CC 写稿 → 用户手机确认 → Bot 配图 → CC 嵌图发布

### Mode B: Hybrid (Bot + CC for writing, Worker for publishing)

```
User → Bot → CC writes article → User confirms → Bot generates images → Worker publishes
                                                                          (shell commands,
                                                                           no CC needed)
```

> CC 只负责写稿，发布环节直接走 Worker shell 命令，不启动 CC

### Mode C: CC-Free (Bot + Worker only, no Claude Code at all)

```
User → Bot writes article → User confirms → Bot generates images → Worker publishes
         (bot's own LLM)                      (Gemini API)           (shell commands)
```

> 全程不需要 CC。Bot 自己的模型写稿，Gemini 生图，Worker 跑 shell 命令发布。适合没有 Claude Code 的用户。

### CC-Free Publishing — How It Works

Instead of dispatching to Claude Code, the bot calls the Worker's `/tasks` endpoint to run shell commands directly:

> 不启动 CC，bot 直接调 Worker 的 `/tasks` 端点执行 shell 命令：

```
# Step 1: Copy images + replace placeholders (sed)
POST /tasks → { "command": "cp images && sed -i '' 's|{{IMAGE_COVER}}|![](images/IMAGE_COVER.png)|g' article.md" }

# Step 2: Run publish script
POST /tasks → { "command": "bun content-publisher/scripts/publish-wechat.ts --markdown article.md --theme grace" }

# Step 3: Cleanup
POST /tasks → { "command": "mv article-dir ~/.Trash/" }
```

**Key difference**: `/tasks` has no callback — bot must poll `GET /tasks/{taskId}?wait=120000` for results.

> **关键区别**：`/tasks` 没有回调机制，bot 需要轮询 GET 拿结果。

### Image Generation — Two Modes

| Bot Model | Image Gen Mode | How |
|-----------|---------------|-----|
| Gemini-based (e.g. Gemini Pro 3) | **Direct** — bot calls the script itself | `exec` → `generate-image.mjs` → Gemini Image API |
| Non-Gemini (e.g. Claude Opus) | **Dispatched** — bot delegates to the script as a sub-task | Bot → `generate-image.mjs` → Gemini Image API |

> If your bot already runs on Gemini, it can generate images directly without dispatching to another agent — same API, zero overhead.
>
> 如果你的 bot 本身就跑在 Gemini 上，生图不需要转发给子 agent——同一个 API，零额外开销。

## Style Catalog — 56 Art Styles

The crown jewel: **56 illustration styles** spanning art history, from Song Dynasty gongbi to Banksy street art. Auto-rotation ensures every article looks different.

> 核心亮点：**56种配图风格**，横跨美术史，从宋式工笔到班克斯街头艺术。自动轮换保证每篇文章风格不重复。

### Style Categories

| Category | Count | Examples |
|----------|-------|---------|
| Modern & Trendy | 5 | Cyberpunk, Pixel Art, Pop Art Collage |
| Craft & Texture | 5 | Claymation, Felt Craft, Chalkboard |
| Chinese Art | 9 | Freehand Ink, Blue-Green Landscape, Qi Baishi, Wu Guanzhong |
| Japanese Aesthetics | 2 | Ukiyo-e (Hokusai), Yoshitomo Nara |
| Renaissance & Classical | 4 | Da Vinci, Botticelli, Raphael, Dürer |
| Baroque & Rococo | 3 | Caravaggio, Rembrandt, Vermeer |
| Romanticism | 3 | Turner, Delacroix, Friedrich |
| Impressionism & Post | 6 | Monet, Van Gogh, Seurat, Cézanne, Gauguin |
| Modernism | 11 | Klimt, Picasso, Matisse, Dalí, Magritte, Frida Kahlo |
| Contemporary | 8 | Hopper, Warhol, Basquiat, Kusama, Banksy |

Plus **style combination rules** — pair any two styles for unique fusion effects (e.g., Cyberpunk + Ukiyo-e, Dunhuang Mural + Neon).

> 还支持**风格组合**——任意两种风格混搭（比如赛博朋克×浮世绘、敦煌壁画×霓虹），效果炸裂。

See [style-catalog.md](style-catalog.md) for the full list with prompt suffixes.

## Files

| File | Purpose |
|------|---------|
| [style-catalog.md](style-catalog.md) | 56 art styles with English prompt suffixes and mood matching |
| [bot-workflow.md](bot-workflow.md) | Complete bot article workflow (Step 1 → 1.5 → 2 → 3) |
| [pipeline-reference.md](pipeline-reference.md) | Technical references to related repos |

## Author's Setup

> 作者的开发环境，仅供参考，你可以用自己喜欢的工具替代

| Item | Setup |
|------|-------|
| **Hardware** | MacBook Air M4, 16GB RAM |
| **Models** | Claude Opus 4.6 (primary), Gemini Pro 3 (secondary), MiniMax M2.5 (scheduled tasks) |
| **Runtime** | Bun, Docker |
| **API** | [OpenClaw](https://github.com/openclaw/openclaw) subscription |

> Author's setup — yours may differ.

### Platform Notes

> **macOS-specific**: WeChat publishing uses `osascript` for clipboard operations (Cmd+C/V). On Linux/Windows, this part needs a platform-specific clipboard tool (e.g., `xclip`, `powershell`).
>
> **macOS 专属**：微信发布用 `osascript` 操作剪贴板。Linux/Windows 需要替换为对应的剪贴板工具。

> **Chrome LaunchAgent**: We use a macOS LaunchAgent to keep a headless Chrome running on port 9222. On Linux, use `systemd` service; on Windows, use Task Scheduler.
>
> **Chrome 常驻**：macOS 用 LaunchAgent 保持 Chrome 调试端口常开。Linux 用 systemd，Windows 用任务计划程序。

## Prerequisites

| Component | Purpose | Link |
|-----------|---------|------|
| OpenClaw bot | Discord bot framework | [openclaw](https://github.com/openclaw/openclaw) |
| openclaw-worker | Task API bridge (CC ↔ Bot) | [openclaw-worker](https://github.com/AliceLJY/openclaw-worker) |
| openclaw-cc-pipeline | Multi-turn CC orchestration skill (Mode A/B only) | [openclaw-cc-pipeline](https://github.com/AliceLJY/openclaw-cc-pipeline) |
| Content Alchemy | Article writing skill for Claude Code, 5-stage pipeline v5.0 (Mode A/B only) | [content-alchemy](https://github.com/AliceLJY/content-alchemy) |
| Content Publisher | Image generation + layout + WeChat API publishing (all modes) | [content-publisher](https://github.com/AliceLJY/content-publisher) |
| Chrome + CDP | Gemini image gen fallback + browser-mode publishing (optional) | Port 9222 debug mode |

## Quick Start

1. Set up [openclaw-worker](https://github.com/AliceLJY/openclaw-worker) with Task API
2. Copy `style-catalog.md` to your bot's workspace: `~/.openclaw-<botname>/workspace/images/`
3. Add the workflow from `bot-workflow.md` to your bot's `MEMORY.md`
4. Tell your bot: "写一篇关于 [topic] 的文章"

> 1. 部署 openclaw-worker + Task API
> 2. 把 style-catalog.md 复制到 bot workspace
> 3. 把 bot-workflow.md 的流程写入 bot 的 MEMORY.md
> 4. 对 bot 说："写一篇关于 [话题] 的文章"

## Origin Story

> This project was born from real production pain. After weeks of manually choosing illustration styles for a WeChat Official Account, it became clear that style selection should be automated — not random, but informed by art history and matched to article tone. Combined with the OpenClaw bot orchestration pipeline, it became a fully automated content publishing system.
>
> 这个项目来自实战踩坑。公众号连续写了几十篇文章后，每次手选配图风格实在太累，索性把美术史梳理成机器可读的风格目录，再结合 OpenClaw bot 编排管道，做成了全自动内容发布系统。

**Original work by [AliceLJY](https://github.com/AliceLJY)**:
- 56-style catalog with art-historical classification and AI prompt engineering
- Auto-rotation algorithm with tone matching and history-based deduplication
- Bot article workflow with mobile-friendly confirmation checkpoints
- CC-Free publishing mode — run the full pipeline without Claude Code
- WeChat login pre-check to avoid wasting API quota (CC mode)

> All style prompt suffixes are hand-crafted based on art history knowledge, not copy-pasted from generic prompt databases. The combination rules are tested with Gemini image generation.

## Ecosystem

> 这些项目配合使用效果更好

| Project | What It Does |
|---------|-------------|
| [content-alchemy](https://github.com/AliceLJY/content-alchemy) | 5-stage writing pipeline skill (v5.0) — from topic mining to polished article |
| [content-publisher](https://github.com/AliceLJY/content-publisher) | Image generation, layout formatting, and WeChat API publishing |
| [openclaw-cc-bridge](https://github.com/AliceLJY/openclaw-cc-bridge) | Discord commands → Claude Code bridge (zero agent tokens) |
| [digital-clone-skill](https://github.com/AliceLJY/digital-clone-skill) | Extract writing DNA to personalize your content voice |

## Author

Built by **小试AI** ([@AliceLJY](https://github.com/AliceLJY)) · WeChat: **我的AI小木屋**

> 医学出身，文化口工作，AI 野路子。公众号六大板块：AI实操手账 · AI踩坑实录 · AI照见众生 · AI冷眼旁观 · AI胡思乱想 · AI视觉笔记

Six content pillars: **Hands-on AI** · **AI Pitfall Diaries** · **AI & Humanity** · **AI Cold Eye** · **AI Musings** · **AI Visual Notes**

Open-source byproducts: [content-alchemy](https://github.com/AliceLJY/content-alchemy) · [content-publisher](https://github.com/AliceLJY/content-publisher) · [openclaw-worker](https://github.com/AliceLJY/openclaw-worker) · [openclaw-cc-pipeline](https://github.com/AliceLJY/openclaw-cc-pipeline) · [openclaw-content-alchemy](https://github.com/AliceLJY/openclaw-content-alchemy) · [openclaw-cc-bridge](https://github.com/AliceLJY/openclaw-cc-bridge) · [digital-clone-skill](https://github.com/AliceLJY/digital-clone-skill)

<img src="./assets/wechat_qr.jpg" width="200" alt="WeChat QR Code">

## License

[MIT](LICENSE)
