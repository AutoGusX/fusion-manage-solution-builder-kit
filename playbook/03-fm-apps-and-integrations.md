# Apps, extensions and integrations on Fusion Manage

Three delivery shapes, and when to use each.

| Shape | Use when | Auth | Watch out for |
|---|---|---|---|
| **Chrome MV3 extension** | You are augmenting FM's own UI or doing admin or bulk tasks for people already signed in | Session cookie exchanged for a bearer token | FM's UI is a moving target; no CORS limits, so uploads work "by accident" |
| **Browser-only SPA** | You want a standalone app for suppliers, reviewers or customers with no server to run | APS 3-legged, PKCE public client | S3 upload CORS; needs the tenant admin to allow your client ID |
| **SPA plus small relay** | You need uploads or read-only reports a browser cannot do alone | Same as SPA, plus a same-origin function | The relay is a security boundary; keep its controls fixed |
| **MCP server or script** | An agent or batch job drives FM | Client credentials or a user token | Same API traps as everything else |

## Browser-only SPA pattern

Vite, React and TypeScript, hosted statically (for example Firebase Hosting). No backend.

- **Auth:** APS 3-legged with PKCE as a public client. A confidential-type app fails with `401 invalid_credentials`, so create the app as a public single-page app.
- **Enablement:** the tenant admin adds your client ID to *Allowed client IDs* in the tenant's general settings. That one switch is the only gate. FM's API answers CORS for any origin.
- **Token handling:** access token in memory only, refresh token in `sessionStorage`, `localStorage` only for non-sensitive things like recent tenant names.
- **HTML from FM:** render it only through a sanitizer that outputs React elements, never `dangerouslySetInnerHTML`.
- **Profiles:** drive per-use-case behavior (which workspaces and fields matter) from a JSON profile with auto-detection and an override, rather than hardcoding.

## The upload relay

Attachment upload goes ticket, then bytes to a signed S3 URL, then a check-in. The S3 bucket answers the browser's CORS preflight with `403` for every origin except the tenant's own. Chrome extensions bypass CORS, so "works in my extension" does not transfer to a web app.

Solution: a same-origin relay (a serverless function) that forwards the bytes. Treat it as a security boundary:

- **Fixed controls, never configurable:** destination allowlist (no SSRF), caller authentication, no forwarding of arbitrary headers, no logging of payloads, identity tied to the caller.
- **Per-app policy only:** allowed file types, size ceiling, rate limits.
- Know the platform ceiling. Serverless request-size limits (for example 30 MiB on Firebase functions) cap file size; larger files need a streaming relay.
- Use an upload queue: ticket, transfer, commit, with retries, a fresh ticket if the signed link expires, and a few files in flight at once.

## Rendering attachments

Do not copy files into a bucket and run Model Derivative. FM has already translated them. `GET` the attachment with the viewable media type and you get FM's own derivative URN, which the APS Viewer loads with the user's own token. See [`gotchas/fm-api.md`](../gotchas/fm-api.md), entry on viewables and markups.

## Chrome extension pattern

- Plain JavaScript with no build step is fine for admin tools; React, TypeScript and Vite is fine for richer UIs. Pick one per project and stay with it.
- Get a bearer token from the page origin with `GET /api/v3/token` and `credentials: 'include'`. Cache it and refresh about 60 seconds before expiry.
- Anchor injected UI **outside** the React subtree. FM's modern admin navigation is React; anything you append inside it is wiped on the next render. Append as a sibling to the Angular wrapper, and add a self-healing `MutationObserver`.
- Pin your selectors to wrapper elements, and expect to revisit them when FM updates its UI.

## Integrations and agents

- Discover the API base per tenant instead of hardcoding it (see the configuration chapter).
- Write one shared HTTP client that defaults `Accept: application/json`, retries on 429 and 5xx, and refreshes tokens. Every call site then inherits the fixes.
- For agent-driven work (MCP), expose a small set of safe, well-described tools, and put a guardrail allowlist in front of anything that writes.

## Packaging for others

- Pure logic ported between projects gets its test assertions ported with it. Port the harness, not only the code.
- Commit the lockfile. Use plain `vite@^7`; see [`gotchas/tooling.md`](../gotchas/tooling.md).
- Before publishing: scan for tenant names, tokens, captures and customer data.
