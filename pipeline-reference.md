# Pipeline Technical Reference

> Pointers to the infrastructure components. This repo focuses on the content workflow; the plumbing lives elsewhere.
>
> 基础设施组件索引。本仓库专注内容流程，技术管道在其他仓库。

## Related Repositories

| Repo | Purpose | Link |
|------|---------|------|
| **content-alchemy** | Claude Code skill for article writing (7-stage pipeline) | [GitHub](https://github.com/AliceLJY/content-alchemy) |
| **openclaw-worker** | Task API bridge between Discord bot and local Claude Code | [GitHub](https://github.com/AliceLJY/openclaw-worker) |
| **openclaw-cc-pipeline** | Multi-turn Claude Code orchestration skill — the core pattern: Bot dispatches → Task API relays → Worker runs → CC executes → Callback delivers result. Install as a Claude Code skill for session persistence and multi-round workflows | [GitHub](https://github.com/AliceLJY/openclaw-cc-pipeline) |

## Key Infrastructure

### Task API (openclaw-worker)

- Runs on `host.docker.internal:3456` inside Docker network
- `POST /claude` — send prompt to Claude Code
- `GET /tasks/:id?wait=120000` — poll for result
- Callback mechanism: `docker exec <container> node openclaw.mjs message send`
- `callbackContainer` per-request override for multi-bot setups

> See [openclaw-worker/server.js](https://github.com/AliceLJY/openclaw-worker/blob/main/server.js) for implementation.

### Chrome Debug Instance

- LaunchAgent: `~/Library/LaunchAgents/com.chrome.debug.plist`
- Flags: `--remote-debugging-port=9222 --user-data-dir=~/chrome-debug-profile --no-startup-window`
- Used for: WeChat publishing (CDP automation) and Gemini image generation
- Must keep WeChat logged in to avoid QR scan on each publish

### Image Generation

- Script: `workspace/scripts/generate-image.mjs`
- Wraps Gemini image generation API
- Fallback: CDP browser automation if API fails
- Output: PNG files to `workspace/images/{slug}/`

### WeChat Publishing

- Script: `baoyu-post-to-wechat` (via content-alchemy dependencies)
- Connects to Chrome CDP port 9222
- Converts Markdown → styled HTML → clipboard paste
- Image placeholders: `![](images/file.png)` → `WECHATIMGPH_N` → actual images
- Requires visible Chrome window (uses `osascript` for clipboard on macOS)
