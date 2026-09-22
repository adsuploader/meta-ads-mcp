# Connect Ads Uploader to Grok

Use Grok to prepare Facebook and Instagram ads, write creative variations, and edit saved builds you can review in Ads Uploader.

You need a paid Ads Uploader account with MCP access enabled and a connected Meta account with access to the ad accounts you want to use. MCP access is not included in trials.

## Connect

1. Open [Grok Connectors](https://grok.com/connectors).
2. Choose **New Connector**, then **Custom**.
3. Enter `https://adsuploader.com/api/mcp` as the server URL.
4. Follow the authentication prompt, sign in to Ads Uploader, and review the requested access before approving it.
5. Ask Grok: **“Use Ads Uploader to show which account I connected and list my Meta ad accounts.”**

For Grok Business and Enterprise, your team admin may need to provision the connector first.

## Try your first build

> Use the settings from this existing ad to prepare three creative variations. Give each a different headline and primary text. Save the build and give me its Ads Uploader link so I can review it.

For media imports, provide public HTTPS download URLs or a public Google Drive file or folder link. Upload local files through the Ads Uploader web app or CLI, then ask Grok to work with the resulting saved build.

Review the destination account, campaign, ad set, media, copy, and requested status before asking for ad creation. A saved build can be opened and edited in the web uploader.

## Troubleshooting

- **Access denied:** confirm that your Ads Uploader account has paid MCP access and CLI access has not been disabled.
- **No ad accounts:** check the connected Meta account and its account permissions in Ads Uploader.
- **Media cannot be imported:** check that the download URL or Drive link is publicly accessible, or upload through the web app.
- **Connector unavailable in a team:** ask the Grok workspace admin to check connector provisioning.

These steps follow [xAI's custom MCP connector documentation](https://docs.x.ai/grok/connectors), checked on September 22, 2026. The Ads Uploader connection has not yet been tested end to end in Grok.

For help, visit [Ads Uploader support](https://adsuploader.com/docs/support/contact).
