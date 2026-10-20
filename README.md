# ProLevelMax MCP

An MCP server for Reddit, TikTok, Instagram, X, YouTube, Xiaohongshu (RedNote) and Douyin data. ProLevelMax gives Claude, ChatGPT, Cursor and other AI agents real-time public posts, full comment threads, profiles and trends from 157 platforms, through a remote MCP server, an Agent Skill or a REST API. Pay per use, no subscription.

## What it is

[ProLevelMax](https://prolevelmax.com) gives AI agents and developers live public data from social media and the web: posts, comments, profiles and engagement data from 157 platforms, including Reddit, TikTok, Instagram, X, YouTube, Xiaohongshu and Douyin. You connect it once and your AI can search for a data source, get a price quote, run it and read the results.

There are three ways in:

- **Remote MCP server** at `https://prolevelmax.com/mcp` (Streamable HTTP, OAuth 2.1 or API key)
- **Agent Skill** at `https://prolevelmax.com/SKILL.md` for AI tools that can run commands or send HTTP requests
- **REST API** at `https://prolevelmax.com/v1` ([OpenAPI 3.1](https://prolevelmax.com/openapi.json))

This repository holds documentation and example client configs only. It contains no server source code. The server is hosted by ProLevelMax; there is nothing to install or run locally.

## Quickstart (Skill)

1. Sign in at [prolevelmax.com](https://prolevelmax.com) and create an API key in the [dashboard](https://prolevelmax.com/dashboard) under **API keys**. Keys look like `sk_procli_...` and are shown once.
2. Send this line to your AI (Claude, Claude Code, Codex, Cursor and others) together with the key:

   ```
   set up https://prolevelmax.com/SKILL.md
   ```

3. Ask for the data you need. The AI finds a source, tells you the price and runs it.

If your AI can run commands, it saves the key as an MCP server config so you do not paste it again. ChatGPT and Claude.ai connect over MCP with OAuth instead and need no key (see below).

## Supported clients

### Claude Code

With an API key:

```bash
claude mcp add --scope user --transport http prolevelmax https://prolevelmax.com/mcp --header "Authorization: Bearer sk_procli_..."
```

Without a key (OAuth): add the server without the header, then run `/mcp` inside Claude Code to sign in.

```bash
claude mcp add --scope user --transport http prolevelmax https://prolevelmax.com/mcp
```

### Codex

Pass the API key through an environment variable:

```bash
export PLM_API_KEY=sk_procli_...
codex mcp add prolevelmax --url https://prolevelmax.com/mcp --bearer-token-env-var PLM_API_KEY
```

### Cursor

One-click install with OAuth sign-in (no key): use **Install in Cursor** on the [docs page](https://prolevelmax.com/docs), confirm in Cursor, then sign in to ProLevelMax and click **Allow**.

Manual setup with an API key: **Settings → MCP → Add New MCP Server**, or put this in `mcp.json` (see [`mcp.json`](./mcp.json)):

```json
{
  "mcpServers": {
    "prolevelmax": {
      "url": "https://prolevelmax.com/mcp",
      "headers": { "Authorization": "Bearer sk_procli_..." }
    }
  }
}
```

### VS Code

One-click install with OAuth sign-in (no key): use **Install in VS Code** on the [docs page](https://prolevelmax.com/docs), confirm in VS Code, then sign in to ProLevelMax and click **Allow**.

Manual setup: add this to `.vscode/mcp.json` (see [`.vscode/mcp.json`](./.vscode/mcp.json)). VS Code starts the OAuth sign-in when the server first connects.

```json
{
  "servers": {
    "prolevelmax": {
      "type": "http",
      "url": "https://prolevelmax.com/mcp"
    }
  }
}
```

### Claude.ai and Claude Desktop

**Settings → Connectors → Add custom connector**, then paste:

```
https://prolevelmax.com/mcp
```

Sign in to ProLevelMax and click **Allow**. No API key needed.

### ChatGPT (web)

ChatGPT cannot call the REST API with a key, so add ProLevelMax as an app once:

1. Open **Plugins**, click **+ → Add custom MCP server**.
2. URL: `https://prolevelmax.com/mcp`, authentication: **OAuth**, then **Create**.
3. Sign in to ProLevelMax and click **Allow**.

In a chat, type `@` and pick **ProLevelMax**.

### Other clients

Any client that can read `SKILL.md` or connect to a remote MCP server over Streamable HTTP can use `https://prolevelmax.com/mcp` with OAuth or a Bearer key.

## Authentication

The MCP server accepts either:

- **OAuth 2.1** with PKCE (S256) and dynamic client registration. Authorization server metadata: [`https://prolevelmax.com/.well-known/oauth-authorization-server`](https://prolevelmax.com/.well-known/oauth-authorization-server). Protected resource metadata: [`https://prolevelmax.com/.well-known/oauth-protected-resource`](https://prolevelmax.com/.well-known/oauth-protected-resource). Scope: `mcp`.
- **API key** as a Bearer token: `Authorization: Bearer sk_procli_...`. The REST API uses this method only.

API keys and connected apps are listed in the [dashboard](https://prolevelmax.com/dashboard) and can be removed at any time. You can set daily or monthly spending limits per key or connected app.

## MCP tools

| Tool | What it does |
| --- | --- |
| `procli_discover` | Find sources for a request, in English or Chinese |
| `procli_inspect` | Input fields, price and quote for a source |
| `procli_run` | Run a source (charges your balance); returns a preview, the total, the cost, the balance and download links |
| `procli_runs_get` | Run status and results, in pages |
| `procli_balance` | Check the current balance |
| `procli_schedule_create` | Schedule a daily or weekly run (previews the cost; saved only after the user confirms) |
| `procli_schedule_list` | List schedules with their next and last runs |
| `procli_schedule_delete` | Delete a schedule |

## REST quick start

All requests use `Authorization: Bearer <key>` and JSON bodies.

**1. Discover** a source for your request:

```bash
curl https://prolevelmax.com/v1/discover \
  -H "Authorization: Bearer sk_procli_xxx" \
  -H "Content-Type: application/json" \
  -d '{"query":"reddit comments"}'
```

The response lists candidate sources (`recommended`, `items`), each with a `scraper` id, `bestFor`, `verdict` and `why`.

**2. Inspect** a source to get its input fields and a quote:

```bash
curl https://prolevelmax.com/v1/inspect \
  -H "Authorization: Bearer sk_procli_xxx" \
  -H "Content-Type: application/json" \
  -d '{"scraper":"procli_reddit_b","input":{"post_id":"t3_1abcde"}}'
```

`quote.maxCharge` is the most the run can cost.

**3. Run** it:

```bash
curl https://prolevelmax.com/v1/run \
  -H "Authorization: Bearer sk_procli_xxx" \
  -H "Content-Type: application/json" \
  -d '{"scraper":"procli_reddit_b","input":{"post_id":"t3_1abcde"}}'
```

The result includes `runId`, `items`, `count`, `cost`, `wallet` and `download` links. For long jobs add `"async": true` to get a `202` with a `runId`, then poll `GET /v1/runs/:id`.

**4. Download** the results as CSV, XLSX or JSON:

```bash
curl -o results.csv "https://prolevelmax.com/v1/runs/RUN_ID/download?format=csv" \
  -H "Authorization: Bearer sk_procli_xxx"
```

Check a key and its balance with `GET https://prolevelmax.com/v1/whoami`. Schedules are at `/v1/schedules`. Full endpoint list, pagination and error codes: [docs](https://prolevelmax.com/docs) and [openapi.json](https://prolevelmax.com/openapi.json).

## Example prompts

- Get this Reddit thread with all its comments: [thread link]. Summarise the main arguments for and against and quote the three most upvoted comments.
- Pull the comments on the top TikTok videos for the hashtag #homeespresso and group them into the most common questions.
- Find recent Xiaohongshu notes for the keyword "露营装备" and list the products people mention most.
- Check Reddit every Monday for posts that mention our brand and flag the negative ones.

## Pricing

Pay per use from one prepaid balance. No subscription.

- Each source is billed either per call, or per result plus a per-run fee with a per-run minimum.
- Every run gets a quote before it starts. The charge never exceeds the quote, and the balance cannot go below zero.
- The AI asks you to confirm before any run whose quote is above $0.10.
- Failed runs are generally not charged; if part of the work completed, a partial charge applies, never more than the quote.
- New users receive $1 in credit after verifying their email (one per person). No card required.

Rates for each platform: [prolevelmax.com/pricing](https://prolevelmax.com/pricing).

## Data scope and limits

- Public data only. Private accounts, messages and anything that needs someone's login are not collected.
- Up to 5 concurrent jobs per account (20 for Enterprise accounts), across all keys and connections.
- Run results are kept for 30 days.

## Support

- Email: [toph@prolevelmax.com](mailto:toph@prolevelmax.com)
- Contact page: [prolevelmax.com/contact](https://prolevelmax.com/contact)
- Roadmap and feature requests: [prolevelmax.com/roadmap](https://prolevelmax.com/roadmap)

## Links

- Docs: [prolevelmax.com/docs](https://prolevelmax.com/docs)
- Agent Skill: [prolevelmax.com/SKILL.md](https://prolevelmax.com/SKILL.md)
- OpenAPI: [prolevelmax.com/openapi.json](https://prolevelmax.com/openapi.json)
- Pricing: [prolevelmax.com/pricing](https://prolevelmax.com/pricing)

## License

[MIT](./LICENSE)
