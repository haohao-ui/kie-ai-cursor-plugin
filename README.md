# kie-ai

Cursor / Agent Plugin that wraps the [kie.ai](https://docs.kie.ai/cn/market/quickstart) Market API for images, video, and music.

This package is a local plugin (`plugin.json` + `mcp.json` + `skills/`). It is not published.

## What it provides

MCP tools, organized around a single task type rather than one tool per model:

- `kie_create_task(model, input, callBackUrl?)` — create a Market job, returns `taskId` (spends credits)
- `kie_get_task(taskId)` — normalized `KieTask`
- `kie_wait_task(taskId, timeoutSec)` — poll with 2–5s backoff until success, fail, or timeout
- `kie_get_credits()` — read-only credit balance
- `kie_upload_file` — upload a local file, URL, or base64 payload and get a public `downloadUrl` (free; deleted after 24 hours)

`KieTask` shape: `{ taskId, model, state, resultUrls[], failMsg?, costTime?, createTime }` plus documented extras when present (`failCode`, `completeTime`, `creditsConsumed`, `progress`).

## Setup

1. Get an API key at https://kie.ai/api-key
2. Install this folder as a local Agent Plugin, or copy it to `~/.cursor/plugins/local/kie-ai` for Cursor IDE.
3. Set the secret plugin variable `KIE_API_KEY` (or export it in the environment when running the server by hand). Never put the real key in `mcp.json` or this repo.

The stdio server is zero-dependency Node.js >= 18 (`server/index.mjs`).

Manual run:

```bash
KIE_API_KEY=your-key node ./server/index.mjs
```

## Notes

- Official Agent Plugins `plugin.json` cannot declare user secrets (closed schema). This repo keeps a valid root `plugin.json`, and puts the `KIE_API_KEY` variable in `.cursor-plugin/plugin.json` for Cursor.
- Cursor expands `${CURSOR_PLUGIN_ROOT}` in `mcp.json`, not `${PLUGIN_ROOT}`. This plugin omits `cwd` (clients default to the plugin root) and uses the relative arg `./server/index.mjs`.
- Current Veo 3.1 and Suno generate/extend docs use `POST /api/v1/jobs/createTask`. This plugin does not wrap the legacy dedicated endpoints.
- Always get an explicit OK from the user before starting video generation. Creates spend credits.

## Docs

- Market quickstart: https://docs.kie.ai/cn/market/quickstart
- Get task detail: https://docs.kie.ai/cn/market/common/get-task-detail
- Credits: https://docs.kie.ai/cn/common-api/get-account-credits
- File upload: https://docs.kie.ai/cn/file-upload-api/quickstart
