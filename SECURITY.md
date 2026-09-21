# Security Policy

## Reporting a vulnerability

If you believe you have found a security vulnerability in the Ads Uploader MCP server or connector, please report it privately.

- **Email:** support@adsuploader.com (subject: `SECURITY`)
- Please include steps to reproduce, affected endpoints, and any relevant request/response detail.
- Do not open a public issue for security reports.

We aim to acknowledge reports promptly and will keep you updated on remediation. Please give us reasonable time to investigate and fix an issue before any public disclosure.

## Scope

- Hosted MCP server: `https://adsuploader.com/api/mcp`
- The `@adsuploader/mcp` npm package (local stdio server)

## Handling of credentials

- The hosted server authenticates via OAuth 2.1 (PKCE) in your browser; it does not ask you to paste Meta tokens.
- The local stdio package uses credentials saved by `ads login` on your own machine and never transmits them to third parties.
- Report any behavior that exposes tokens, other users' data, or another account's ads.
