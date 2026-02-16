# OpenClaw Content Alchemy

> Turn your OpenClaw bot into a content publishing pipeline — from idea to WeChat article, fully automated.
>
> 让你的 OpenClaw bot 变成内容发布流水线——从想法到微信公众号文章，全自动。

## What Is This?

A ready-to-use configuration kit for [OpenClaw](https://github.com/openclaw/openclaw) bots that want to write, illustrate, and publish articles — powered by Claude Code as the local execution engine.

> 一套开箱即用的 OpenClaw bot 配置方案，让 bot 能写文章、配图、发布到微信公众号。底层通过本地 Claude Code 执行实际任务。

## Architecture

```
User (Discord) → OpenClaw Bot → Claude Code (local) → WeChat
                     │
                     ├── Step 1: CC writes article (Content Alchemy Stages 1-5)
                     ├── Step 1.5: User confirms angle/framework (mobile-friendly)
                     ├── Step 2: Bot generates illustrations (auto style rotation)
                     ├── Step 3: CC embeds images + publishes to WeChat
                     └── Login check before publish (abort if not logged in)
```

> 用户通过 Discord 发指令 → Bot 调度 → CC 本地写稿 → 用户手机确认 → Bot 配图 → CC 嵌图发布

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

## Prerequisites

| Component | Purpose | Link |
|-----------|---------|------|
| OpenClaw bot | Discord bot framework | [openclaw](https://github.com/openclaw/openclaw) |
| openclaw-worker | Task API bridge (CC ↔ Bot) | [openclaw-worker](https://github.com/AliceLJY/openclaw-worker) |
| openclaw-cc-pipeline | Multi-turn CC orchestration skill (Bot → API → Worker → CC → Callback) | [openclaw-cc-pipeline](https://github.com/AliceLJY/openclaw-cc-pipeline) |
| Content Alchemy | Article writing skill for Claude Code | [content-alchemy](https://github.com/AliceLJY/content-alchemy) |
| Chrome + CDP | WeChat publishing automation | Port 9222 debug mode |

## Quick Start

1. Set up [openclaw-worker](https://github.com/AliceLJY/openclaw-worker) with Task API
2. Copy `style-catalog.md` to your bot's workspace: `~/.openclaw-<botname>/workspace/images/`
3. Add the workflow from `bot-workflow.md` to your bot's `MEMORY.md`
4. Tell your bot: "写一篇关于 [topic] 的文章"

> 1. 部署 openclaw-worker + Task API
> 2. 把 style-catalog.md 复制到 bot workspace
> 3. 把 bot-workflow.md 的流程写入 bot 的 MEMORY.md
> 4. 对 bot 说："写一篇关于 [话题] 的文章"

## License

MIT

---

*Built with [Content Alchemy](https://github.com/AliceLJY/content-alchemy) and battle-tested by AntiBot.*
