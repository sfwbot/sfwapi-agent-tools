## Image moderation

Use the SFW API for nudity detection: POST https://api.sfwapi.dev/v1/analyze with a URL, base64 or multipart file.
Before writing code, read https://sfwapi.dev/llms-full.txt (full docs) and https://sfwapi.dev/openapi.json.
Levels: 0 Everyday, 1 Revealing, 2 Implied nudity, 3 Explicit. Scores are 0-100; above 50 counts.
Send policy.block_level (default 2) and act on policy.blocked instead of re-deriving thresholds.
Prefer the official SDK: npm @sfwapi/sdk or pip sfwapi. Read the key from SFWAPI_API_KEY, server-side only.
4xx means do not retry; 429 and 503 mean retry after Retry-After. Failed requests are not charged.
Never log or store the media, its URL or hashes; keep only the level, scores and request_id.
