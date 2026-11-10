---
name: ads-mcp
description: Create, manage, and report on Facebook/Meta ads through the Ads Uploader MCP server. Query Ads Insights, upload media, preview ad specs, create ads, browse campaigns/ad sets/ads, and manage presets/uploads/builds/jobs.
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

For automations and servers, the hosted MCP also accepts `Authorization: Bearer adk_...`. Create a key under **Account > MCP, CLI & API**; each key is for one person. Signing in with OAuth remains the normal way to connect AI apps like Claude or ChatGPT.

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
| `ads_list_campaigns` | List campaigns in an account | `accountId`, `search`, `status` (`active`, `inactive`, `archived`, `all`), `fields`, `limit`, `offset`, `after`, `fresh` |
| `ads_list_adsets` | List ad sets in a campaign, or across the account without `campaignId` (find ad sets by name; rows include `campaignId`, `campaignName`) | `accountId`, `campaignId` (optional), `search`, `status` (`active`, `inactive`, `archived`, `all`), `fields`, `limit`, `offset`, `after`, `fresh` |
| `ads_list_ads` | List ads in an ad set, or across the account without `adSetId` (find ads by name; rows include `adsetName`, `campaignId`, `campaignName`). Name `reviewFeedback` / `issues` in `fields` for Meta's rejection reasons | `accountId`, `adSetId` (optional), `search`, `status` (`active`, `inactive`, `archived`, `all`), `fields`, `limit`, `offset`, `after`, `fresh` |
| `ads_get_ad` | Get one ad with its creative details; DISAPPROVED / WITH_ISSUES ads add `reviewFeedback` and `issues` (Meta's reasons) | `accountId`, `adId` |
| `ads_list_pages` | List Facebook Pages + connected Instagram | `accountId`, `limit`, `offset` |
| `ads_search_targeting` | Look up city keys and detailed-targeting IDs | `accountId`, `query`, `type`, `limit` |
| `ads_list_insight_fields` | Discover supported Insights fields and query options | `accountId`, `search`, `kind`, `group`, `ids`, `recommended`, `limit`, `offset` |
| `ads_get_insights_job` | Check or page an Insights report | `accountId`, `jobId`, `waitSeconds`, `after`, `limit` |
| `ads_query_insights` | Read bounded Ads Insights pages | `accountId` or `accountIds`, `compare`, `include`, fields, date, breakdown, filter, attribution, sort, paging inputs |
| `ads_list_presets` | List saved API or ad-text presets | `accountId`, `type`, `limit`, `offset` |
| `ads_get_preset` | Get one preset | `accountId`, `presetId` |
| `ads_save_preset` | Save an existing ad as a reusable preset | `accountId`, `fromAdId`, `name`, `shareWithTeam` |
| `ads_list_builds` | List saved uploader builds | `accountId`, `limit`, `offset` |
| `ads_save_build` | Save a durable build (spec and/or webState) | `accountId`, `name`, `notes`, `spec`, `webState` |
| `ads_get_build` | Get one build (full spec and webState by default) | `buildId`, `include` |
| `ads_fork_build` | Duplicate a build into a new draft | `buildId` |
| `ads_update_build` | Rename/patch/replace a build | `buildId`, `expectedRevision`, `name`, `notes`, `specPatch`, `spec`, `webState` |
| `ads_delete_build` | Delete a build | `buildId` |
| `ads_discard_launch` | Discard a launch waiting in the Agent Inbox; the build stays | `accountId`, `buildId` |
| `ads_list_uploads` | List recent media upload batches | `accountId`, `limit`, `offset` |
| `ads_get_upload` | Get a batch, or poll an ingest job | `batchId` (or its alias `uploadId`) or `jobId`, `compact` |
| `ads_upload` | Upload media (paths / URLs / Drive / base64 / presigned) | `accountId`, `paths`, `files`, `driveFolderUrl`, `presigned`, `script`, `inlineScript`, `batchId`, `cwd` |
| `ads_upload_refresh_urls` | Refresh expired upload links in the same batch | `accountId`, `batchId`, `files` (optional multipart `partNumbers`) |
| `ads_upload_finalize` | Finalize a presigned upload batch | `accountId`, `batchId`, `files`, `thumbnailMap` |
| `ads_preview` | Dry-run with `lifetimeBudget` when applicable and an always-present `warnings` array | same body as `ads_create` |
| `ads_create` | Create ads from a spec body or saved build | see "Create and preview inputs" below |
| `ads_get_job` | Get/resume an ad-creation job | `jobId`, `operations`, `compact` |
| `ads_cancel_job` | Cancel an ad-creation or ingest job | `jobId` |
| `ads_duplicate_preview` | Preview regular duplication (the default), including tree, names and overrides, requiring explicit daily ad set budgets for CBO-to-ABO copies | `accountId`, `level`, `ids`, optional destination, rename, status, childStatus, overrides |
| `ads_duplicate` | Submit a previewed regular duplication; returns `pending_approval` with `reviewUrl` under Launch Approval, otherwise a copy job | `resolvedRequest` plus approved confirmation flags |
| `ads_duplicate_by_post_preview` | Preview social-proof duplication only when keeping engagement was requested | see "Duplicate by post ID" |
| `ads_duplicate_by_post` | Duplicate ads keeping engagement via the same Page post; only when explicitly requested | see "Duplicate by post ID" |
| `ads_get_duplication` | Get/resume a duplication job | `jobId` |

Tool inputs follow the public `/api/v1/` contracts. Campaign, ad set, ad and resource reads account cursor paging as described under Resource reads. Existing Page, preset, build and upload browse tools retain offset paging. The Insights field catalog uses offset paging with a default limit of 100; Insights queries use opaque `paging.after` cursors.

## Create and preview inputs

`ads_create` and `ads_preview` accept the same request body. Preview first, create when it looks right. You must supply media one of three ways, then layer the ad configuration on top.

### Required: pick a source

- `accountId` + `uploadId` - create from a completed upload batch.
- `accountId` + `mediaItems` - create from portable saved-build media metadata (images resolve by `mediaHash`, videos by `mediaId`; missing library-video thumbnails are recovered from Meta during preview/create).
- `buildId` - create from a saved build's portable spec. Runtime `accountId` is taken from the build unless you pass it.

Fields:

- `accountId` (string) - ad account ID, e.g. `act_123`.
- `uploadId` (string) - batch ID from `ads_upload` / `ads_get_upload`.
- `buildId` (string) - saved build ID; its spec becomes the create body.
- `mediaItems` (array) - saved-build media metadata; account-bound, so `accountId` must match the account the media was captured under.

### Reuse an existing ad or preset

Presets reuse their saved campaign and ad set when no destination is supplied, including preset-only creates and older saved builds. Preview and create return a non-fatal `PRESET_DESTINATION_USED` warning identifying the saved destination. To choose a different destination, pass `campaign.name` for a new campaign, named `campaign.campaigns` entries in duplicate/split mode, or `campaign.id` / `adSet.id` for an existing destination. With only an explicit `campaign.id`, new ad sets are created. With `copyFromAd` and no `campaign` or `adSet`, a new paused campaign and ad set are created from the template ad's settings; the template's own campaign and ad set are never used unless you pass their IDs.

Cross-campaign create/preview is available only for direct agent requests. Saved builds and Agent Inbox launches must use the template ad's campaign or a new campaign; to save a build for another existing campaign, copy from an ad in that campaign (campaign.id + adSet.id picks one). Existing stored cross-campaign builds can still launch, but cannot be re-saved.

- `copyFromAd` (string) - existing Meta ad ID to copy settings from. Mutually exclusive with `adPresetId`.
- `adPresetId` (string) - saved API preset settings. Mutually exclusive with `copyFromAd`. Without a destination it reuses the preset's saved campaign and ad set (`PRESET_DESTINATION_USED` warning).
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

To remove a copied headline, primary text or description, send an empty list (`[]`) for that field. Omitting the field, or `[""]`, keeps the copied text. This applies to `texts.common`, `texts.perAd`, `texts.perAdset` and `texts.perAdSet`.

Use canonical arrays `headlines`, `bodies`, and `descriptions`. For uploaded media, string aliases `headline`, `primaryText` (or `body`), and `description` are accepted. Legacy aliases `title` / `titles` → `headlines` and `linkDescription` → `descriptions` accept strings or string arrays; `websiteUrl` → `link` accepts a string. These aliases work in common, per-ad and per-ad-set text blocks. Unknown text keys, invalid types, an alias with its canonical field, and multiple aliases for the same field are rejected.

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

A carousel takes one headline and one description. Set per-card text with `cards[].cardText` or `cardTexts`; several primary texts (`bodies`) are allowed. Multiple own headlines or descriptions warn at preview/save and are refused at create. Inherited variants warn and use only the first on carousel cards. When every card has a size pair (for example `_4x5` + `_9x16`), the carousel launches as a placement-variant carousel, as it does in the uploader, and uses only its first primary text; preview warns with `CAROUSEL_PRIMARY_TEXT_LIMITED`.

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

Each group: `name`, `assets` (2-10 media references), optional `assetTexts` lined up with `assets` by index (each field a single per-asset override; blank fields fall back to the ad's main text). Videos use a public thumbnail URL, recovered from Meta when missing.

Each aspect-ratio size is sent as its own entry, and every placement goes to the size the single-image and video placement rules pick, including the account's saved placement priorities (for example 4x5 for Feeds, 9x16 for Stories, Reels and Audience Network, and 1x1 for Marketplace, Search and Right Column). Sizes do not count toward the limit of 2-10 visuals.

### `creativeEnhancements` - Advantage+ creative features

Via MCP, pass this as an **object** (the string shorthand `"all"`/`"none"`/`"metaDefaults"` and bare feature arrays are CLI-only):

```json
{ "creativeEnhancements": { "useMetaDefaults": false, "features": ["text_translation", "image_touchups"] } }
```

- `{ "useMetaDefaults": true }` - let Meta decide (this is also what you get if you omit `creativeEnhancements` entirely).
- `{ "useMetaDefaults": false, "features": [...] }` - turn on the listed enhancements, other enhancements off; inherit source formats.
- `{ "text_translation": true, "enhance_cta": false }` - explicit per-feature on/off (any feature key is accepted).

Available features: `text_translation`, `inline_comment`, `enhance_cta`, `text_optimizations`, `reveal_details_over_time`, `show_destination_blurbs`, `image_brightness_and_contrast`, `image_touchups`, `video_auto_crop`, `video_filtering`, `image_animation`, `image_templates`, `adapt_to_placement`, `product_extensions`, `description_automation`, `add_text_overlay`, `music`, `carousel_to_video`, `carousel_dynamic_description`, `multi_share_end_card`, `multi_share_optimized`. Only list features relevant to the media type (`video_auto_crop`/`video_filtering` are video-only; `carousel_*` are carousel-only). `product_tags` only works when the source ad has a catalog with positioned product tags.

`show_destination_blurbs` is displayed as **Show spotlights**.

Source format display options inherit even with an empty or omitted `features` list. To turn one or both off, use `{ "useMetaDefaults": false, "features": [], "excludedFormats": ["format_carousel", "format_collection"] }`. `format_carousel` is Show as carousel and `format_collection` is Show as collection. Omit `excludedFormats` to inherit. A per-feature object such as `{ "format_carousel": false }` also works. Excluded formats must be cloneable on the source (catalog-backed formats require a catalog), cannot also be enabled in `features` or the per-feature object, and are unsupported for existing-post creatives. Non-arrays and unknown IDs are rejected. Formats cannot be enabled if the source lacks them.

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
- `schedule.startTime` / `schedule.endTime` (strings) - scheduled start/end. Account-local `YYYY-MM-DDTHH:mm` or `YYYY-MM-DDTHH:mm:ss`, or ISO 8601 with an offset or `Z` (converted to the account time zone).
- New campaigns/ad sets with lifetime budgets are blocked at create with HTTP 400; preview marks these warnings `blocking: true`: `LIFETIME_BUDGET_END_MISSING`, `LIFETIME_BUDGET_END_PASSED`, `LIFETIME_BUDGET_END_BEFORE_START`, `LIFETIME_BUDGET_END_TOO_SOON` can be fixed with `options.schedule.endTime` at least 24 hours after the start or by using a daily budget. `EXISTING_AD_SET_ENDED` and `EXISTING_AD_SET_ENDS_SOON` stay non-blocking and require extending the existing ad set in Ads Manager.

### Saved-build metadata (ignored on create)

`launchable` and `launchBlockers` may appear on a spec read back from a saved build. They are metadata only and are ignored when creating ads; do not set them by hand.

## Browsing and lookup

- `ads_list_campaigns` / `ads_list_adsets` / `ads_list_ads` - each takes `accountId` (and optionally `campaignId`/`adSetId`; omit it to list or search ad sets and ads across the whole account), plus `search` (name text), `status` (`active` by default; other values mean no filter for campaigns and parent-scoped lists; account-wide lists also support `inactive` and `archived`), optional `fields`, `limit` (1-100, default 25), `after`, `offset`, and `fresh`. Use fields to return only the columns you need; large pages are trimmed to fit the output budget. Follow `pagination.nextOffset` with the same input `after` to consume trimmed rows before advancing the cursor (see Resource reads below).
- `ads_get_ad({ accountId, adId })` - one ad with full creative details. Handy to find an ad ID for `copyFromAd` or `ads_save_preset`. For a DISAPPROVED or WITH_ISSUES ad it also returns `reviewFeedback` (`global` and `placementSpecific` lists of `{ policy, reason }`) and `issues` (`errorCode`, `errorSummary`, `errorMessage`, `level`); both are null when Meta's reasons could not be read.
- `ads_list_pages({ accountId })` - Facebook Pages plus their connected Instagram accounts, for `profile` overrides.
- `ads_search_targeting({ accountId, query, type, limit })` - `type` is `city` or `detailed`; `query` needs 2+ characters; `limit` is 1-25 (default 8). The only correct way to get city keys and detailed-targeting IDs.
- `ads_list_presets({ accountId, type })` - `type` is `api` (default) or `adText`. `ads_get_preset({ accountId, presetId })` reads one. `ads_save_preset({ accountId, fromAdId, name, shareWithTeam })` saves an existing ad as a reusable API preset (`shareWithTeam` default false).

## Performance reporting

Discovery without filters returns groups and recommended fields. Use `search`, `group`, `kind`, or `ids` for full entries. Optional `accountId` adds custom conversion definitions and names, refreshed after 24 hours.

Use friendly metrics (`leads`, `purchases`, `cost_per_purchase`, `roas`) for business questions. Use `results`/`cost_per_result` to assess optimization goals at campaign, ad set or ad level. Missing native results can be derived from ad set setup and delivery, with `result_source: "derived"` and `result_indicator`; incomplete data stays unknown. Custom conversion fields are `custom_conversion.<id>`, `custom_conversion_value.<id>`, and `cost_per_custom_conversion.<id>`, using digits for the ID. See the [friendly metric formulas](https://adsuploader.com/docs/automation/insights#friendly-metrics-and-campaign-goals). Omitted counts and values are 0 with delivery (impressions > 0) and null otherwise; a positive count without an action value stays null. Native result indicators with empty values are 0 on delivered rows; cost per result stays null for zero results. Ratios with missing or zero denominators are null.

Queries and job reads default to `format: "table"`: row arrays follow `columns` order. Use `format: "objects"` for keyed rows and to distinguish absent keys from explicit nulls. Formatting shares the same cache. Native results preserve indicator and attributed-value indexes, such as `results.0.indicator` and `results.0.values.0.value`.

Use `where` for up to 5 typed `{ field, operator, value }` filters; `excludeZeroImpressions` defaults to true; false includes zero rows. Comparisons test delivery in either period after joining. Numeric comparisons use `gt/gte/lt/lte/eq/neq`; numeric and name membership use `in/not_in` arrays. Names also allow `contains/not_contains/starts_with`. Effective statuses use `in/not_in`; campaign objectives use `in`.

Use asset breakdowns to analyze flexible/dynamic creative. Asset, media, and hourly breakdowns must each be used alone. Object assets flatten into string columns such as `title_asset.text`. Non-default `actionBreakdowns` return separate `actionColumns` and `actionRows` with a zero-based `row` index and `family`; do not combine them with friendly aliases. Both tables count toward the cell and output budgets. A truncated page must be re-requested at a smaller limit before following its cursor.

Use `ads_list_insight_fields` whenever a field ID, action family, date preset, or breakdown is uncertain. Breakdowns belong in `ads_query_insights.breakdowns`, never in `fields`.

`ads_list_insight_fields({ accountId?, search?, kind?, group?, ids?, recommended?, limit?, offset? })`:

- `search` (string) - text matched against field IDs, labels, kinds, and groups.
- `kind` (string) - `dimension`, `metric`, `action_family`, `alias`, `breakdown`, or `date_preset`.
- `group` (string) - exact catalog group such as `delivery`, `conversions`, `video`, `identity`, `breakdown`, or `date`.
- `recommended` (boolean) - when true, return the recommended catalog entries.
- `limit` (integer, 1-100, default 100) - field-catalog page size.
- `offset` (integer, minimum 0, default 0) - field-catalog page offset.

`ads_query_insights({ accountId, ...query })` returns a pending job or `meta`, dynamic `columns`, normalized `rows`, and cursor `paging`:

- `accountId` (string, required) - ad account ID, e.g. `act_123`.
- `mode` (string) - `auto` (default), `sync`, or `async`.
- `fresh` (boolean) - defaults to false; true refreshes the query cache.
- `level` (string) - `account` (default), `campaign`, `adset`, or `ad`.
- `fields` (array, 1-30) - Native metric, dimension, or friendly alias IDs. Request only needed columns.
- `actionBreakdowns` (array) - defaults to action_type; other dimensions produce separate action rows.
- `where` (array) - up to 5 typed value filters.
- `excludeZeroImpressions` (boolean, default true) - exclude zero-impression rows; false includes them. Compare retains delivery in either period.
- `format` (string) - table (default) or objects.
- `actionTypes` (array) - Meta `action_type` values used to select and flatten action-family fields such as `actions` or `cost_per_action_type`.
- `datePreset` (string) - a supported Meta date preset. Mutually exclusive with `timeRange`; defaults to `last_30d` when neither is supplied.
- `timeRange.since` / `timeRange.until` (`YYYY-MM-DD`) - inclusive explicit range in the account timezone. Mutually exclusive with `datePreset`.
- `timeIncrement` (string) - `all_days` (default), `daily`, `weekly`, or `monthly`. Daily ranges are limited to 90 days in sync mode or 400 in auto/async mode.
- `breakdowns` (array, maximum 2) - allowlisted dimensions discovered with `ads_list_insight_fields({ kind: "breakdown" })`.
- `filters.campaignIds` / `filters.adSetIds` / `filters.adIds` (arrays) - entity ID filters using Meta `IN` matching.
- `attribution.useAccountSetting` (boolean) - defaults to true and cannot be true with explicit windows.
- `attribution.windows` (array) - explicit action attribution windows such as `7d_click` and `1d_view`. Friendly metrics such as `purchases` and `roas` report the one window requested; several windows with friendly metrics are refused (`ALIAS_WITH_ATTRIBUTION_WINDOWS`), so request native `actions` or `action_values` with `actionTypes` to compare windows side by side.
- `attribution.reportTime` (string) - `conversion`, `impression`, `mixed`, or `lifetime`.
- `sort.field` / `sort.direction` - sort one native scalar field by `asc` or `desc`; action-family fields are not sortable.
- `limit` (integer, 1-100, default 25) - maximum rows in this Meta page.
- `after` (string) - opaque `paging.after` cursor from the previous response. Keep the query unchanged and use it only when another page is needed.

Example prompts:

- "Which ads spent the most in the last 7 days in account act_X?"
- "Daily CPA for campaign 1201 over the last 30 days."
- "Placement breakdown for ad set 55 over the last 14 days."
- "Compare last week with the week before."

One tool call returns one bounded page per account. Reach, frequency, and CTR are non-additive, so never sum them across rows. Empty rows are a successful response. Zero-impression rows are excluded by default (`excludeZeroImpressions: true`); set `excludeZeroImpressions: false` or use CLI `--include-zero-impressions` to include them. Comparisons keep a row when either period has impressions above zero, so drops to zero remain visible. If a result warns that rows were truncated for the MCP output budget, rerun the same page with fewer fields or a smaller limit before following its cursor. On a 429, report the Retry-After wait and do not retry in a loop.

Use `compare: { type: "previous_period" }` for "vs last period", `previous_year` for last year, or `{ type: "custom", timeRange: { since, until } }` for explicit prior dates. Comparison requires `all_days`, sync execution, a bounded preset/range, and default action breakdowns. Previous-year dates clamp Feb 29 to Feb 28. Each numeric metric gets adjacent `.previous`, `.change`, and `.change_pct` columns. Missing matches remain null; a zero prior value yields a null percentage. `meta.compare` records the ranges. `COMPARE_PARTIAL` means prior coverage was bounded at 500 rows and three cursor pages.

Use `include: ["status", "budget", "creative"]` for context instead of separate list calls. With a non-empty `include`, the default limit is 10 rows; otherwise it remains 25. Explicit limits are unchanged. Tool results use compact JSON, and Insights and resource lists share a 40 KB output budget. Other categories are `schedule`, `setup`, and `targeting`. `entities` holds campaign, ad set, and ad maps; `meta.entityRefs` links them to each row by index, including table format. Parent budgets use major currency units and keep raw `<field>_minor` strings. Targeting and dynamic creative assets are summaries. Targeting summaries use `geo.countries`, `country_groups` (`{ key, label }`, including European Economic Area (EEA)), region and geo-market names, cities with radius/unit, zip and custom-location samples, and `location_types`. The same fields appear under `geo.excluded`. Lists contain at most 25 items with full counts (`country_count`, `country_group_count`, `region_count`, `city_count`, `geo_market_count`, `zip_count`, `custom_location_count`, `location_type_count`); `truncated: true` on the containing object marks a capped list. Additional geo dimensions are retained. `flexible_spec` preserves each detailed-targeting group with capped interest, behavior and demographic names, per-list counts, `flexible_spec_count` and truncation flags. Custom audience lists are capped and counted too. Metadata has its own 10-minute cache; missing entities are null with `ENTITY_METADATA_UNAVAILABLE` while performance rows remain available.

Requested compare fields are retained, including missing native `results` and `cost_per_result`. Friendly formulas are evaluated independently for each period. Native result values match by indicator and attribution window, preserving their expanded column paths. A field without comparable numeric values retains its current value and is listed in `COMPARE_FIELD_UNSUPPORTED`.

The cell estimate uses `min(limit, row upper bound)`: account level without breakdowns or time buckets is one row per account. Time buckets and known bounded breakdowns increase that bound; unknown breakdown sizes and entity levels use the page limit. The actual output still cannot exceed 2,000 cells.

Use `accountIds: ["act_123", "act_456"]` instead of `accountId` for 2 to 5 distinct accounts. Execution is sequential and synchronous, including per-account comparisons. Never add totals across accounts. Rows lead with `account_id` and `account_currency`; `meta.accounts` carries each account's currency, timezone, cache, warnings, paging, comparison ranges, and safe errors. `MIXED_CURRENCIES` flags different currencies. `PARTIAL_RESULTS` means some accounts failed; respect their retry waits and query them separately. If all accounts fail, return the error, preferring 429 for a throttle. Query heavy accounts separately rather than requesting async multi-account reports.

Multi-account pages use `paging.byAccount`. Keep the query unchanged and send `after: { act_123: "CURSOR", act_456: null }`; null marks an exhausted account, while an omitted key starts its first page. Keys use canonical `act_` IDs. `limit` is per account within the combined 2,000-cell bound. Comparison columns count toward the bound. If MCP truncates output, rerun that same page with fewer fields or a smaller limit before following a cursor; returned rows retain their complete entity metadata.

For single-account queries without comparison, `mode` is `auto` by default. Daily ranges over 90 and up to 400 days, or Meta "too much data" responses, become async jobs. `sync` preserves the 90-day daily cap; `async` always submits a report. The field catalog publishes modes and limits.

A `status: "pending"` response means call `ads_get_insights_job({ accountId, jobId: job.id, waitSeconds: 15 })`. Never repeat the query to poll. The job tool accepts `format`, `after`, `limit` (1-100), and `waitSeconds` (default 15, maximum 20). Running results include progress and `pollAfterSeconds`. Completed results use the normal rows/columns/paging contract with `meta.job`. Continue pages using this job tool and the same job ID. References expire after 24 hours.

Cached results include `meta.cache.ageSeconds` and `meta.cache.ttlSeconds`. Query cache lifetimes are 10 minutes for ranges including today, 15 minutes for 1 to 3 days ago, 1 hour for 4 to 28 days ago, and 24 hours for older ranges. Completed job pages use a 1-hour cache. Pass `fresh: true` to refresh a query instead of reading its existing cached result. `429 INSIGHTS_ACCOUNT_BUSY` means the account's concurrent request cap is full, with `Retry-After: 5`. Surface the wait rather than retrying in a loop.

## Saved builds

Builds are durable, editable launch configurations. Compare saved display totals, per-ad copy, budgets and status with preview before creating. Read `WEB_EDIT_UNSUPPORTED` warnings: the web keeps per-ad-set budgets but cannot edit them. A spec-only build is preferred for agent-created work: the web uploader hydrates it and generates its own `webState` on resume.

- `ads_save_build({ accountId?, name?, notes?, spec?, webState? })` - provide at least `spec` or `webState`. `accountId` can be derived from `webState.selectedAccount` or `spec.accountId`. `name` is up to 120 chars; `notes` up to 2000. Do not construct `webState` by hand.
- `ads_get_build({ buildId, include? })` - full `spec`, untruncated. `webState` is included by default; request `include: ["-webState"]` to omit the opaque uploader snapshot. Read `revision` here before patching.
- `ads_fork_build({ buildId })` - duplicate a build into a new autosaved draft; use this when the user wants a variant and the original must stay intact.
- `ads_update_build({ buildId, expectedRevision?, name?, notes?, specPatch?, spec?, webState? })`:
  - `specPatch` (object) - preferred for content edits. A field-aware merge: omitted fields survive, nested objects (including individual `texts.perAd`/`perAdset` entries) merge, supplied arrays replace, and `null` removes a field. So changing headlines does not wipe `link`, `displayUrl`, CTA, URL tags, or `aiDisclosure`. Requires `expectedRevision` from `ads_get_build`.
  - `spec` (object or null) - full replacement of the portable spec; send the entire object, not a diff. `null` clears it. Do not send both `spec` and `specPatch`.
  - `expectedRevision` (integer) - optimistic-lock guard; required with `specPatch`, recommended on every update so a stale write fails instead of clobbering a newer revision.
    - Send edits to a build one at a time, never in parallel. Changes to `spec`, `webState`, `mediaItems` or `uploadBatchId` bump the revision. A rename or notes-only update does not bump it. Batch many ad edits into one `specPatch` where you can. On "Build changed since it was read" (HTTP 409), call `ads_get_build` again and retry with its current revision as `expectedRevision`.
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
- `script` (boolean) - with `presigned:true`, return a short-lived `scriptUrl` and `expiresAt`; recommended for agents with a shell. Run `curl -fsS -o upload.sh '<scriptUrl>' && sh upload.sh <folder>`. It requires curl, dd, and standard POSIX file utilities (wc, mkdir, rm), verifies local sizes, retries transfers, and runs up to four transfers at once. URLs live in the script; `uploads` retains only finalize and refresh metadata.
- `inlineScript` (boolean) - requires `script:true`; returns `uploadScript` inline when a client cannot fetch downloads. Storage failures also fall back to inline, with a short `scriptDelivery` explanation.

The script prints `OK <name>`, `EXPIRED <name> [parts...]`, or `FAILED <name> <http code>` per file and exits nonzero if any file is not OK. If a PUT returns 403 / its link expired, call `ads_upload_refresh_urls({ accountId, batchId, files })` for only those files or parts; do not re-upload files that already succeeded. If `ads_upload_refresh_urls` is not in your tool list, re-run `ads_upload` with `presigned=true` for only the failed files and the same `batchId`, then finalize the whole batch; already-uploaded files keep their `fileKey`s. A 403 can also mean the bytes sent differ from the signed size; the file size must match the signed size exactly (upload the real file, not a symlink or a re-encoded copy). Each refresh file needs `name`, `size`, and `contentType`. Multipart files also need the original `fileKey`, `uploadId`, and `partSize`; supply `partNumbers` to refresh only the expired parts, or omit it for all parts. Re-run only the refreshed transfers. Single-PUT refreshes may change `fileKey`; use the new key for finalization and update any thumbnail mapping that references it.

`ads_upload_finalize({ accountId, batchId, files, thumbnailMap? })` completes a presigned batch after the client has PUT each file, preserving `name`/`type`/`size`/`contentType`/`fileKey` (and `uploadId` for multipart) from the latest returned metadata. Signed URLs are optional for finalization. `thumbnailMap` optionally maps video keys to thumbnail keys.

Finalizes queue automatically behind an active import. When `status` is `queued`, keep polling the returned `jobId`; do not resubmit. `queuePosition: 1` means next to start, and `activeJobId` identifies the current import.

Poll ingest jobs with `ads_get_upload({ jobId })` until `complete` is true. Job polls return the full `progress`, `result` and file list by default. Use `compact:true` for counts, queue information, errors and failed files only, or `ads_get_upload({ batchId })` for the full completed batch. Cancel an unwanted import with `ads_cancel_job({ jobId })`.

## Which duplicate tool?

Agent duplications from MCP, CLI, or the public API follow the account’s Agent Launch setting on the Account Permissions page, the same as launches; Agent Edit does not apply. With Launch Allowed (the default), the job starts immediately; regular ACTIVE copies (`ads_duplicate`) require `confirmSpend: true` when the approval rule is not `none`, after explicit user approval, and post-ID copies never require it. With Launch Approval, submission returns `mode: "review"`, `status: "pending_approval"` and `reviewUrl`. Nothing has been duplicated yet: share the link and wait for the user to approve in your Agent Inbox (`/dashboard/inbox`). Do not retry or poll: there is no job until approval. With Launch Off, the request is refused with `AGENT_LAUNCH_DISABLED`. Admin-only `testMode: true` is preserved through approval and simulates the job without live Meta writes. Uploads that create ads from new media and the human-operated web Duplicator are unaffected. Each post-ID copy can be approved or rejected on its own; `ads_get_edit_review` / `ads edit:review` report `copies[].status` (`pending`, `approved`, `made`, `failed`, `rejected`). For post-ID requests, `group` links proposals split from one request, and `summary.copies` totals copies across the whole review.

`ads_duplicate_preview` and `ads_duplicate` duplicate campaigns, ad sets or ads the same way as Ads Manager's Duplicate. This is the default when a user asks to duplicate, copy or clone. Campaigns and ad sets include children by default (`deepCopy: false` copies only the selected level). Copy roots stay **PAUSED** by default; children keep their source status (`childStatus: "INHERITED"`). Preview warns how many ACTIVE children will deliver when the root is activated. Use `childStatus: "PAUSED"` to pause every copied child. Destinations stay within the same account, and `{name}`, `{date}` (UTC) and `{index}` control names. Only the top level is renamed unless `renameChildren: true`.

Preview first, show the tree, overrides, skipped objects and warnings, then submit its `resolvedRequest` including `sourceFingerprint`. Set `acknowledgeWarnings: true` after approval; when Launch is Allowed, ACTIVE copies also need `confirmSpend: true` unless the approval rule is `none`. Increasing a budget on an ACTIVE copied object also needs `confirmSpend` under `budget_and_going_live` or `all_live_changes`, including inherited ACTIVE children. `SOURCE_CHANGED` means material source settings, the child tree, destination or intent changed: preview again and obtain approval for the new preview. Reviewed copies require a new approved proposal. Volatile delivery metadata does not invalidate the preview. With Launch Approval, the result is `pending_approval`; share `reviewUrl` and wait for web approval; do not retry the request. Launch Allowed starts a job. Hosted tools poll for about 40 seconds, then return `still_running`; use `ads_get_duplication` until complete and inspect every `created`, `failed` and `skipped` entry. Never retry a partial job blindly.

Overrides use the same targeting, budget, schedule, bid and ad set settings inputs as edits. They apply to copied ad sets; when copying a CBO campaign, budgets and `bid.bid_strategy` apply to the copied campaign. Ad set copies accept `overrides.budget` and `bid.bid_strategy` when the destination campaign uses ad set budgets (ABO), including copies from a campaign-budget (CBO) campaign. CBO-to-ABO ad set copies require `overrides.budget.daily`; preview returns `ADSET_BUDGET_REQUIRED` without a budget. Meta gives these copies a daily budget, so only a daily budget override is supported. Switching a copy between daily and lifetime budgets is refused at preview with `BUDGET_TYPE_CHANGE_UNSUPPORTED`, including ad sets inside deep campaign copies and CBO campaign copies. Source CBO ad sets have no budget of their own, and the source campaign budget is never carried over. For CBO-to-ABO copies, `overrides.bid.bid_strategy` is required when the source campaign has a capped or minimum-ROAS strategy, or the source ad set has a bid amount or constraints. Omitting the strategy in those cases returns `ADSET_BID_STRATEGY_REQUIRED`. Uncapped sources without those settings default to `LOWEST_COST_WITHOUT_CAP`. An explicit strategy takes precedence: capped strategies require a source or override bid amount, and minimum ROAS requires VALUE optimization plus a source ROAS floor or `overrides.bid.roas_floor`. Updates omit bid amounts when the chosen strategy does not allow them. Lifetime budget overrides on existing lifetime-budget ad set sources still require a future end time from the source ad set or `overrides.schedule.end_time`. A CBO campaign shell (`deepCopy: false`) accepts budget and uncapped strategy overrides. Capped strategies require `deepCopy: true` with compatible source ad sets: bid amounts and ROAS settings live on ad sets. Preview validates the copied children; it never changes existing destinations. `targeting.add.countries` adds countries; to target only the US, add US and remove the other source countries and locations. Ad-only copies cannot override an existing destination ad set. Each plan supports up to 50 selected sources and 500 copied objects. Admin-only `options.testMode` produces a simulated job with zero Meta writes.

Preview includes `executionPlan` with copy, override, rename and activation call counts. Copies run serially, object by object, so Meta's deep-copy child limits (3 synchronous / 51 async) do not apply. Large copies run as a background job; poll for progress. Flexible/dynamic creatives, partnership/branded-content ads, Instagram-post ads and catalog ads have grouped warnings with source IDs and warning codes on each affected tree node. They may not copy exactly; review the copied ads before activating.

`ads_duplicate_by_post_preview` and `ads_duplicate_by_post` duplicate ads while keeping their social proof (likes, comments, shares) by reusing the same Page post. **Only use these when the user asks to keep engagement; for regular duplication use `ads_duplicate`.** Dynamic-creative fallback warnings still apply. `ads_get_duplication` covers both kinds of job.

## Duplicate by post ID

Copy existing ads by their Page post ID, preserving the original post's engagement (likes, comments, shares). This is the web Duplicator's behavior, headless.

- `ads_duplicate_by_post_preview({ accountId, postIds, campaignId, ... })` first. It resolves each `postIds` entry to a specific ad (discovery) and returns the source-to-destination plan. If a post matches more than one ad it returns `sourceCandidates` with `requiresSourceSelection: true` and no `resolvedRequest`; pick exact `adIds` and preview again.
- `ads_duplicate_by_post({ ... })` with the resolved request to start a job (or submit it for approval when Agent Launch is Approval). The destination is always an existing `campaignId` (new campaigns are not supported), into an existing `adSetId`, or one `newAdSet` / `newAdSetPerAd` that uses `adSetId` as its template. `adSetId` is always required. Set `paused: true` to create the ads paused.
- If it returns `still_running`, poll `ads_get_duplication({ jobId })` until `complete` is true.

Provide either `postIds` (discovery) or ordered `adIds` (exact selection), never both. New ad sets clone targeting, budget, schedule and bid from the template ad set (`adSetId`, an ad set in the destination campaign; the source ad's own ad set is never used). Dynamic-creative source ads reuse their creative ID and cannot preserve engagement; the preview lists them as warnings.

## Jobs

- `ads_get_job` preserves `jobId`, server `status`, `complete`, `lastUpdated`, `error`, `result` (or null), and the original `progress` fields and stage-based meaning by default. `progress.operations` includes the last 20 entries; `operations:"full"` returns all retained entries. Pass `compact:true` to opt in to an ads summary with `created`/`failed`/`total`, one `ads` list, and progress measured in ads with separate stage counts. Compact running jobs include a `nextStep` hint. After `ads_create` returns `still_running`, poll until `complete` is true.
- `ads_cancel_job({ jobId })` - cancel a running ad-creation job or a media ingest job (`ingest_...`).

## Tool Notes

- Local `ads_create` can watch for up to 30 minutes. Hosted calls return `still_running` after about 40 seconds while the server continues; resume with `ads_get_job`. If a call times out or drops without a reply, do not create a new launch: within 10 minutes, call ads_create again with the exact same arguments (an identical repeat returns the existing job instead of creating duplicates; pass the same idempotencyKey to extend this to 24 hours), otherwise poll the known jobId with ads_get_job or check ads_list_history. Set options.allowRepeatLaunch only when the user explicitly wants the same ads created again.
- With Agent Launch set to Approval, `ads_create` launches nothing: it returns `status: "approval_required"` with `url` and `buildId`, and the launch waits in the user's Agent Inbox. Optional `runTitle`, `reviewId` and `reason` mean what they mean for edit previews and copies: pass the `reviewId` from an earlier `ads_edit_preview` or `ads_duplicate` (or reuse its `runTitle`) to hold the launch in that run, so the user reviews the launch beside your changes; the changes and the launch run separately. The result then includes that `reviewId`. A `reviewId` from another user or ad account is refused. When Launch is Allowed these fields are ignored.
- `ads_preview` reports `launch.mode`: `allowed` sends a subsequent create straight to Meta; `approval` holds it for approval. With `allowed`, show the `LAUNCH_GOES_LIVE` warning: ACTIVE objects can start spending immediately, while PAUSED objects are created paused on Meta without review. Set `options.status: "PAUSED"` or change Launch to Approval on Account Permissions to review first. The approval rule shown by an edit preview applies to edits, not new-ad launches. `reviewId`, `runTitle` and `reason` only take effect when Launch is Approval; supplying them with Allowed returns `IGNORED_FIELD` with the ignored names and does not add the launch to the Agent Inbox review. CLI `ads create:preview` and `ads create` print these warnings; dispatched `ads create --json` emits a `warnings` event before polling. Test mode reports ignored run fields without claiming a live launch.
- `ads_upload` accepts local filesystem paths relative to the MCP server working directory. Hosted connectors should prefer public HTTPS URLs or `driveFolderUrl`, which are fetched server-side without sending bytes through the MCP transport. Public Drive file links in `files[].url` are detected automatically.
- Poll media-ingest jobs with `ads_get_upload({ jobId })` until `complete` is true. Default polling includes `progress.files` and the final summary in `result`. Use `compact:true` for counts and failed files only; cancel an unwanted import with `ads_cancel_job({ jobId })`. If the full result (`compact:false`) has `result.blocked` set, report completed, failed, and skipped counts and tell the user to retry the Drive import later. `result.thumbnailsAttached` counts `*_thumbnail` images consumed as custom video thumbnails, which is why `result.total` can be lower than the number of submitted files - report it rather than treating the difference as a lost file.
- Filenames for `files[].url` sources always come from the remote source (Content-Disposition or the URL); `name` is only for base64 and presigned files and is rejected alongside `url`.
- Use inline base64 only for a couple of small images. Use `presigned:true` only from an environment that can PUT bytes itself.
- If credentials are missing or invalid, tools return: `Run 'ads login' in your terminal first.`
- After updating an existing saved build, confirm that the update was saved; an open uploader tab picks it up automatically. Share the build URL only if the user asks for it or says the tab is closed. For a newly created build with no open uploader tab, include the URL so the user can review it.
- Use `ads_fork_build` instead of updating in place when the user wants a variant and the original build must remain unchanged.

## CLI-only, not available via MCP

These exist in the `ads` CLI but have no MCP equivalent, so do not send them to MCP tools:

- `--json` output formatting and other CLI flag ergonomics. The MCP tools always return structured JSON; configuration is passed as JSON fields, not flags.
- Default-account config (`ads account`). MCP account-scoped tools require an explicit `accountId` except where a `buildId` supplies it or `ads_query_insights` uses `accountIds`; `ads_list_insight_fields` accepts optional account scope for custom conversions.
- `creativeEnhancements` string/array shorthand (`"all"`, `"none"`, `"metaDefaults"`, `["feature1"]`). Via MCP, use the object form shown above.
- Local `paths` uploads work on the local stdio server but not on the remote/hosted MCP server; hosted callers use `files[].url` or `driveFolderUrl` instead.

## History

Use `ads_list_history` for launches and changes made through Ads Uploader: web Uploader, Duplicator, AI Edits, CLI, MCP and API. It returns the same History page rows, newest first, with readable before/after values and status labels. It requires `accountId`. Optional filters are `days` (7, 30 or 90; default 30), `type` (`all`, `launches` or `changes`; default `all`), `source` (`web`, `duplicator`, `mcp`, `cli` or `api`), `q` (up to 200 characters), and `member` (a 24-character team member ID). Use `limit` (1–100, default 50) and `paging.before` as the next `before` value, keeping the same filters.

`ads_get_launch` takes `accountId` and a `launchId` from those rows and returns one row per ad (ID, name, status, ad set, campaign, result and error), plus totals and failure messages. Pass `detail: 'full'` for targeting, budgets, schedule, creative settings and media. Both tools are read-only. The History page is open to everyone; reading History through the API, CLI and MCP is in beta, and hosted MCP hides these tools without beta access.

`ads_list_inbox_history` reads retained Agent Inbox suggestions and direct edit outcomes from the last 90 days, newest first, without calling Meta. It returns agent notes, run titles, client names, before/after values and outcomes. Filter by `accountId`, `campaignId`, `since`/`until`, `reviewId` (one run), `status`, `kind` or `action`. Without `accountId` it uses the selected account or, if none is selected, all permitted accounts. Use `limit` (1–100, default 50) and `paging.cursor` as the next `cursor` value, keeping the same filters. Read permission is required and hidden accounts are excluded. It is in the same beta.

`ads_list_change_history` continues to read Meta's ad account activity log, including edits in Ads Manager and other tools. Use `ads_list_history` for activity through Ads Uploader. `ads_list_edits` is deprecated: use `ads_list_history` with `type: "changes"`. `ads_get_edit` still returns an individual edit's full results.

## Resource reads

| Tool | Use it for | Additional inputs |
|---|---|---|
| `ads_get_account` | Account status, timezone, spend and funding display | `fresh` |
| `ads_campaign_health` | “How is campaign X doing?” | `campaignId`, `datePreset` or `timeRange` |
| `ads_list_audiences` | Available audience definitions and size bounds | `type`: custom, lookalike, saved or all; `search` |
| `ads_list_conversions` | Pixels, custom conversion definitions and standard events | Cursor inputs |
| `ads_estimate_delivery` | “How big is this audience?” | AdSpec `targeting`, `optimizationGoal`, optional `promotedObject` |
| `ads_list_change_history` | “What changed?” in Meta | `since`, `until`, `objectId` |
| `ads_list_library_media` | Meta ad account media, separate from Uploader uploads | `type`: image or video |
| `ads_list_posts` | Promotable Page posts or connected Instagram media | Exactly one of `pageId` or `igUserId` |

These tools are read-only and require `accountId`. All accept `fresh`. Lists return one bounded page with `limit` (default 25, maximum 100), opaque `after`, and `paging { after, hasMore }`. Health has a maximum of 25 campaigns. Preserve filters when continuing a cursor. Custom and saved audiences share one cursor; pixels and custom conversions share one cursor. The limit bounds the combined definition count.

Use `ads_query_insights` for custom reporting. Health keeps native result indicators and attribution windows separate, reports null metrics when delivery rows are absent, and warns when the 100-ad-set health sample is partial. Never sum unlike results or describe total actions as conversions. `ads_list_change_history` defaults to the last 7 days and has a 90-day maximum; it reads Meta activity, including Ads Manager and other tools. History IDs are strings, including nested `extra_data` IDs. Integer literals of 16 or more digits in Meta history JSON are preserved exactly as strings before parsing; secret redaction still applies. Rows add `objectType` (`AD`, `AD_SET`, `CAMPAIGN`, `AUDIENCE`, `ACCOUNT`, or another upper-cased Meta type), `adId`, `adSetId`, `campaignId`, and `audienceId` (null when unknown). Audience and account events are typed by event, because Meta reports some of them with a legacy ad object type. `metaObjectType`, `object_type`, and `extra_data` retain Meta’s legacy names: `extra_data.campaign_id` on an ad identifies its ad set. `extra_data` is always an object or null; structured data uses a 4 KB trimming budget, with `truncated: true` when oversized. Fixed keys and scalar values are retained if they alone exceed that budget; strings and array tails are shortened where possible. Non-JSON text uses `{ text, truncated }`, with text capped at 1 KB.

`ads_list_campaigns`, `ads_list_adsets` and `ads_list_ads` preserve the named list and `pagination` object, adding enriched fields, `paging` and cache metadata. Use `after` for continuation. Legacy `offset` is limited to a window within one bounded request, with `offset + limit <= 100`; it never scans earlier pages. `pagination.total` is null when the total is unknown. Existing campaign and parent-scoped lists filter only exact `status: "active"` (the default); other values, including `inactive`, `archived` and uppercase `ACTIVE`, mean no status filter. Account-wide ad-set and ad lists additionally interpret `inactive` as non-active states excluding deleted, and `archived` as archived only. Meta's unfiltered list may omit archived items. Use `fields` to select output columns; `id` is always returned, and money fields keep their `Minor` twin and `currency`. Large pages are trimmed to the 40 KB compact JSON budget, with a warning, `pagination.hasMore: true`, `pagination.total: null`, and `pagination.nextOffset`. To consume trimmed rows, keep the same input `after` (omit it again for the first page), set `offset` to `pagination.nextOffset`, and use `limit <= 100 - offset`. Repeat until no trim warning remains, then follow `paging.after` with offset reset to 0.

Money uses the account's major currency units with raw minor strings alongside. Estimates use the existing ad-creation targeting builder and require a location; `customAudiences` and `excludedCustomAudiences` accept numeric ID strings for estimates. On 429, report the retry wait and stop. Recommendations and opportunity-score reads are not exposed.

## Reporting beta and capacity

New account-wide reads, insights (including the field catalog), edits, copies and history require beta access. Ask Ads Uploader support to join the beta. Existing CLI/MCP access is still required. Hosted MCP hides the new tools for users without access; local stdio does so when `/v1/me` returns `betaFeatures: false`. An unavailable or older identity response preserves local discovery, with access enforced by the server. Existing tools and parent-scoped campaign, ad set and ad reads remain available. Denials return HTTP 403 with code `BETA_FEATURE` and the message "This is in beta. Ask Ads Uploader support to join the beta." The CLI prints that message plainly; `--json` returns an error with the code and exits nonzero.

Queries allow 10 requests per minute, report-job reads 60, and other new reads 20. Meta's reported API usage determines when fresh calls pause. At 75% usage, Ads Uploader waits for Meta's recovery estimate; without an estimate, the wait is 60 seconds, or 300 seconds at 90% usage. Fresh calls also share a concurrency cap of 5 across the app and 1 per account for each read family.

Ads Uploader also has a short-window read safety limit: a burst of 120 calls per user, replenished at 30 calls per minute, and 240 calls per ad account across users, replenished at 60 calls per minute. This is an Ads Uploader limit, not a Meta limit. Async status checks skip concurrency slots but use this safety limit. Existing campaign, ad set and ad lists keep their request burst limit and now respect Meta usage signals. Cached results make no Meta calls.

Edits, previews, applies and copies skip every Ads Uploader read limit, pause and budget, and go straight to Meta; only one apply runs at a time per ad account. They stop only when Meta itself throttles a request (Meta error 4, 17, 32, 613 or 80000-80014, or HTTP 429). That stop says it was Meta, with Meta's code and Meta's own wait, and carries `source: "meta"`, `metaCode` and `metaSubcode`; untouched changes stay pending.

Reporting and resource reads share 20 requests per second per user, plus 600 per minute per operation. `RATE_LIMITED` returns `Retry-After: 1`. Fresh calls queue for up to 20 seconds for app and account capacity; a queue timeout returns `Retry-After: 5`. An eligible first-page Insights query in auto mode returns a queued job instead. Follow its job ID. At most 10 duplicate requests wait for the same result; `TOO_MANY_DUPLICATE_REQUESTS` returns `Retry-After: 2`.

Fresh Meta rows, including metadata, resource pages, multi-account pages and job results, share a monthly UTC quota. Cached rows do not count again. Defaults are 1,000,000 rows per tier, with unlimited admins. `meta.quota` contains `rowsUsed`, `rowsLimit` and `resetsAt`; unlimited limits and unavailable usage are null. `ROW_QUOTA_EXCEEDED` means wait for the reset on the first day of the next month. Respect that reset. The client's existing 60-second automatic retry ceiling is unchanged; reads still make one request attempt.

Respect `Retry-After` for `META_READS_BUSY`, `META_READS_BACKING_OFF` and `READS_GUARD_LIMIT`. `READS_GUARD_LIMIT` replaces `META_READS_HOURLY_LIMIT` and `META_READS_ACCOUNT_LIMIT`, which remain legacy aliases for the Ads Uploader safety limit. Never retry in a loop. Cached rows remain available during backoff; `ENTITY_METADATA_DEFERRED` and `COMPARE_DEFERRED` mean optional fresh reads were deferred.

`META_REQUEST_TOO_LARGE`: do not retry the same request; ask the user to contact support.

Campaign health preserves native result arrays and adds `result` with its source, metric, value and cost per result. When native results are absent, it uses delivering ad sets' optimization goals; mixed goals remain separate and incomplete coverage is unknown. `learningStageCounts` is an empty object when no learning information was returned, and null when the ad sets could not be read. Account `spend_cap: null` with `hasSpendCap: false` means no cap; CLI tables show None. Major money amounts are numbers; raw minor amounts remain strings.

## Edits And Edit Review

Enable edits with **Agent Edit** on the **Account Permissions** page: an account-wide default or a per-ad-account setting, which wins. **Approval** is review mode; **Allowed** applies changes directly. `ads_edit_preview` returns `mode: "review"`, `reviewUrl`, `reviewId` and proposal IDs on `items`. Share `reviewUrl` with the user, do not apply, and pass `reviewId` on later previews in the same conversation. The user chooses and approves items in their Agent Inbox. Reviews expire seven days after their last update.

Nine account-scoped tools cover edits: `ads_edit_cancel`, `ads_discard_launch`, `ads_edit_preview`, `ads_edit_apply`, `ads_edit_undo`, `ads_list_edits`, `ads_get_edit`, `ads_list_edit_reviews` and `ads_get_edit_review`. The last two are read-only and show proposal statuses and edit IDs. All require `accountId`. Hosted MCP hides these tools without edit access. `ads_get_account` shows effective settings and their source; MCP cannot change them.

Pass `idempotencyKey` (8–128 letters, digits, underscores or hyphens) to `ads_edit_preview` and reuse it after a timeout. Identical requests replay for 24 hours; `PREVIEW_IN_PROGRESS` means retry in a few seconds. A different body needs a new key. Pending suggestions deduplicate across reviews for the same user and account: reused `items` carry `reused: true`, and `summary.new`/`summary.reused` give counts; there is no top-level `reused` array. Items include `reviewId` only when it differs from the response review; `detail: "full"` adds `reusedProposalIds` when non-empty. A slow preview is visible as a `building` review, becoming `open` when complete.

Passing `reviewId` moves identical pending suggestions into that review with new proposal IDs and cancels their old copies; moved rows count in both `summary.reused` and `summary.moved`. Suggestions already being applied or in a building review stay there; without `reviewId`, identical pending suggestions are reused where they are.

`ads_list_edit_reviews` returns compact metadata and status counts without proposals by default; use `include: "proposals"` for full proposals. `ads_get_edit_review` returns compact rows, 50 per page by default. Use `limit` (maximum 200) and `paging.after` for more; use `detail: "full"` for full proposal fields. A suggestion shows `applying` while an approved change is being sent to Meta; read again shortly for `applied` or `failed`.

Review mode queues proposals after batched reads and local checks, with zero Meta POSTs. Meta accepts or rejects each change when the user applies it; results show each item as applied, failed, skipped or pending. Pending means not sent yet, with a reason; apply the same review again to send these changes. Failed proposals retain a safe Meta reason. One ad's failure does not stop other ads; account errors stop the run. Direct previews also skip validate_only for status, archive, delete and rename; other direct edits still validate.

Preview responses are compact by default: `summary` contains proposal/change counts, `byOp`, object and irreversible counts, and `needsConfirmation`; `items` contains before/after rows and review proposal IDs. Warnings with the same code and message are grouped once with `ids`; differing messages stay separate. Targeting and creative items use `detail: "summary"` and `detailPaths` counts; targeting before/after values summarize the paths. Request `detail: "full"` for full `changes`, per-path `details`, individual warnings and any validation actually performed. Skipped validation has no passed rows. `currency` and `moneyUnit` appear only for money changes. Review responses keep the effective `settings.approvalRule` and omit the top-level rule. Both summary and full edit previews include `settings: { mode, approvalRule, approvalRuleAppliesTo: "edits", launch, source }`. `launch` is the separate Agent Launch mode (`allowed`, `approval` or `off`) applied to the caller; an edit review does not gate a new-ad launch. Older stored plans may omit `launch`.

Deleted, missing and archived objects that cannot take the requested operation are returned in `skipped` with their IDs, operations, reason codes and plain reasons, plus `summary.skipped`; eligible items continue, and archived objects can still be deleted. If every item is skipped, preview returns `422 NOTHING_TO_PREVIEW` with `details.skipped` and creates no review.

Use `ads_edit_cancel` to cancel pending suggestions in one review. Supply `accountId` and required `reviewId`, plus optional `proposalIds`; omit IDs to cancel every pending suggestion in that review. Nothing changes in Meta. The result contains `reviewId`, `cancelled`, per-suggestion `results` (`cancelled` or `refused`, with a reason), and `pendingCount`. Applied suggestions are refused with an `editId` and an `ads_edit_undo` pointer. In-progress and other non-pending suggestions are refused. Cancelled suggestions leave your Agent Inbox.

`ads_get_edit_review` marks pending copies in other open reviews with `duplicateOf` and reports `summary.duplicates`. To clean up older copies, call `ads_edit_cancel` with the older `reviewId` and `duplicatesOnly: true` (CLI: `--duplicates-only`), without `proposalIds`; only duplicate pending suggestions are cancelled and nothing changes in Meta.

Each call supports up to 100 objects (50 IDs per bulk operation). Append two 100-object previews with the same `reviewId` for up to 200 proposals. `reviewUrl` opens `/dashboard/inbox?account_id=...&review=...` at that review.

In **Apply Directly** mode, always show the preview's changes and warnings before applying its 15-minute plan. Set `confirmSpend`, `confirmLiveChange`, `confirmIrreversible` and `allowLargeChange` only after explicit approval of the corresponding change. `EDITS_REVIEW_REQUIRED` means approval must happen on the linked review page, including after an account switches modes. Never retry a partial apply blindly; inspect `ads_get_edit` and current Meta values first.

Archive and delete cannot be undone. `ads_edit_undo` previews reversible changes and creates review proposals in review mode. Money inputs are major units; edit preview/detail money values are minor units; History rows use readable currency amounts. See https://adsuploader.com/docs/automation/edits.

`ads_edit_preview` also accepts ad set `set_targeting` and `set_adset_settings`, plus `set_bid` with `LOWEST_COST_WITH_MIN_ROAS` and `roas_floor` (2.5 means 250%). Use `ads_search_targeting` for location and detailed-targeting IDs. Targeting uses `add`, `remove` and `set` patches on the current audience; `detail: "full"` includes per-path `details` and the full resulting audience, including all detailed-targeting types. Supply a zero-based `group` when several `flexible_spec` groups exist. Undo restores the full recorded audience; internal restore operations are not public inputs.

Ad set settings cover optimization, billing, promoted object, Standard attribution, incremental attribution, DSA beneficiary and payor. Turn incremental attribution off before changing Standard attribution. `destination_type` is read-only. Minimum ROAS requires `VALUE` optimization: campaign-budget accounts change the strategy on the campaign and floors on ad sets. Targeting and settings do not increase spend; `all_live_changes` still requires live-change approval. New operations follow the same Edit Review queue; Meta decides at apply.

### Changing An Existing Ad Creative

Use `ads_edit_preview` with `{ "operations": [{ "op": "set_creative", "level": "ad", "id": "<AD_ID>", "primary_text": "Updated primary text" }] }`. Single-text fields are `primary_text`, `headline`, `description`; multi-text creatives use `texts.bodies`, `texts.headlines`, `texts.descriptions` arrays instead. Link fields are `call_to_action`, HTTPS `link`, `display_link`, and `url_tags` without `?`. Duplicate text variants are refused.

You can edit placement-customised creatives (for example a 4:5 feed + 9:16 story/reels pair) and dynamic asset feeds with several images or videos. Edits preserve every media asset and placement rule. A text field with one row accepts its single-text field; several rows require `texts.bodies`, `texts.headlines` or `texts.descriptions`. Labelled rows require the same number of entries, matched in order; unlabelled dynamic text allows 1 to 5. Non-carousel placement feeds allow only one description. Link and display-link edits update every link row; CTA edits require a single CTA.

To change an image or video, create a new ad in the Uploader. Text-only edits keep the existing image or video.

`existing_post: { post_id: "<pageId>_<postId>" }` switches to an existing Page post; only URL tags can accompany it.

Show `ENGAGEMENT_RESET` prominently: old-post likes, comments and shares stay with the old post. Switching to an existing post avoids that warning. Also explain Meta review and possible learning reset. With `detail: "full"`, the creative row has readable `details`, the current ID in `before`, and `after: null` until apply. The prepared payload is stored for direct apply and rebuilt from current source values during review approval; up to 10 creative edits fit in a plan. These are not spend increases; live-change approval still applies. In review mode share `reviewUrl` and reuse `reviewId`.

Plain and placement-customised carousels support primary text, whole-carousel link, CTA, display link and URL tags. Use `cards: [{ card: 2, headline: "Updated headline", description: "Updated description" }]` for per-card text; card numbers start at 1. Plain carousels also accept per-card `link`; placement-customised carousels refuse it. Scalar headline and description edits are refused on carousels. Existing-post sources accept only URL tags or another post. `CREATIVE_EDIT_UNSUPPORTED` gives a precise reason for flexible, catalog, partnership, language-customised or segment-customised creatives, and unknown asset-feed optimization types. Relaunch with `ads_create` (CLI: `ads create`) using the same media, or use the Uploader's **copy settings from this ad** flow. If attachment fails after creation, inspect `error.details.orphanCreativeId` and the ad; never retry blindly. Undo reattaches the recorded old creative without creating another. Public requests cannot use internal restore operations.

## Saved-build safety and issue reports

Save and update responses include `warnings` and `display` with `totalAdSets`, `totalAds`, and the ads in each ad set. The display uses the web uploader's grouping logic. Compare it with preview totals before launching. An imported-post configuration that the web editor cannot represent returns `display.complete: false` with a reason and null totals; resolve that limitation before comparing totals. Report mismatches, contradictory results, silently ignored fields, or broken tool descriptions using `ads_report_issue` or `ads report:issue`.

Group media and per-ad text keys resolve by upload ID, Meta media ID/hash, exact file name, then variant base name. Ambiguous references fail. Unmatched custom groups fail at preview and create. To reuse an existing destination, supply its campaign/ad-set ID explicitly.

Per-ad copy names ignored shared headlines, primary text and descriptions in `warnings`. Shared link, CTA, display URL and URL tags still apply unless a row overrides them. Group `dailyBudget`, `lifetimeBudget` and `bidAmount` use whole account currency units; `minimumRoas` is a ratio. Daily and lifetime budgets are exclusive; lifetime budgets require `options.schedule.endTime`. Ad-set budgets cannot be used under a campaign budget. These group budget controls are currently server/spec-only.

Existing library videos keep their Meta video ID. At preview/create, the server recovers a missing thumbnail from Meta and uploads only that image for its hash. If Meta has no usable thumbnail, the request fails with the video ID.

```bash
ads report:issue --tool ads_preview --expected "Two ad sets" --actual "One ad set" --build-id build_123 --severity error --json
```

`POST /api/v1/issues` accepts `tool`, `expected`, `actual`, and optional `buildId`, `requestId`, `jobId`, `adIds`, `accountId`, and `severity` (`info`, `warning`, `error`, `critical`). Text and IDs are capped and redacted. Reports are logged for the Ads Uploader team to review. The response is `{ "reported": true, "reference": "report_...", "status": "queued" }`. The limit is 10 reports per user per hour. Reporting is available to all CLI/MCP users.

## Request missing capabilities

When a user goal cannot be done with the available tools or parameters, call `ads_request_feature` yourself without asking the user, then continue with the best workaround. Use `ads_report_issue` when an existing tool misbehaves.

Provide `goal` (up to 500 characters) and `missingCapability` (up to 300). Optional fields are `toolsTried` (up to 20 tool names), `accountId`, `level` (`account`, `campaign`, `adset`, `ad`, `creative`, `build`, `upload`, `insights`, or `other`), and `workaround`. Describe the capability; omit names, emails, ad copy, tokens, and other personal data.

Similar requests are grouped. The response includes `requested: true`, a `reference`, `status: 'recorded'`, `duplicate`, and `timesRequested`. A repeated request from the same user within 24 hours does not increase the count. Available to every MCP/CLI user, limited to 20 requests per user per hour.

Post-ID duplication starts directly by default, with or without a beta grant. With the beta, the account’s Agent Launch setting decides, as for every agent copy: Approval sends it for approval and Off refuses it. Agent Edit does not apply.

## Product Showcase

Product Showcase shows products from a catalog under a single image or video ad. Full catalog (dynamic/DPA) ads are not supported yet.

IDs must be strings of digits. Recommendations require Sales campaigns (`OUTCOME_SALES`); manual catalog/set choices also work on other objectives. Carousel, flexible, multi-media, existing-post and Partnership ads cannot enable Product Showcase.

Discover catalogs and sets, then request a recommendation:

- `ads_list_catalogs({ accountId })`
- `ads_list_product_sets({ accountId, catalogId, productSetId? })`
- `ads_get_product_showcase_recommendation({ accountId, adUrl, pageId?, pixelId? })`

Create and preview accept `productShowcase: { enabled?, catalogId?, productSetId?, useRecommendation? }`; `enabled` defaults to true. Omit the field to inherit the source creative. `RECOMMENDED` selects Meta’s default (`choiceSource: "DEFAULT"`); `MUST_PICK` requires a returned candidate ID as `productSetId` (`"MANUAL"`). `MANUAL` requires a catalog and product set. A validated override of Meta’s default uses `"MANUAL"`.

CLI create and preview flags: `--product-showcase <on|off|recommended>`, `--catalog <id>`, `--product-set <id>`. IDs alone mean on; recommended maps to `useRecommendation: true`. Off maps to `{ enabled: false }` and cannot include IDs; it disables showing products while keeping the source ad’s own set attached.

Preview returns `productShowcase` with enabled, catalog/set IDs, optional set name, choiceSource and source (`recommendation`, `manual`, `source_ad`), or null when omitted. Recommendations are read-only and do not enable the feature. Test Mode makes no catalog calls, requires explicit catalog/set IDs when enabled and notes that membership/recommendations were not verified. Saved UI choices map to Product Showcase; saved defaults are checked again before launch.
