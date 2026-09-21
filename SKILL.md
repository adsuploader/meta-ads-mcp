---
name: ads-mcp
description: Create and manage Facebook/Meta ads through the Ads Uploader MCP server. Upload media, preview ad specs, create ads, browse campaigns/ad sets/ads, manage presets/uploads/builds/jobs.
homepage: https://adsuploader.com
---

## Setup

### Option A: Hosted server (recommended)

Add this URL as a custom MCP connector in Claude.ai, ChatGPT, or Cursor, then complete the sign-in and consent screen in your browser:

```text
https://adsuploader.com/api/mcp
```

In Claude Code, run:

```bash
claude mcp add --transport http ads-uploader https://adsuploader.com/api/mcp
```

For hosts that only support stdio, bridge to the hosted server with:

```bash
npx mcp-remote https://adsuploader.com/api/mcp
```

The hosted server authenticates through OAuth in your browser. No CLI login or token copying is needed. Ask the agent to list your ad accounts to check the connection.

### Option B: Local stdio package

Requires Node.js 18 or later. Install both packages and authenticate once in your terminal:

```bash
npm install -g @adsuploader/cli @adsuploader/mcp
ads login
```

Configure your MCP host with:

```json
{
  "mcpServers": {
    "ads-uploader": {
      "command": "ads-mcp"
    }
  }
}
```

The local server reads `~/.config/adsuploader/credentials.json` saved by `ads login`. It does not open a browser or run OAuth itself. Use local stdio when the agent needs to upload files from disk with `paths`.

## Workflow

1. Use `ads_list_accounts` to pick an `accountId`.
2. Use `ads_upload` with local `paths`, public HTTPS `files[].url` sources, or a public Google Drive `driveFolderUrl`. URL and Drive modes return a `jobId`; poll it with `ads_get_upload` until complete and use the final `batchId`.
3. Use `ads_preview` with `accountId`, `uploadId`, and ad configuration JSON.
4. Use `ads_create` with the same body when the preview is correct.

## Tools at a glance

| Tool | What it does | Key inputs |
|------|--------------|-----------|
| `ads_whoami` | Show the authenticated user and API host | none |
| `ads_list_accounts` | List Meta ad accounts you can use | none |
| `ads_list_campaigns` | List campaigns in an account | `accountId`, `search`, `status`, `limit`, `offset` |
| `ads_list_adsets` | List ad sets in a campaign | `accountId`, `campaignId`, `search`, `status`, `limit`, `offset` |
| `ads_list_ads` | List ads in an ad set | `accountId`, `adSetId`, `search`, `status`, `limit`, `offset` |
| `ads_get_ad` | Get one ad with its creative details | `accountId`, `adId` |
| `ads_list_pages` | List Facebook Pages + connected Instagram | `accountId`, `limit`, `offset` |
| `ads_search_targeting` | Look up city keys and detailed-targeting IDs | `accountId`, `query`, `type`, `limit` |
| `ads_list_presets` | List saved API or ad-text presets | `accountId`, `type`, `limit`, `offset` |
| `ads_get_preset` | Get one preset | `accountId`, `presetId` |
| `ads_save_preset` | Save an existing ad as a reusable preset | `accountId`, `fromAdId`, `name`, `shareWithTeam` |
| `ads_list_builds` | List saved uploader builds | `accountId`, `limit`, `offset` |
| `ads_save_build` | Save a durable build (spec and/or webState) | `accountId`, `name`, `notes`, `spec`, `webState` |
| `ads_get_build` | Get one build (full spec + webState) | `buildId` |
| `ads_fork_build` | Duplicate a build into a new draft | `buildId` |
| `ads_update_build` | Rename/patch/replace a build | `buildId`, `expectedRevision`, `name`, `notes`, `specPatch`, `spec`, `webState` |
| `ads_delete_build` | Delete a build | `buildId` |
| `ads_list_uploads` | List recent media upload batches | `accountId`, `limit`, `offset` |
| `ads_get_upload` | Get a batch, or poll an ingest job | `batchId` or `jobId` |
| `ads_upload` | Upload media (paths / URLs / Drive / base64 / presigned) | `accountId`, `paths`, `files`, `driveFolderUrl`, `presigned`, `batchId`, `cwd` |
| `ads_upload_finalize` | Finalize a presigned upload batch | `accountId`, `batchId`, `files`, `thumbnailMap` |
| `ads_preview` | Dry-run: show what ads would be created | same body as `ads_create` |
| `ads_create` | Create ads from a spec body or saved build | see "Create and preview inputs" below |
| `ads_get_job` | Get/resume an ad-creation job | `jobId`, `operations` |
| `ads_cancel_job` | Cancel an ad-creation or ingest job | `jobId` |
| `ads_duplicate_by_post_preview` | Dry-run a post-ID duplication; returns the plan and any source candidates | see "Duplicate by post ID" |
| `ads_duplicate_by_post` | Duplicate existing ads by Page post ID, preserving their post references | see "Duplicate by post ID" |
| `ads_get_duplication` | Get/resume a duplication job | `jobId` |

All tool inputs mirror the public `/api/v1/` JSON request bodies. Every list/browse tool takes the same paging shape: `limit` (1-100, default 25) and `offset` (default 0), plus a name `search` and a `status` filter (`active` default, or `all` to include paused/archived) where noted above.

## Create and preview inputs

`ads_create` and `ads_preview` accept the same request body. Preview first, create when it looks right. You must supply media one of three ways, then layer the ad configuration on top.

### Required: pick a source

- `accountId` + `uploadId` - create from a completed upload batch.
- `accountId` + `mediaItems` - create from portable saved-build media metadata (images resolve by `mediaHash`, videos by `mediaId`; standard/carousel/flexible/placement videos also need `thumbnailHash`).
- `buildId` - create from a saved build's portable spec. Runtime `accountId` is taken from the build unless you pass it.

Fields:

- `accountId` (string) - ad account ID, e.g. `act_123`.
- `uploadId` (string) - batch ID from `ads_upload` / `ads_get_upload`.
- `buildId` (string) - saved build ID; its spec becomes the create body.
- `mediaItems` (array) - saved-build media metadata; account-bound, so `accountId` must match the account the media was captured under.

### Reuse an existing ad or preset

- `copyFromAd` (string) - existing Meta ad ID to copy settings from. Mutually exclusive with `adPresetId`.
- `adPresetId` (string) - saved API preset ID. Mutually exclusive with `campaign`, `adSet`, and `copyFromAd`.
- `textPresetId` (string) - saved text preset ID. Cannot be combined with `texts`.

### `campaign` - campaign structure

```json
{ "campaign": { "mode": "duplicate", "campaigns": [{ "name": "Campaign A" }, { "name": "Campaign B" }] } }
```

- `mode` (string, default `single`) - `single` (one campaign), `duplicate` (all media into each campaign), or `split` (media split across campaigns).
- `id` (string) - target an existing campaign.
- `name` (string) - create a new campaign with this name.
- `campaigns` (array) - for `duplicate`/`split`, each `{ id?, name?, mediaIds? }`.
- `dailyBudget` (number, > 0) - CBO daily campaign budget in whole account currency units (e.g. `100` = $100). Mutually exclusive with `lifetimeBudget` and ad-set budgets. Omit both campaign budget fields to inherit the source campaign's budget.
- `lifetimeBudget` (number, > 0) - CBO lifetime campaign budget in whole account currency units. Mutually exclusive with `dailyBudget` and ad-set budgets. Omit both campaign budget fields to inherit the source campaign's budget.

Single campaign is the default: ads go into the template ad's campaign, or a new one when you pass `campaign.name`.

### `adSet` - ad set structure, budget, naming

```json
{ "adSet": { "mode": "autoGroup", "adsPerAdSet": 5, "namePattern": "Ad Set {index:01}", "dailyBudget": 50 } }
```

- `mode` (string, default `single`) - `single`, `perUpload` (one ad set per file), `autoGroup` (split evenly, pair with `adsPerAdSet`), or `custom` (use `groups`).
- `id` (string) - target an existing ad set.
- `name` (string) - name for a single new ad set.
- `namePattern` (string) - naming template for multi-ad-set modes, e.g. `Ad Set {index:01}`.
- `adsPerAdSet` (integer, min 1) - ads per ad set in `autoGroup`.
- `groups` (array) - explicit `custom` groups, each `{ name, mediaIds? }`.
- `groupVariations` (boolean) + `variationIdentifier` (string) - group variant files (same prefix before the identifier) into one ad set.
- `dailyBudget` (number, > 0) - ABO daily budget in account currency units (e.g. `50` = $50). Do not set this for CBO campaigns (budget lives on the campaign; Meta rejects an ad-set budget).
- `bidAmount` (number, > 0) - bid cap in account currency units. Bid-cap strategies (`COST_CAP`, `LOWEST_COST_WITH_BID_CAP`, `TARGET_COST`) need this. `dailyBudget` and `bidAmount` are independent Meta fields; set either or both.
- `minimumRoas` (number, > 0) - minimum ROAS ratio for `LOWEST_COST_WITH_MIN_ROAS` (e.g. `1.5` = 1.5x). Mutually exclusive with `bidAmount`; it can coexist with an ABO daily budget or a CBO campaign budget.
- `minSpend` (number, >= 0) - minimum spend under CBO in whole account currency units. Positive values require `campaign.dailyBudget` or `campaign.lifetimeBudget` in the same request; `0` removes the limit inherited from the source ad set. Mutually exclusive with `minSpendPercentage`.
- `maxSpend` (number, >= 0) - maximum spend under CBO in whole account currency units. Positive values require `campaign.dailyBudget` or `campaign.lifetimeBudget` in the same request; `0` removes the limit inherited from the source ad set. Mutually exclusive with `maxSpendPercentage`.
- `minSpendPercentage` (number, 5-100 in steps of 5, matching Ads Manager) - minimum spend as a percentage of the CBO campaign budget. Also valid when the destination is an existing CBO campaign. Mutually exclusive with `minSpend`.
- `maxSpendPercentage` (number, 5-100 in steps of 5, matching Ads Manager) - maximum spend as a percentage of the CBO campaign budget. Also valid when the destination is an existing CBO campaign. Mutually exclusive with `maxSpend`.

### `targeting` - audience for new ad sets

One audience applied to every new ad set in the launch. Omit `targeting` entirely to inherit the source ad set's audience unchanged. This is an AdSpec-shaped override, not raw Meta `flexible_spec`.

```json
{
  "targeting": {
    "selectionVersion": 4,
    "countries": ["US", "CA"],
    "cities": [{ "key": "2525495", "name": "Austin", "region": "Texas", "countryCode": "US", "radius": 25, "distance_unit": "mile" }],
    "ageSelected": true, "ageMin": 27, "ageMax": 52,
    "genderSelected": true, "genders": "women",
    "detailedTargetingSelected": true,
    "detailedTargetingGroups": [[
      { "id": "6003700363290", "name": "Performance-based advertising", "type": "interests" }
    ]]
  }
}
```

- `countries` (array of strings) - ISO 3166-1 alpha-2 codes, e.g. `["US","CA"]`.
- `cities` (array) - each needs `key` (from `ads_search_targeting` type `city`); optional `radius` (10-50 miles or 17-80 km) and `distance_unit` (`mile`|`kilometer`). Cities are added to countries, not subtracted: to target only a city, send no countries.
- `ageMin` / `ageMax` (integers 13-65) - opt-in: set `ageSelected: true` or the values are ignored. `ageMin` must be <= `ageMax`.
- `genders` (string) - `all`, `men`, or `women`. Opt-in: set `genderSelected: true`.
- `detailedTargetingGroups` (array, one OR group for now) - each selection needs `id` and `type`, plus `name` or `value`. `type` is the Meta `flexible_spec` key (`interests`, `behaviors`, `work_employers`, `work_positions`, `industries`, `life_events`, and so on). Get `id`/`name`/`type` from `ads_search_targeting` type `detailed`; never guess them. Detailed targeting replaces, it does not merge - any type you omit is dropped.
- `detailedTargetingSelected` (boolean) - set `true` to override detailed targeting. `true` with an empty `detailedTargetingGroups` array clears the source's detailed targeting; omit or `false` inherits it.
- `selectionVersion` (integer 2|3|4) - metadata version. Use `4` for mixed detailed targeting (required when you send `detailedTargetingGroups`/`detailedTargetingSelected`). `3` is for legacy interest-only builds (`interestGroups`/`interestsSelected`); `2` is for legacy age/gender-only builds. Does not reach Meta.
- `interestGroups` (array) + `interestsSelected` (boolean) - legacy version-3 interest-only shape, kept so older saved builds still launch. Prefer `detailedTargetingGroups` for new work.

Use `ads_search_targeting({ accountId, query, type })` to discover values: `type: "city"` returns the `key` for `cities[]`; `type: "detailed"` returns `{ id, name, type }` for `detailedTargetingGroups`. IDs and city keys must never be guessed.

### `profile` - Page / Instagram / Threads override

By default created ads inherit the Page, Instagram, and Threads identity from the template ad or preset. Override with `profile`:

```json
{ "profile": { "pageId": "123456789012345", "instagramId": "17841400000000000", "threadsId": "987654321098765" } }
```

- `pageId` (string or null) - Facebook Page ID. List available pages with `ads_list_pages`.
- `instagramId` (string or null) - Instagram account ID.
- `useFacebookPage` (boolean) - use the Facebook Page as the Instagram identity when it has no linked Instagram account. Cannot be combined with `instagramId`.
- `threadsId` (string or null) - Threads profile ID.
- `campaigns` (object) - per-campaign identity overrides keyed by campaign ID or unambiguous campaign name, each `{ pageId?, instagramId?, useFacebookPage?, threadsId? }`, for multi-campaign launches.

If you override only `pageId`, the API automatically uses that Page's linked Instagram account, or the Facebook Page identity when no account is linked. Threads still resets because it may belong to the old Page; set `threadsId` explicitly when needed. `ads_list_pages` returns each Page's connected Instagram account.

### `texts` - headlines, bodies, descriptions, CTAs, links

```json
{
  "texts": {
    "mode": "common",
    "strategy": "flexible",
    "common": { "headlines": ["Headline 1", "Headline 2"], "bodies": ["Primary text"], "descriptions": ["Description"] }
  }
}
```

- `mode` (string, default `common`) - `common` (same text for all ads), `perAdset` (text per ad set), or `perAd` (text per file).
- `strategy` (string, default `flexible`) - `flexible` (Meta mixes multiple headlines/bodies inside one ad) or `separate` (each text combination becomes a separate ad).
- `common` (text block) - shared text (see text-block fields below).
- `perAd` (object) - text keyed by **filename** (not full path); unspecified fields inherit from the template ad. Preferred shape for one-ad-per-ad-set builds because filenames always match.
- `perAdset` (object) - text keyed by the **final planned ad-set name or ID**. Ad-set names are often generated at create time (e.g. `Ad Set 01`), so a descriptive key that does not match the planned name will not resolve. Use `perAd` unless an ad set genuinely holds multiple ads.
- `urlVariants` (array) - 2-5 destination URLs for a landing-page A/B test; each `{ link, label? }`. Every generated ad set is duplicated once per URL so Meta optimizes each ad+URL pair independently. `label` is optional (falls back to the URL slug). Your ad set budget is multiplied by the number of variants. Each URL must be distinct.

**Text block fields** (used in `common`, each `perAd`/`perAdset` entry):

- `headlines` (array of strings), `bodies` (array of strings), `descriptions` (array of strings).
- `cta` or `callToAction` (string) - CTA button type (see CTA values below); both keys are accepted.
- `link` (string) - destination URL.
- `displayUrl` (string) - display link shown on the ad.
- `urlTags` (string) - query-string tracking params, e.g. `utm_content=hero`.
- `aiDisclosure` (boolean) - per-ad or per-ad-set AI self-disclosure override (see AI disclosure below).

`textPresetId` cannot be combined with `texts`.

### `cta` - top-level call to action and links

```json
{ "cta": { "type": "SHOP_NOW", "link": "https://example.com", "displayUrl": "example.com" }, "urlTags": "utm_source=facebook&utm_medium=paid" }
```

- `cta.type` (string) - CTA button type. Applies to all ads unless a per-ad `cta` overrides it.
- `cta.link` (string) / `cta.displayUrl` (string) - destination and display URL.
- `urlTags` (string, top-level) - tracking query string appended to every ad's link.

**Common CTA values:** `LEARN_MORE`, `SHOP_NOW`, `SIGN_UP`, `SUBSCRIBE`, `GET_OFFER`, `GET_QUOTE`, `CONTACT_US`, `GET_IN_TOUCH`, `BOOK_TRAVEL`, `ORDER_NOW`, `BUY_NOW`, `APPLY_NOW`, `DOWNLOAD`, `SEE_DETAILS`, `WATCH_MORE`, `LISTEN_NOW`, `PLAY_GAME`, `DONATE_NOW`, `OPEN_LINK`, `MESSAGE_PAGE`, `WHATSAPP_MESSAGE`, `INSTAGRAM_MESSAGE`. Other Meta-valid CTA values also pass through. Objective-specific CTAs (`MESSAGE_PAGE`, `WHATSAPP_MESSAGE`, `INSTAGRAM_MESSAGE`, `CALL_NOW`) require a matching source ad/objective and are normally inherited, not set by hand.

### `carousel` - carousel ads

```json
{
  "carousel": [{
    "name": "My Carousel",
    "cards": ["slide1.jpg", "slide2.jpg", "slide3.jpg"],
    "headlines": ["Overall Carousel Headline"], "bodies": ["Overall primary text"], "descriptions": ["Overall description"],
    "cta": "SHOP_NOW", "link": "https://example.com/carousel", "displayUrl": "example.com", "urlTags": "utm_content=my_carousel",
    "cardTexts": [{ "name": "slide1.jpg", "headline": "Card 1", "description": "Card 1 desc", "link": "https://example.com/1", "callToAction": "LEARN_MORE" }]
  }]
}
```

Each group: `name` (string), `cards` (array of 2-10 media references by filename), optional overall `headlines`/`bodies`/`descriptions`, `cta`, `link`, `displayUrl`, `urlTags`, and `cardTexts` (per-card `{ name, headline, description, link, callToAction/cta }`).

### `flexible` - flexible-format ads

```json
{ "flexible": [{ "name": "Flex Ad", "assets": ["hero.jpg", "promo.mp4", "banner.jpg"] }] }
```

Each group: `name` (string) and `assets` (array of 2-10 media references by filename).

### `multimedia` - Multi Media ads

```json
{
  "multimedia": [{
    "name": "Mixed Media Ad",
    "assets": ["hero.jpg", "promo.mp4", "banner.jpg"],
    "assetTexts": [{ "headline": "Hero headline", "primaryText": "Hero primary text", "description": "Hero desc", "link": "https://example.com/hero", "displayUrl": "example.com/hero" }]
  }]
}
```

Each group: `name`, `assets` (2-10 media references), optional `assetTexts` lined up with `assets` by index (each field a single per-asset override; blank fields fall back to the ad's main text). Videos need a captured public thumbnail URL.

### `creativeEnhancements` - Advantage+ creative features

Via MCP, pass this as an **object** (the string shorthand `"all"`/`"none"`/`"metaDefaults"` and bare feature arrays are CLI-only):

```json
{ "creativeEnhancements": { "useMetaDefaults": false, "features": ["text_translation", "image_touchups"] } }
```

- `{ "useMetaDefaults": true }` - let Meta decide (this is also what you get if you omit `creativeEnhancements` entirely).
- `{ "useMetaDefaults": false, "features": [...] }` - turn on the listed features, everything else off.
- `{ "text_translation": true, "enhance_cta": false }` - explicit per-feature on/off (any feature key is accepted).

Available features: `text_translation`, `inline_comment`, `enhance_cta`, `text_optimizations`, `reveal_details_over_time`, `show_destination_blurbs`, `image_brightness_and_contrast`, `image_touchups`, `video_auto_crop`, `video_filtering`, `image_animation`, `image_templates`, `adapt_to_placement`, `product_extensions`, `description_automation`, `add_text_overlay`, `music`, `carousel_to_video`, `carousel_dynamic_description`, `multi_share_end_card`, `multi_share_optimized`. Only list features relevant to the media type (`video_auto_crop`/`video_filtering` are video-only; `carousel_*` are carousel-only). `product_tags` only works when the source ad has a catalog with positioned product tags.

`show_destination_blurbs` is displayed as **Show spotlights**.

### `aiDisclosure` - Meta AI-content self-disclosure

```json
{ "aiDisclosure": true }
```

Self-disclose that an ad's creative was made or significantly edited with AI (Meta's GenAI transparency). Type: boolean. **Off by default and never auto-enabled** - if you omit it, no disclosure is sent. When `true`, every ad in the launch opts in. An explicit value (including `false`) always wins; if you only copy from an existing ad and set nothing, the disclosure state is inherited from that source ad. This matches the CLI `--ai-disclosure` behaviour.

You can also set it per ad or per ad set inside a `texts` block: put `"aiDisclosure": true` (or `false`) inside a `texts.perAd["file.jpg"]` or `texts.perAdset["Ad Set 01"]` entry. Per-entry values override the top-level flag for those ads.

### `adNamePattern` and ad naming

```json
{ "adNamePattern": "{filename} - {date}", "resetAdNameIndexPerAdSet": true }
```

- `adNamePattern` (string) - ad name template. Placeholders: `{filename}`, `{index:01}`, `{variation}`, `{group}`, `{adset}`, `{campaign}`, `{date}`, `{date:short}`, `{timestamp}`. Transforms: `{clean:...}` (strips a trailing aspect-ratio suffix), `{lowercase:...}`, `{uppercase:...}`, `{titlecase:...}`, `{split:DELIM:POS}`.
- `resetAdNameIndexPerAdSet` (boolean) - restart the `{index}` counter in each ad set.

### `options` - status, scheduling, pause level

```json
{ "options": { "status": "PAUSED", "pauseAt": "ad", "schedule": { "startTime": "2026-04-01T09:00:00", "endTime": "2026-04-30T23:59:59" } } }
```

- `status` (string) - `ACTIVE` (default) or `PAUSED`.
- `pauseAt` (string) - which level `PAUSED` applies to: `ad` (default), `adSet`, or `campaign`.
- `schedule.startTime` / `schedule.endTime` (ISO 8601 strings) - scheduled start/end, interpreted in the ad account timezone. `schedule.timezone` may also be set.
- `testMode` (boolean, default false) - **admin accounts only.** Runs the create path without creating real ads; non-admin callers get a 403. Leave it off for normal launches.

### Saved-build metadata (ignored on create)

`launchable` and `launchBlockers` may appear on a spec read back from a saved build. They are metadata only and are ignored when creating ads; do not set them by hand.

## Browsing and lookup

- `ads_list_campaigns` / `ads_list_adsets` / `ads_list_ads` - each takes `accountId` (and `campaignId`/`adSetId`), plus `search` (name text), `status` (`active` default, `all` includes paused/archived), `limit` (1-100, default 25), and `offset`.
- `ads_get_ad({ accountId, adId })` - one ad with full creative details. Handy to find an ad ID for `copyFromAd` or `ads_save_preset`.
- `ads_list_pages({ accountId })` - Facebook Pages plus their connected Instagram accounts, for `profile` overrides.
- `ads_search_targeting({ accountId, query, type, limit })` - `type` is `city` or `detailed`; `query` needs 2+ characters; `limit` is 1-25 (default 8). The only correct way to get city keys and detailed-targeting IDs.
- `ads_list_presets({ accountId, type })` - `type` is `api` (default) or `adText`. `ads_get_preset({ accountId, presetId })` reads one. `ads_save_preset({ accountId, fromAdId, name, shareWithTeam })` saves an existing ad as a reusable API preset (`shareWithTeam` default false).

## Saved builds

Builds are durable, editable launch configurations. A spec-only build is preferred for agent-created work: the web uploader hydrates it and generates its own `webState` on resume.

- `ads_save_build({ accountId?, name?, notes?, spec?, webState? })` - provide at least `spec` or `webState`. `accountId` can be derived from `webState.selectedAccount` or `spec.accountId`. `name` is up to 120 chars; `notes` up to 2000. Do not construct `webState` by hand.
- `ads_get_build({ buildId })` - full `spec` and `webState`, untruncated. Read `revision` here before patching.
- `ads_fork_build({ buildId })` - duplicate a build into a new autosaved draft; use this when the user wants a variant and the original must stay intact.
- `ads_update_build({ buildId, expectedRevision?, name?, notes?, specPatch?, spec?, webState? })`:
  - `specPatch` (object) - preferred for content edits. A field-aware merge: omitted fields survive, nested objects (including individual `texts.perAd`/`perAdset` entries) merge, supplied arrays replace, and `null` removes a field. So changing headlines does not wipe `link`, `displayUrl`, CTA, URL tags, or `aiDisclosure`. Requires `expectedRevision` from `ads_get_build`.
  - `spec` (object or null) - full replacement of the portable spec; send the entire object, not a diff. `null` clears it. Do not send both `spec` and `specPatch`.
  - `expectedRevision` (integer) - optimistic-lock guard; required with `specPatch`, recommended on every update so a stale write fails instead of clobbering a newer revision.
  - `name` - setting a name promotes an autosaved build to a kept, named build.
- `ads_delete_build({ buildId })` - delete a build.

Launch a saved build headlessly with `ads_create({ buildId })` when its spec is `launchable`. `launchable` only describes headless `ads_create` launches; a `launchable:false` build still opens, edits, and launches fine in the web uploader, so do not warn web users about it.

## Uploads

`ads_upload({ accountId, ... })` accepts one media source per call:

- `paths` (array of strings) - local files or directories, resolved from `cwd` (or the server's working directory). Local stdio server only; not available on the remote/hosted server.
- `files` (array, up to 50) - each item is one of:
  - `{ "url": "https://..." }` - public HTTPS URL or Google Drive file link (detected automatically). The filename comes from the source; do not send `name` with a URL. Returns a background `jobId`.
  - `{ "name": "...", "contentType": "image/jpeg", "base64": "..." }` - inline bytes, for a couple of small images only.
  - `{ "name", "contentType", "size" }` with `presigned: true` - metadata for environments that can PUT bytes themselves.
  - `thumbnailFor` (string) on a file attaches it as a custom thumbnail to the named video; `*_thumbnail` names are matched automatically when omitted.
- `driveFolderUrl` (string) - public Google Drive folder link; imported server-side. Returns a background `jobId`.
- `batchId` (string) - group this upload with an earlier batch.
- `presigned` (boolean) - request PUT URLs (large files may come back as multipart parts). Do not mix URL and base64 media in one call; submit separate batches.

`ads_upload_finalize({ accountId, batchId, files, thumbnailMap? })` completes a presigned batch after the client has PUT each file, echoing back `name`/`type`/`size`/`contentType`/`fileKey` (and `uploadId`/`parts` for multipart) exactly as returned. `thumbnailMap` optionally maps video keys to thumbnail keys.

Poll ingest jobs with `ads_get_upload({ jobId })` until `complete` is true; the final batch summary is in `result`. `ads_get_upload({ batchId })` reads a completed batch. Cancel an unwanted import with `ads_cancel_job({ jobId })`.

## Duplicate by post ID

Copy existing ads by their Page post ID, preserving the original post's engagement (likes, comments, shares). This is the web Duplicator's behavior, headless.

- `ads_duplicate_by_post_preview({ accountId, postIds, campaignId, ... })` first. It resolves each `postIds` entry to a specific ad (discovery) and returns the source-to-destination plan. If a post matches more than one ad it returns `sourceCandidates` with `requiresSourceSelection: true` and no `resolvedRequest`; pick exact `adIds` and preview again.
- `ads_duplicate_by_post({ ... })` with the resolved request to run it. The destination is always an existing `campaignId` (new campaigns are not supported), into an existing `adSetId`, one `newAdSet`, or `newAdSetPerAd`. Set `paused: true` to create the ads paused.
- If it returns `still_running`, poll `ads_get_duplication({ jobId })` until `complete` is true.

Provide either `postIds` (discovery) or ordered `adIds` (exact selection), never both. New ad sets clone targeting, budget, schedule and bid from the template ad set (`adSetId` if given, otherwise each source ad's own set). Dynamic-creative source ads reuse their creative ID and cannot preserve engagement; the preview lists them as warnings.

## Jobs

- `ads_get_job({ jobId, operations })` - get or resume an ad-creation job. `operations` is `recent` (default, last 20) or `full` (complete log). After `ads_create` returns `still_running`, poll this until `complete` is true.
- `ads_cancel_job({ jobId })` - cancel a running ad-creation job or a media ingest job (`ingest_...`).

## Tool Notes

- Local `ads_create` can watch for up to 30 minutes. Hosted calls return `still_running` after about 60 seconds while the server continues; resume with `ads_get_job`.
- `ads_upload` accepts local filesystem paths relative to the MCP server working directory. Hosted connectors should prefer public HTTPS URLs or `driveFolderUrl`, which are fetched server-side without sending bytes through the MCP transport. Public Drive file links in `files[].url` are detected automatically.
- Poll media-ingest jobs with `ads_get_upload({ jobId })` until `complete` is true. Several files may be downloading, staging, or processing concurrently in `progress.files`. The final upload summary is in `result`; cancel an unwanted import with `ads_cancel_job({ jobId })`. If `result.blocked` is true, report completed, failed, and skipped counts and tell the user to retry the Drive import later. `result.thumbnailsAttached` counts `*_thumbnail` images consumed as custom video thumbnails, which is why `result.total` can be lower than the number of submitted files - report it rather than treating the difference as a lost file.
- Filenames for `files[].url` sources always come from the remote source (Content-Disposition or the URL); `name` is only for base64 and presigned files and is rejected alongside `url`.
- Use inline base64 only for a couple of small images. Use `presigned:true` only from an environment that can PUT bytes itself.
- If credentials are missing or invalid, tools return: `Run 'ads login' in your terminal first.`
- After updating an existing saved build, confirm that the update was saved; an open uploader tab picks it up automatically. Share the build URL only if the user asks for it or says the tab is closed. For a newly created build with no open uploader tab, include the URL so the user can review it.
- Use `ads_fork_build` instead of updating in place when the user wants a variant and the original build must remain unchanged.

## CLI-only, not available via MCP

These exist in the `ads` CLI but have no MCP equivalent, so do not send them to MCP tools:

- `--json` output formatting and other CLI flag ergonomics. The MCP tools always return structured JSON; configuration is passed as JSON fields, not flags.
- Default-account config (`ads use`). MCP tools require an explicit `accountId` (except where a `buildId` supplies it).
- `creativeEnhancements` string/array shorthand (`"all"`, `"none"`, `"metaDefaults"`, `["feature1"]`). Via MCP, use the object form shown above.
- Local `paths` uploads work on the local stdio server but not on the remote/hosted MCP server; hosted callers use `files[].url` or `driveFolderUrl` instead.
