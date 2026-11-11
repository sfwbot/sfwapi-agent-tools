# SFW API reference (offline copy)

Generated from https://sfwapi.dev/openapi.json by the sfwapi-site repo. Prefer the live docs when you can fetch them:
https://sfwapi.dev/llms-full.txt (everything) and https://sfwapi.dev/openapi.json (schemas).

Base URL: `https://api.sfwapi.dev/v1`. Auth: `Authorization: Bearer <key>`; keys start with `sk_live_` and come from https://sfwapi.dev/console.

## Levels and flags

- Level 0 Everyday: Regular, fully clothed content. Always allowed.
- Level 1 Revealing: Swimwear, lingerie and outfits that show more skin.
- Level 2 Implied Nudity: Unclothed, with intimate areas covered or out of view.
- Level 3 Explicit: Visible intimate areas.

Scores are 0-100 per level; a level counts when its score is above 50. The response's `nudity.level` is the highest level that counts. Flags: `art` (Drawing, anime, painting, statue, stylised 3D), `sex_object` (Adult toy or genital-shaped object).

## Credits

500 free credits per month. Prepaid packs: $10 = 10,000, $40 = 50,000, $150 = 250,000, $500 = 1,000,000. One credit per image or analysed GIF/video frame; failed requests cost nothing.

## POST /v1/analyze

Analyze an image, GIF or video

Send media by URL, base64 or multipart upload. Returns the nudity level (0-3), the three level scores, the art and adult-object flags and, when a policy is given, whether that policy blocks the media. Media is processed in memory and never stored.

- Request (application/json): AnalyzeJsonRequest
- Request (multipart/form-data): AnalyzeMultipartRequest
- 200: Scores → AnalyzeResponse
- 400: Invalid request or URL not allowed → ErrorResponse
- 401: Missing or invalid API key → ErrorResponse
- 402: Insufficient credits → ErrorResponse
- 403: Key revoked or account suspended → ErrorResponse
- 413: Media too large → ErrorResponse
- 415: Unsupported media type → ErrorResponse
- 422: Media could not be downloaded or decoded → ErrorResponse
- 429: Rate limited → ErrorResponse
- 503: Temporarily unavailable or overloaded → ErrorResponse
- 504: Timed out → ErrorResponse

## GET /v1/me

The key, tier, limits and balance behind this request



- No request body
- 200: OK → MeResponse
- 401: Missing or invalid API key → ErrorResponse

## GET /v1/usage

This month's usage and the current balance



- No request body
- 200: OK → UsageResponse
- 401: Missing or invalid API key → ErrorResponse

## GET /v1/status

Public service status



- No request body
- 200: OK → StatusResponse

## Schemas

### AnalyzeResponse

| Field | Type | Meaning |
| --- | --- | --- |
| `id` | string |  |
| `media` | object |  |
| `nudity` | object |  |
| `flags` | Flags |  |
| `policy` | PolicyResult? |  |
| `frames` | Frame[]? |  |
| `usage` | object |  |
| `timing_ms` | object |  |

### MediaKind

"image" | "gif" | "video"

### Level

object: 0 Everyday · 1 Revealing · 2 Implied nudity · 3 Explicit

### LevelScores

| Field | Type | Meaning |
| --- | --- | --- |
| `revealing` | number | Level 1 or higher |
| `implied_nudity` | number | Level 2 or higher |
| `explicit` | number | Level 3 |

### Flags

| Field | Type | Meaning |
| --- | --- | --- |
| `art` | Flag |  |
| `sex_object` | object |  |

### Flag

| Field | Type | Meaning |
| --- | --- | --- |
| `score` | number | 0-100, calibrated so 50 is the decision threshold |
| `detected` | boolean |  |

### PolicyResult

| Field | Type | Meaning |
| --- | --- | --- |
| `blocked` | boolean |  |
| `reason` | "level" | "sex_object" | "art_level" | null | Why it was blocked: a level at or above block_level, an adult object, or a level under the art policy |
| `matched_level` | object |  |

### Frame

| Field | Type | Meaning |
| --- | --- | --- |
| `index` | integer |  |
| `timestamp_ms` | integer,null |  |
| `level` | Level |  |
| `scores` | LevelScores |  |
| `flags` | Flags |  |

### ErrorResponse

| Field | Type | Meaning |
| --- | --- | --- |
| `error` | object |  |

### AnalyzeJsonRequest

| Field | Type | Meaning |
| --- | --- | --- |
| `url` | string? | Public http(s) URL of an image, GIF or video. Fetched by our servers; never stored. |
| `base64` | string? | Raw media bytes, base64 encoded (prefer multipart upload: base64 costs 33% more bandwidth) |
| `max_frames` | integer? | Frames sampled evenly over a GIF or video (first and last always included). Default 5 for GIF, 10 for video. One credit per analysed frame. |
| `include_frames` | boolean? | Return per-frame scores for GIF/video |
| `policy` | Policy? |  |

### Policy

| Field | Type | Meaning |
| --- | --- | --- |
| `block_level` | object | Block media at this level or above (1 Revealing, 2 Implied nudity, 3 Explicit) |
| `block_sex_objects` | boolean? | Also block adult toys and objects |
| `art` | ArtPolicy? |  |

### ArtPolicy

"same_as_photos" | "allow" | "block_from_1" | "block_from_2" | "block_from_3": How drawn/anime/stylised content is treated: like photos, never blocked, or blocked from a different level

### AnalyzeMultipartRequest

| Field | Type | Meaning |
| --- | --- | --- |
| `file` | string | The image, GIF or video |
| `max_frames` | string? | See max_frames |
| `include_frames` | string? |  |
| `policy` | string? | JSON-encoded Policy object |

### MeResponse

| Field | Type | Meaning |
| --- | --- | --- |
| `account_id` | string |  |
| `key_prefix` | string |  |
| `tier` | Tier |  |
| `limits` | Limits |  |
| `balance` | Balance |  |

### Tier

"free" | "paid" | "pro" | "enterprise"

### Limits

| Field | Type | Meaning |
| --- | --- | --- |
| `requests_per_second` | number |  |
| `burst` | integer |  |
| `max_inflight` | integer |  |
| `max_media_bytes` | integer | Largest accepted upload / base64 / URL image |
| `video_allowed` | boolean |  |
| `bytes_per_day` | integer |  |

### Balance

| Field | Type | Meaning |
| --- | --- | --- |
| `free` | integer |  |
| `paid` | integer |  |
| `total` | integer |  |
| `free_resets_at` | string |  |

### UsageResponse

| Field | Type | Meaning |
| --- | --- | --- |
| `month` | object |  |
| `balance` | Balance |  |

### StatusResponse

| Field | Type | Meaning |
| --- | --- | --- |
| `status` | "ok" | "degraded" | "down" |  |
| `replicas_ready` | integer |  |
| `replicas_total` | integer |  |

## Prompt that works in any agent

Add nudity moderation to image uploads using the SFW API. Read https://sfwapi.dev/llms-full.txt first. Use the official SDK if one exists for this language (npm @sfwapi/sdk, pip sfwapi), otherwise POST to https://api.sfwapi.dev/v1/analyze. Block at level 2 and above by default and make the level configurable. Keep the API key in the environment variable SFWAPI_API_KEY on the server and never send it to the browser. Retry only on 429 and 503, using the Retry-After header.
