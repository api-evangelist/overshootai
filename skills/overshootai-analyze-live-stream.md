---
name: Analyze a live video stream with Overshoot
description: Create a stream, keep it alive while publishing frames, query a vision-language model about the video, and tear the stream down.
api: openapi/overshootai-openapi.yaml
operations: [createStream, keepaliveStream, chatCompletions, getStream, deleteStream]
---

# Analyze a live video stream

Use Overshoot to ask a vision-language model questions about live video in real time.
The API is OpenAI-compatible and authenticated with a bearer API key.

## Prerequisites
- An API key from https://platform.overshoot.ai/api-keys, sent as `Authorization: Bearer ovs_...` on every call except `GET /models` and `GET /billing/pricing`.
- Base URL: `https://api.overshoot.ai/v1beta`.
- A prepaid balance (inference returns `402` when billing denies a request — check with `getBalance`).

## Steps
1. **Create the stream** — `createStream` (`POST /streams`). The request body is empty; you receive a stream id and LiveKit `PublishInfo` (room + publish token). Frame retention is fixed at 600 seconds.
2. **Publish frames** — connect a LiveKit publisher (browser webcam, screen share, native app, or server feed) using the returned token. The publish token is for media only; it does NOT replace the API key for HTTP calls.
3. **Keep the lease alive** — call `keepaliveStream` (`POST /streams/{stream_id}/keepalive`) every 10–20 seconds; streams expire after ~45 seconds without a keepalive. Each keepalive pays for elapsed streaming time and returns a fresh token.
4. **Query the video** — call `chatCompletions` (`POST /chat/completions`) with a `model` from `listModels` and messages that reference frames via `ovs://` URLs (a single frame, a window, or several windows). Set `stream: true` for continuously streamed results (latency as low as 200ms).
5. **Inspect state if needed** — `getStream` (`GET /streams/{stream_id}`) reports lifecycle state and frame availability.
6. **Tear down** — `deleteStream` (`DELETE /streams/{stream_id}`) releases resources; it is idempotent on already-deleted streams within the lookup window.

## Error handling
- `401` missing/invalid key; `403` resource belongs to another user.
- `402` billing denied — top up via checkout.
- `409` wrong region (`RegionError` gives `expected_region`) or frame not yet arrived; retry against the expected region.
- `410` requested frame index evicted (outside the 600s window).
- `429` rate limited — honor `retry-after` and `x-ratelimit-remaining`.
See `errors/overshootai-problem-types.yml` and `conventions/overshootai-conventions.yml`.
