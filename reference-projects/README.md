# Reference projects

Architecture summaries of patterns that proved out. They describe the approach so you can reproduce it. They do not include source code or tenant details.

## Public

- **[Fusion Manage live viewer](https://github.com/AutoGusX/fusion-manage-live-viewer)**: a Claude Code plugin that runs a local APS viewer with BOM, table and record panels, and drives FM operations through a CLI. Bring your own APS client ID and tenant.
- **[Fusion Manage PLM docs site](https://github.com/AutoGusX/fusion-manage-plm-docs)**: an Astro Starlight site documenting the FM API surface used in this kit. Useful alongside [`gotchas/fm-api.md`](../gotchas/fm-api.md).

## Patterns

### Browser-only supplier portal
React and TypeScript SPA over FM. APS 3-legged with PKCE, no backend, hosted statically. A JSON profile with scored auto-detection drives which workspaces and fields matter for the use case. Tenant admin enables it with one setting. Uploads go through a same-origin relay. Shows: the SPA pattern, bulk actions with one transport seam, safe HTML rendering, in-app file viewing with markups.

### Upload and report relay kit
A reusable hardened relay for browser-only FM apps. Fixed security controls (destination allowlist, caller check, no header forwarding, no payload logging, identity binding) with only a per-app policy configurable: a strict image profile or a bulk-files profile. Includes an upload queue (ticket, transfer, commit, retries, re-ticket on expiry). Shows: how to share a security boundary across apps without letting it drift.

### Admin utilities extension
Plain JavaScript MV3 extension for admin tasks: workspace migration over the Configuration API, roles backup and creation. In-memory fake API for fast tests, a planner and diff with no deletes by design, and per-tenant API base discovery. Shows: configuration as code with a safe apply.

### Fusion add-in for PLM workflows
Python add-in with an HTML palette, a pure-Python model tested offline, and a phased in-Fusion verification doc. Exports assemblies, drawings and electronics BOMs to FM as a staged job with progress. Shows: Python-authoritative design and the owed-verification discipline.

### Configuration framework
Requirements, baseline snapshot, architecture as code (YAML compiled to the admin tool's formats), plan and diff in a single interactive `plan.html`, logged apply, and an agent-executable handoff. Shows: taking a tenant from discovery to configured with a human approving each stage.

### MCP servers for PLM
Python HTTP MCP servers exposing a small, guarded set of FM tools to agents. Shows: a guardrail allowlist in front of anything that writes.
