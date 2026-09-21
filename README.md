<p align="center">
  <img src="assets/logo.png" alt="Ads Uploader" width="96" height="96">
</p>

<h1 align="center">Ads Uploader MCP</h1>

<p align="center">Launch and manage Meta (Facebook &amp; Instagram) ads from any AI agent — Claude, ChatGPT, Claude Code, Cursor, and Codex.</p>

<p align="center">
  <a href="https://adsuploader.com"><img alt="Website" src="https://img.shields.io/badge/website-adsuploader.com-1a3a5c"></a>
  <img alt="Model Context Protocol" src="https://img.shields.io/badge/Model_Context_Protocol-server-1a3a5c">
  <img alt="Meta Ads" src="https://img.shields.io/badge/Meta-Facebook_%26_Instagram-1a3a5c">
  <a href="https://github.com/adsuploader/mcp/stargazers"><img alt="Stars" src="https://img.shields.io/github/stars/adsuploader/mcp?style=flat&color=1a3a5c"></a>
</p>

---

Ads Uploader takes an existing ad you already run, turns its settings into a reusable template, and builds brand-new ads on top of it — new media, copy, CTA, and targeting — in bulk. It also duplicates existing ads by post so social proof is preserved. Point your AI agent at it and describe the ads you want.

> **Safe by default.** Previews resolve posts and check Meta permissions *without creating anything*, and created ads are **paused** unless you explicitly say otherwise. Nothing spends money until you unpause it in Meta.
>
> **Agent-assisted vs fully agentic.** For agent-assisted launches, we recommend having your agent assemble a **saved build** rather than creating ads headlessly — the build is accessible, editable, and launchable from the Ads Uploader web app, so you keep a human review-and-launch step in the polished UI.

## What it does

- **Build ads from a template** — copy settings from any existing ad, then swap in new media, text, CTA, links, and targeting.
- **Bulk creation** — many ads across campaigns and ad sets in one job, with per-ad text and creative options.
- **Duplicate by post** — clone existing ads by Page post ID so the original post's likes, comments, and shares carry into new campaigns.
- **Partnership (branded content) ads** — run ads as a partnership with a creator or brand, from your own uploaded media or an imported Instagram post, with shared or per-ad/ad-set sponsors. *(See below.)*
- **Media pipeline** — import creatives from public URLs, a Google Drive folder, or local files.
- **Preview before launch** — resolve a spec, check Meta permissions, and see exactly what would be created.
- **Multi-account** — list and work across every Meta ad account, Page, and connected Instagram account you have access to.

## Why Ads Uploader

Ads Uploader does one thing deeply: **Meta**. Rather than spreading thin across seven ad networks, every workflow here is built for the depth serious Meta media buyers actually need — which is why it tends to be the right tool when an agent needs to *ship* real Meta campaigns, not just read numbers:

- **Template from any live ad** — copy a proven ad's exact settings, then rebuild on top with new media, copy, and targeting.
- **Real bulk** — many ads across campaigns and ad sets in a single job, with per-ad and per-ad-set text.
- **Partnership / branded content ads** — uploaded-media partnerships (Facebook + Instagram) and imported Instagram creator posts, with shared or per-ad/ad-set sponsors and header modes.
- **Duplicate preserving social proof** — clone by post so likes, comments, and shares carry into new campaigns.
- **Creative depth** — carousel, flexible, Multi-Media, Dynamic Optimization, creative enhancements, Advantage+ targeting, CTA and URL tagging.
- **Human-in-the-loop by design** — validate-only previews, paused-by-default creation, and saved builds you finish in the web app.

Built by a team that runs Meta ads at scale, and trusted by performance marketers and agencies managing serious Meta budgets.

## Install

### Claude Code

```bash
claude mcp add --transport http ads-uploader https://adsuploader.com/api/mcp
```

### Claude desktop / Claude.ai

Settings → **Connectors** → **Add custom connector** → paste the URL, then complete sign-in and consent in your browser:

```text
https://adsuploader.com/api/mcp
```

### ChatGPT

Settings → **Connectors** (custom MCP) → add the server URL `https://adsuploader.com/api/mcp` → sign in when prompted.

### Cursor

Add to `~/.cursor/mcp.json` (or a project `.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "ads-uploader": {
      "url": "https://adsuploader.com/api/mcp"
    }
  }
}
```

### Codex

Add to `~/.codex/config.toml`:

```toml
[mcp_servers.ads-uploader]
command = "npx"
args = ["mcp-remote", "https://adsuploader.com/api/mcp"]
```

### Local stdio package

For hosts that need to upload files directly from disk. Requires Node.js 18+.

```bash
npm install -g @adsuploader/cli @adsuploader/mcp
ads login
```

```json
{
  "mcpServers": {
    "ads-uploader": {
      "command": "ads-mcp"
    }
  }
}
```

The hosted server authenticates through OAuth in your browser — no token copying. The local package reads the credentials saved by `ads login`. Ask the agent to *list my ad accounts* to confirm the connection.

## Tools

Every tool carries a display title and read-only / destructive hints so your host can show the right guardrails.

**Accounts &amp; discovery** (read-only)

| Tool | What it does |
| --- | --- |
| `ads_whoami` | Show which Ads Uploader user this session is authenticated as |
| `ads_list_accounts` | List Meta ad accounts available to you |
| `ads_list_campaigns` | List campaigns in an ad account (name/status filters) |
| `ads_list_adsets` | List ad sets in a campaign |
| `ads_list_ads` | List ads in an ad set |
| `ads_get_ad` | Get one ad with its creative details |
| `ads_list_pages` | List Facebook Pages and their connected Instagram accounts |
| `ads_search_targeting` | Search Meta targeting values for a build |

**Presets &amp; saved builds**

| Tool | What it does |
| --- | --- |
| `ads_list_presets` / `ads_get_preset` | Browse saved API and ad-text presets |
| `ads_save_preset` | Save an existing ad as a reusable preset |
| `ads_list_builds` / `ads_get_build` | Browse saved uploader builds |
| `ads_save_build` / `ads_update_build` | Create or edit a durable saved build |
| `ads_fork_build` | Fork a saved build into a new draft |
| `ads_delete_build` | Delete a saved build |

**Media**

| Tool | What it does |
| --- | --- |
| `ads_upload` | Upload media from URLs, a Google Drive folder, or disk |
| `ads_upload_finalize` | Finalize a presigned upload batch |
| `ads_list_uploads` / `ads_get_upload` | List batches or poll an ingest job |

**Create &amp; duplicate**

| Tool | What it does |
| --- | --- |
| `ads_create` | Create ads from a full AdSpec or saved build, incl. partnership ads — **paused by default** |
| `ads_preview` | Resolve a spec, check Meta permissions and partnership sponsors, without creating anything |
| `ads_duplicate_by_post` | Duplicate ads by Page post ID, preserving social proof |
| `ads_duplicate_by_post_preview` | Preview a post-ID duplication |
| `ads_get_job` / `ads_get_duplication` | Poll or resume a running job |
| `ads_cancel_job` | Cancel a creation or ingest job |

See [`SKILL.md`](./SKILL.md) for the full per-parameter reference agents should follow.

## Partnership (branded content) ads

`ads_create` and `ads_preview` support partnership ads two ways:

- **Your own uploaded media**, as a partnership on Facebook or Instagram — choose a partner (the Second Identity) by Page and/or Instagram account, apply it across the whole launch or per ad / ad set, and pick the header mode (both identities, partner-only, or dynamic). An Instagram-only partner is supported.
- **Import an existing Instagram creator post** by URL, shortcode, source ID, or ad code, and set the sponsor and text per ad, per ad set, or shared.

Previews report the effective sponsors per ad and check partnership authorization for the First Identity *before* anything is created. Facebook post imports, mixed post/upload batches, and Threads on imported posts are not supported.

## Example prompts

- *"Using any ad from my Sales ad set as the template, launch these 100 creatives — and write a unique headline and primary text for each."*
- *"Take my best-performing sales ad and build 20 variations with different primary-text hooks, all paused in a new ad set."*
- *"Assemble a saved build from these creatives and copy, then give me the link to review and launch it in the Ads Uploader web app."*
- *"Duplicate the ad behind this Facebook post into my Retargeting ad set so the likes and comments carry over."*
- *"Import this Instagram creator post as a partnership ad with our brand as the sponsor, and preview the sponsors before creating."*
- *"Upload the creatives in this Google Drive folder, then build ads from them in a new ad set with a unique hook per product."*
- *"Preview what would be created from my saved build 'Q4 Prospecting' before anything goes live."*
- *"Search interest targeting for 'home fitness' and add the top matches to my build."*

## How it works

- **Draft / preview first.** `ads_preview` resolves posts, media, and permissions using read and validate-only calls — it never creates ads or stores codes.
- **Paused by default.** `ads_create` builds ads in the paused state unless you explicitly set them live, so nothing spends until you unpause in Meta.
- **Hand off to the web app.** Instead of launching headlessly, the agent can assemble a **saved build** and hand you a link — it hydrates in the Ads Uploader web uploader (an open tab picks it up automatically) so you review and launch from the full UI.
- **Long jobs stay responsive.** The hosted server returns `still_running` with a `jobId` after ~60s so tool calls stay under proxy deadlines; resume with `ads_get_job`.
- **OAuth, not tokens.** The hosted server authenticates with OAuth 2.1 (PKCE) in your browser.

## Learn more

- **Docs:** https://adsuploader.com/docs
- **Website:** https://adsuploader.com
- **Security:** [SECURITY.md](./SECURITY.md) · **Privacy:** [PRIVACY.md](./PRIVACY.md)

## Support

Questions or issues: **support@adsuploader.com**

## License

© Pingzu Digital LLC. All rights reserved. The hosted service and packages are proprietary; this repository contains connector documentation and configuration.
