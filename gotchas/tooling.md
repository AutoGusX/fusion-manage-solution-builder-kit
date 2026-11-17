# Tooling gotchas

## `rolldown-vite` alias breaks fresh installs

- **Symptom:** A fresh `npm install` fails with `ERESOLVE unable to resolve dependency tree … Found: vite@undefined … peer overridden vite@"npm:rolldown-vite@…" from @vitejs/plugin-react`.
- **Cause:** `"vite": "npm:rolldown-vite@X"` in devDependencies plus a matching `overrides` block cannot be resolved against the plugin's `vite` peer range. Existing projects survive only because a committed `package-lock.json` pins resolution. Delete the lockfile and they break too.
- **Fix, in order of preference:**
  1. Use plain `vite@^7`. The build-speed win is not worth an install-time trap in a repo other people clone.
  2. Keep the alias, but always commit `package-lock.json`.
  3. `npm install --legacy-peer-deps` works, but every consumer has to know.

## A piped install reports the wrong exit code

`npm install … 2>&1 | tail -25` returns `tail`'s exit code, not npm's. A failed install looked like success. Redirect output to a file and check `$?` directly, or use `PIPESTATUS`.

## Corporate registry throttling

If `npm install` times out on a managed machine, try an alternate mirror registry for the install, then restore the lockfile's resolved URLs to the canonical registry before committing. Do not commit mirror URLs.

## Windows agent shells

- `node` and `npm` may be installed but not on the agent shell's `PATH`. Export the install directory first, or builds and tests silently cannot run.
- `python` and `python3` on `PATH` may be Microsoft Store stubs. Use the real interpreter's full path.

## Pre-publish scan

Before making any repo public, search the working tree **and git history** for:

- Access or refresh tokens, API keys, client secrets, service account JSON
- Tenant names and `autodeskplm360.net` hostnames
- APS client IDs you do not intend to publish
- HAR files, viewer URNs, signed S3 URLs
- Customer names and customer data
- Personal paths and email addresses

If anything sensitive ever touched history, rotate it and publish from a fresh single-commit history rather than trying to scrub.
