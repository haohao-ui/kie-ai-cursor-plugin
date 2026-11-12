---
name: kie-generate
description: Generate or edit images, video, and music via kie.ai Market APIs. Use when the user wants to create, edit, upload references for, or poll kie.ai generation tasks (GPT Image, Kling, Seedance, Veo, Suno, and other Market models).
---

# kie.ai generation

Use the `kie-ai` MCP tools to talk to [kie.ai](https://docs.kie.ai/cn/market/quickstart). All Market image, video, and current Suno/Veo generation tasks share one task API.

## When to use

- Generate or edit images, video, or music through kie.ai
- Check a previous task, wait for it to finish, or read credit balance
- Turn a local file into a public URL before passing `input_urls` / `image_urls`

## Flow

1. If the model needs a public image/video/audio URL and the user only has a local file, call `kie_upload_file` first. Uploads are free; files are deleted after 24 hours.
2. Optionally call `kie_get_credits` to confirm the account has balance. Generations spend credits.
3. Call `kie_create_task` with a verified `model` id and that model's `input` object. It returns `{ taskId }`.
4. Call `kie_wait_task(taskId, timeoutSec)` (default 600s; use a longer timeout for video, which can exceed 10 minutes). Or poll `kie_get_task`.
5. On `state: success`, use `resultUrls`. On `state: fail`, report `failMsg`. Intermediate states are `waiting`, `queuing`, and `generating`.

Do not invent model ids or input fields. If a field is not in the official docs for that model, do not send it.

## Safety

- **Video: always get the user's explicit OK before calling `kie_create_task` for any video model.** Video is slower and usually costs more credits.
- Tell the user that a create call spends credits. Do not retry a failed create blindly.
- Never print or commit `KIE_API_KEY`.

## Verified model ids

Only the ids below were checked against official docs. Many other Market models also use `POST /api/v1/jobs/createTask` — look up their docs before using them.

| Kind | Model id | Endpoint | Required `input` | Docs |
| --- | --- | --- | --- | --- |
| Image | `gpt-image-2-6/flare-text-to-image` | `POST /api/v1/jobs/createTask` | `prompt` | https://docs.kie.ai/cn/market/gpt/gpt-image-2-6-flare-text-to-image |
| Image | `gpt-image-2-6/flare-image-to-image` | `POST /api/v1/jobs/createTask` | `prompt`, `input_urls` (max 16 public URLs) | https://docs.kie.ai/cn/market/gpt/gpt-image-2-6-flare-image-to-image |
| Image | `grok-imagine/text-to-image` | `POST /api/v1/jobs/createTask` | `prompt`, `aspect_ratio` | https://docs.kie.ai/cn/market/grok-imagine/text-to-image |
| Video | `kling-3.0/video` | `POST /api/v1/jobs/createTask` | `prompt` (when `multi_shots` is false), `duration` (3–15 as a string), `sound`, `mode` (`std`/`pro`/`4K`), `multi_shots` | https://docs.kie.ai/cn/market/kling/kling-3-0 |
| Video | `kling/v3-turbo-text-to-video` | `POST /api/v1/jobs/createTask` | `prompt`, `duration`, `aspect_ratio`, `resolution` | https://docs.kie.ai/cn/market/kling/v3-turbo-text-to-video |
| Video | `bytedance/seedance-2-5` | `POST /api/v1/jobs/createTask` | `input` required; `prompt` for text-to-video (duration default 5, or `-1` for auto) | https://docs.kie.ai/cn/market/bytedance/seedance-2-5 |
| Video | `bytedance/seedance-2` | `POST /api/v1/jobs/createTask` | `input` required; typically `prompt` | https://docs.kie.ai/cn/market/bytedance/seedance-2 |
| Video | `veo-3-1` | `POST /api/v1/jobs/createTask` | `prompt`; optional `input.model` `veo3_fast` (default) / `veo3_lite` / `veo3` | https://docs.kie.ai/cn/veo3-api/generate-veo-3-video |
| Music | `ai-music-api/generate` | `POST /api/v1/jobs/createTask` | `custom_mode`, `instrumental`, `model` (`V6` / `V6_MINI` / `V6_WILD`) | https://docs.kie.ai/cn/suno-api/generate-music |
| Music | `ai-music-api/extend` | `POST /api/v1/jobs/createTask` | `audio_id`, `model` | https://docs.kie.ai/cn/suno-api/extend-music |

Current official docs for Veo 3.1 and Suno music generation use the unified `createTask` + `GET /api/v1/jobs/recordInfo` flow. Legacy dedicated paths (`/api/v1/veo/...`, `/api/v1/generate`) remain only under old-model docs and are not wrapped by this plugin.

Suno success results may appear as `data.response.data[]` with `audio_url` / `stream_audio_url` / `image_url`. The `kie_get_task` tool folds those into `resultUrls` when present. Other Market models typically put URLs in `resultJson.resultUrls`. Result files are kept for 14 days. `createTime` / `completeTime` are Unix timestamps in milliseconds.

## Auth and errors

- Base URL: `https://api.kie.ai`. Header: `Authorization: Bearer <KIE_API_KEY>`.
- Upload base URL: `https://kieai.redpandaai.co`.
- Credits: `GET /api/v1/chat/credit`.
- HTTP 200 can still be a business error — tools map `code` 401 / 402 / 422 / 429 / 433 / 455 / 500 / 501 / 505 to a readable message.
