# Way of working

The method behind every project in this kit. It is deliberately light.

## 1. Plan before building

For anything non-trivial, propose an approach and get agreement before writing code. With Claude Code, use plan mode: the agent reads, proposes, and you approve. Small fixes skip this. New features, new integrations and anything that writes to a tenant do not.

## 2. Specs and backlog are the source of truth

Every project has:

- `specs/NNNN-short-name.md`: one numbered spec per feature, from the [spec template](../claude-code/templates/spec-template.md). Status moves `draft → approved → in-progress → done`.
- `BACKLOG.md`: `Now`, `Next`, `Later`, `Done`, each item linking to its spec.
- `CLAUDE.md`: architecture, commands and conventions, checked into git so it travels with the code. Keep it short, because it loads every session.

Read the relevant spec before implementing. Update the backlog as work completes. Acceptance criteria in the spec must be concrete and checkable, because they are how you (and the agent) know the work is done.

## 3. Verify before calling it done

"Tests pass" is not done. Done means you drove the change end to end:

- Offline logic: unit tests with an in-memory fake of the FM API.
- Anything that talks to FM: run it against a sandbox tenant, with real records you can throw away.
- UI: look at it. Headless Chrome screenshots catch layout problems, but for Fusion add-ins the real check is inside Fusion.

Keep an explicit **owed list** in the backlog for anything that was built and tested offline but has not yet run in the real host (Fusion, Chrome, a live tenant). Do not build further on an unverified path without checking whether it has run.

## 4. Probe, then record

Most FM surprises come from endpoints behaving differently from what you expect. When you hit one:

1. Reproduce it with the smallest request you can.
2. Record symptom, cause, fix and the date in [`gotchas/`](../gotchas/).
3. Fix the shared helper, not the one call site.

A gotcha that lives only in someone's head gets rediscovered, often in a second project.

## 5. Safety rules for tenants

- Use a sandbox tenant for development. Archive or delete the scratch records afterward.
- Dry run first: build a plan or diff, review it, then apply. Log what was applied.
- Never delete configuration automatically. Hide or retire things by hand.
- Writes need explicit approval when they touch a tenant someone else owns.
- Tokens expire (around an hour). Treat a 401 as routine, not a bug.

## 6. Keep secrets out of git

Gitignore token files, preference files with account data, and `__pycache__`. Never commit tenant names, client IDs you do not intend to publish, API captures (HAR files, viewer URNs and attachment links embed the tenant name), or customer data. See [`gotchas/tooling.md`](../gotchas/tooling.md) for a pre-publish scan.

## 7. Small commits, honest versions

Small, reviewable commits with clear messages. For extensions, bump the version by hand at the end of a work session (patch for fixes, minor for features, major only on request), keeping `manifest.json` and `package.json` in sync. Do not automate this with an edit hook: hooks on version files loop. The [`/bump` command](../claude-code/commands/bump.md) is the deliberate mechanism.

## 8. Show progress while waiting

For any UI that waits on data: show the window before the data is ready, show progress on every wait, and parallelize independent requests. A tool that looks frozen gets abandoned.
