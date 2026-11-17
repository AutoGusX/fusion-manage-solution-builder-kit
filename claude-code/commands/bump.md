---
description: Bump the extension version in manifest.json + package.json (kept in sync)
argument-hint: "[patch | minor | major]  (optional—will infer + confirm if omitted)"
---

Bump the version for the **current project** (an MV3 browser extension), keeping
`manifest.json` and `package.json` in lockstep.

Requested level (may be empty): **$ARGUMENTS**

Rules (versioning policy):
- **patch** (`+0.0.1`)—after any edit session: small fixes, tweaks, CSS, minor behaviour changes.
- **minor** (`+0.1.0`, reset patch to 0)—after a significant block of work: new features, major bug fixes, architectural changes.
- **major** (`+1.0.0`, reset minor+patch to 0)—ONLY when the user explicitly asks to roll a major version.

Steps:

1. **Locate both files.** Manifest is usually `public/manifest.json` (fall back to `manifest.json` or `src/manifest.json`). Package is `package.json` in the project root. If either is missing, stop and tell the user.

2. **Read current versions.** Report both. If they don't match, flag it and ask the user which is authoritative before proceeding.

3. **Decide the level.** If `$ARGUMENTS` names a level, use it. Otherwise infer from what changed this session (small fixes → patch; new feature/architecture → minor) and state your reasoning in one line, then confirm with the user before writing. **Never** bump major unless the user explicitly asked.

4. **Compute the new semver** per the rules above and **edit the `"version"` field in BOTH files** to the identical new value.

5. **Report** `old → new` and which files were touched.

IMPORTANT: Do NOT create a hook to automate this—Edit/Write hooks on version files cause infinite loops. This command is the deliberate manual mechanism; run it at the end of a work session or when scope is clear.
