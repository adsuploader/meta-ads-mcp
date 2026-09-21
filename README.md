<p align="center">
  <img src="assets/logo.png" alt="Ads Uploader" width="96" height="96">
</p>

<h1 align="center">Ads Uploader MCP</h1>

<p align="center">Launch and manage Meta ads from any AI agent — Claude, ChatGPT, Claude Code, Cursor, and Codex.</p>

---

Launch and manage Meta (Facebook & Instagram) ads from any AI agent — Claude, ChatGPT, Claude Code, Cursor, and Codex. Ads Uploader turns an existing ad into a reusable template, then builds brand‑new ads with new media, text, and targeting — in bulk, and always paused until you say go.

- **Website:** https://adsuploader.com
- **Hosted server:** `https://adsuploader.com/api/mcp`
- **Docs:** https://adsuploader.com/docs

> Draft‑first and safe by default: previews resolve posts and check Meta permissions without creating anything, and created ads are paused unless you explicitly ask otherwise.

## Setup

### Option A — Hosted server (recommended)

Add this URL as a custom MCP connector in Claude.ai, ChatGPT, or Cursor, then complete sign‑in and consent in your browser:

```text
https://adsuploader.com/api/mcp
```

In Claude Code:

```bash
claude mcp add --transport http ads-uploader https://adsuploader.com/api/mcp
```

For hosts that only speak stdio, bridge to the hosted server:

```bash
npx mcp-remote https://adsuploader.com/api/mcp
```

The hosted server authenticates through OAuth in your browser — no CLI login or token copying. Ask the agent to list your ad accounts to confirm the connection.

### Option B — Local stdio package

Requires Node.js 18+. Install and authenticate once in your terminal:

```bash
npm install -g @adsuploader/cli @adsuploader/mcp
ads login
```

Configure your MCP host:

```json
{
  "mcpServers": {
    "ads-uploader": {
      "command": "ads-mcp"
    }
  }
}
```

Use local stdio when the agent needs to upload files from disk. It reads the credentials saved by `ads login` and does not run OAuth itself.

## What the agent can do

Tools cover accounts, campaigns, ad sets, ads, pages, presets, media uploads, saved builds, previews, ad‑creation jobs, and post‑ID duplication. Every tool carries a display title and read‑only / destructive hints so hosts can show the right guardrails. Highlights:

- **`ads_create`** — build new ads from a full AdSpec (media, copy, CTA, targeting, campaign/ad‑set structure, creative enhancements). Created paused by default.
- **`ads_preview`** — resolve a spec or saved build and check Meta permissions without creating anything.
- **`ads_duplicate_by_post`** — duplicate existing ads by Page post ID so the original post's social proof is preserved.
- **`ads_upload`** — import creatives from public URLs or a Google Drive folder.
- Plus list/get tools for accounts, campaigns, ad sets, ads, pages, presets, uploads, builds, and jobs.

See [`SKILL.md`](./SKILL.md) for the full per‑parameter reference agents should follow.

## Security & privacy

- [SECURITY.md](./SECURITY.md) — how to report a vulnerability.
- [PRIVACY.md](./PRIVACY.md) — what the connector accesses and how data is handled.

## Support

Questions or issues: support@adsuploader.com
