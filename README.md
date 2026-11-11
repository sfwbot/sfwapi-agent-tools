# sfwapi-agent-tools

Plugin and rule files for using the [SFW API](https://sfwapi.dev) (nudity detection for images) from Claude Code,
Codex and Cursor.

## Claude Code

```bash
claude plugin marketplace add sfwbot/sfwapi-agent-tools
claude plugin install sfwapi@sfwapi-agent-tools
```

Inside a session: `/plugin marketplace add sfwbot/sfwapi-agent-tools` then
`/plugin install sfwapi@sfwapi-agent-tools`. The plugin ships one skill, `sfwapi`, that activates when you ask
for nudity detection, an NSFW filter or upload moderation. Third-party marketplaces do not auto-update by
default; turn it on under `/plugin` if you want new versions.

## Codex

Copy the `## Image moderation` block from [`AGENTS.md`](AGENTS.md) into your repository's `AGENTS.md`.

## Cursor

Copy [`.cursor/rules/sfwapi.mdc`](.cursor/rules/sfwapi.mdc) into your project. Cursor also reads `AGENTS.md`
and `.claude/skills/`, so either of the files above works too.

## Anything else

Paste this prompt into your agent:

> Add nudity moderation to image uploads using the SFW API. Read https://sfwapi.dev/llms-full.txt first. Use
> the official SDK if one exists for this language (npm `@sfwapi/sdk`, pip `sfwapi`), otherwise POST to
> https://api.sfwapi.dev/v1/analyze. Block at level 2 and above by default and make the level configurable.
> Keep the API key in the environment variable `SFWAPI_API_KEY` on the server and never send it to the
> browser. Retry only on 429 and 503, using the Retry-After header.

## Docs for agents

| Resource | URL |
| --- | --- |
| Index for agents | https://sfwapi.dev/llms.txt |
| Every docs page in one file | https://sfwapi.dev/llms-full.txt |
| Any docs page as Markdown | `https://sfwapi.dev/md/<page>`, for example https://sfwapi.dev/md/quickstart |
| OpenAPI 3.1 | https://sfwapi.dev/openapi.json |

## Layout

```
.claude-plugin/plugin.json       plugin manifest
.claude-plugin/marketplace.json  marketplace manifest (the plugin lives at the repo root)
skills/sfwapi/SKILL.md           the skill
skills/sfwapi/reference.md       offline API reference, generated
AGENTS.md                        Codex / Cursor recipe, generated
.cursor/rules/sfwapi.mdc         Cursor project rule, generated
```

The generated files are built from the docs at sfwapi.dev, so edit them there rather than here.

## License

MIT
