---
name: sfwapi
description: Integrate the SFW API (api.sfwapi.dev), a nudity-detection REST API that returns a content level 0-3, art and adult-object flags and a policy verdict for images, GIFs and videos. Use when the user asks for nudity or NSFW detection, an NSFW filter, moderating image or video uploads, filtering AI-generated images, or mentions SFW API or sfwapi.
allowed-tools: WebFetch
---

# SFW API integration

The SFW API scores media on four nudity levels (0 Everyday, 1 Revealing, 2 Implied nudity, 3 Explicit),
adds `art` and `sex_object` flags, and answers `policy.blocked: true | false` against a `block_level` the
caller sends. One request, one answer, nothing stored.

## Before writing code

1. Fetch https://sfwapi.dev/llms-full.txt (the whole documentation in one Markdown file).
2. Fetch https://sfwapi.dev/openapi.json when you need exact field names or error shapes.
3. If either fetch fails, use `reference.md` next to this file; it is an offline copy generated from the
   same sources and may lag the live docs.

## Integration recipe

1. **Key.** Read it from the environment variable `SFWAPI_API_KEY` on the server. Never put it in a
   browser bundle, a mobile app or a repository. If the project has no key yet, tell the user to create
   one at https://sfwapi.dev/console (500 free credits a month, no card).
2. **Client.** Prefer the official SDK: `npm install @sfwapi/sdk` (`SfwApi.analyzeUrl / analyzeFile /
   analyzeBase64`, `evaluatePolicy`) or `pip install sfwapi` (`SfwApi` / `AsyncSfwApi` with `analyze_url /
   analyze_bytes / analyze_file`). Otherwise call `POST https://api.sfwapi.dev/v1/analyze` with
   `Authorization: Bearer <key>` and exactly one of `url`, `base64`, or a multipart `file`.
3. **Policy.** Send `policy: { block_level: 2 }` by default and make the level configurable (1 strict,
   2 default, 3 only explicit). Act on `policy.blocked`; do not re-derive thresholds from the scores.
   Scores are 0-100 and a level counts above 50.
4. **Errors.** 4xx (400 invalid request, 401 bad key, 402 no credits, 413 too large, 415 unsupported, 422
   could not download or decode): do not retry, surface the `error.code` and `error.request_id`. 429 and
   503: retry after the `Retry-After` header. 504: treat as unknown, do not charge the user twice. Failed
   requests are never charged.
5. **Moderators.** Keep `nudity.level`, `nudity.scores`, `flags` and `id` with the moderation decision so a
   human can review why something was blocked. Show the level name, not only the number.
6. **Privacy.** Never log or persist the media, its URL, hashes or filenames. The API itself stores none of
   them; keep the integration to the same standard.

## Media kinds

Images, GIFs and videos are all `POST /v1/analyze`. GIF and video frames are sampled (`max_frames`,
default 5 for GIF and 10 for video); each analysed frame costs one credit and `media.frames_analyzed`
tells you how many were used. Add `include_frames: true` only when the caller needs per-frame scores.

## Do not

- Do not claim the API detects child sexual abuse material or estimates age. It does neither.
- Do not call the API from client-side code.
- Do not build a thresholding scheme of your own when `policy` already expresses it.
