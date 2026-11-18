# Fusion Manage solution builder kit

A practical playbook for building on Autodesk Fusion Manage (FM, also known as Fusion PLM): Fusion add-ins, browser apps, Chrome extensions, integrations and tenant configuration, developed with Claude Code.

It captures a working method plus the API behavior that cost real debugging time to learn. It is written for solution architects and developers who want to move fast without relearning the same traps.

> **Community project, not officially supported by Autodesk. Use at your own risk.** This kit is shared as an example for the open Autodesk developer ecosystem and platform services developer advocacy. It is not official Autodesk documentation. Much of what is described here rests on behavior observed against live tenants, including endpoints that are not publicly documented (visible to any user in the browser's network tab). Those can change without notice. Every gotcha carries the date it was last verified. Re-check before you depend on one, and test in a sandbox tenant first.

## What is in here

| Folder | What it covers |
|---|---|
| [`playbook/`](playbook/) | The method: spec-first workflow, Fusion add-ins, FM apps and integrations, FM configuration as code |
| [`gotchas/`](gotchas/) | FM API, web app, extension and tooling traps, each with symptom, cause and fix |
| [`claude-code/`](claude-code/) | Slash commands, templates and a starter `CLAUDE.md` to adapt for your own Claude Code setup |
| [`reference-projects/`](reference-projects/) | One-page architecture summaries of the patterns that proved out |

For endpoint-level detail, see the companion [Fusion Manage PLM docs site](https://autogusx.github.io/fusion-manage-plm-docs/). This kit is the method and the traps; that site is the API reference.

## Start here

1. Read [`playbook/01-way-of-working.md`](playbook/01-way-of-working.md). It is five minutes and sets the rules everything else follows.
2. Skim [`gotchas/fm-api.md`](gotchas/fm-api.md) before you write your first FM client. Most of the pain lives there.
3. Pick the playbook chapter for what you are building:
   - A Fusion desktop add-in: [`02-fusion-addins.md`](playbook/02-fusion-addins.md)
   - A web app, extension or integration: [`03-fm-apps-and-integrations.md`](playbook/03-fm-apps-and-integrations.md)
   - Tenant configuration: [`04-fm-configuration.md`](playbook/04-fm-configuration.md)
4. Copy what you want from [`claude-code/`](claude-code/) into your own `~/.claude/`.

## Principles in one breath

Plan before building. Specs and a backlog are the source of truth. Verify against a live tenant before calling anything done. Read the gotchas before you probe. Never commit tokens, tenant names or customer data.

## Contributing

Found a trap that is not listed, or one that no longer reproduces? Open an issue or pull request. Please include the endpoint, the request, the response, the date, and no tenant names, tokens or customer data.

## License

[MIT](LICENSE)
