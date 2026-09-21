# Privacy

This describes what the Ads Uploader MCP connector accesses when you use it from an AI agent. For the full company privacy policy, see https://adsuploader.com/privacy-policy.

## What it accesses

When you connect and authorize the Ads Uploader MCP server, the agent acts on your behalf against your Ads Uploader account, which in turn connects to the Meta (Facebook/Instagram) Marketing API you have authorized. Through the tools, an agent can:

- Read your ad accounts, campaigns, ad sets, ads, Pages, and presets.
- Upload media you provide (from public URLs, a Google Drive folder, or local files via the stdio package).
- Create and duplicate ads — created **paused** by default.

## Authentication

- **Hosted server** (`https://adsuploader.com/api/mcp`): OAuth 2.1 with PKCE in your browser. The connector receives a scoped Ads Uploader token; it never asks you to paste Meta access tokens.
- **Local stdio package**: uses credentials saved locally by `ads login`. These stay on your machine.

## Data handling

- The connector operates on the ad data in the accounts you authorize. It does not sell your data.
- Media you import is staged and processed to create ads in your own Meta ad accounts.
- You can revoke access at any time by disconnecting the connector in your MCP host and/or revoking the app in your Ads Uploader and Meta account settings.

## Contact

Questions about data or privacy: support@adsuploader.com
