# Fusion add-ins that talk to Fusion Manage

Lessons from building add-ins that run inside Fusion (desktop) and call FM. Fusion's own API reference covers the basics; this chapter covers what you learn the second time.

## Shape of a good add-in

- **Python is authoritative, the palette is a view.** Business logic, API calls and state live in Python. The HTML palette renders and sends events. Create and edit flows that split logic between the two drift apart.
- **Keep the model pure.** Put the logic that decides what to do (for example, resolving an assembly into records to create) in plain Python modules with no `adsk` imports. That lets you test it offline in seconds.
- **Ship as a separate add-in when you rewrite.** Suffix every UI ID (for example `_V2`) so the old and new add-ins can be installed side by side while you compare.

## Dev loop

1. Source lives in a git repo, not in Fusion's AddIns folder.
2. A sync step copies the repo into `…/Autodesk Fusion 360/API/AddIns/<name>` (on Windows, under `%APPDATA%\Autodesk\Autodesk Fusion 360\API\AddIns`).
3. Run the offline self-test first. A single `python docs/dev/selftest.py` style script with a few hundred checks catches most regressions before you open Fusion.
4. Then run in Fusion from the Scripts and Add-Ins dialog.

Gitignore `tokens.json` (live access and refresh tokens), preference files, `__pycache__/` and `*.zip`. A token file in a public repo is a leaked account.

## Verification doc

Keep a `VERIFICATION.md` organized in numbered phases. Each phase lists exact steps to run inside Fusion and what you expect to see, including edge cases such as a shared library part, a nested external reference, a document with unsaved edits, and a very large STEP file.

Offline tests and static checks are not the same as a GUI run. Track "built and unit-tested, never run in Fusion" as an explicit list at the top of the backlog so nobody builds on top of an unverified path.

## Fusion API traps

- **A new Python wrapper per call.** Fusion returns a new wrapper object each time you ask for the same thing. Compare with `==`, never `is`.
- **Resolve assembly children per occurrence.** Use `occurrence.documentReference.dataFile`. `Component.parentDesign` returns the host file and collapses every child onto the root record.
- **Find drawings through the data file.** `DataFile.parentReferences` leads to the drawings that reference a part.
- **Blocking calls freeze the palette.** A long synchronous call stops the palette from repainting. Break long operations into staged calls (for example open, render, close) so you can update progress between them. Whether the palette repaints during a blocking call is worth testing in your Fusion version.
- **Test the whole flow in Fusion.** UI that looks right in headless Chrome can still misbehave inside Fusion's embedded browser.

## Auth

Use an APS 3-legged flow with a public (PKCE) client. Store tokens on disk only if you must, in a gitignored file. Expect expiry about an hour out and refresh transparently.

## Talking to FM from the add-in

Everything in [`gotchas/fm-api.md`](../gotchas/fm-api.md) applies. The two that bite add-ins most:

- `PUT /items/{id}` replaces the whole record. Use `PATCH` for partial updates.
- Attachment upload is a three-step flow (ticket, transfer to S3, commit). From a desktop add-in you have no CORS restriction, so you can PUT to the signed URL directly.

## Show progress

Export and sync flows take time. Show a job with per-stage progress rather than a spinner, and keep the palette responsive by staging the work.
