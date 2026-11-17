# Fusion Manage API gotchas

Each entry: symptom, cause, fix, and when it was last verified. Tenant placeholder is `<tenant>`; base URL is `https://<tenant>.autodeskplm360.net`.

These come from live probing. Many endpoints are not publicly documented and can change. Re-verify before relying on one.

---

## 1. Attachment upload fails with `400 Incorrect payload`

- **Symptom:** `POST /api/v3/workspaces/{ws}/items/{item}/attachments` returns `400 {"message": "Incorrect payload"}`. The message names no field and points at the body.
- **Cause:** The request carried `Accept: */*`. Browser `fetch()` cannot omit `Accept`, so it always sends that. A server-side client that sends no `Accept` at all gets `201` for the identical body, so the bug reproduces only in the browser.
- **Fix:** Default `Accept: application/json` on every request in your shared client. GETs tolerate `*/*`, which is why the symptom is "all reads work, all uploads fail". Keep two opt-outs: pre-signed S3 transfers must send no `Accept`, and endpoints needing a vendor media type (for example the attachment list's bulk type) must send it. Sending that bulk type on the upload POST also yields `Incorrect payload`.
- **Related facts:**
  - Upload step 1 returns `201`, not `200`. The commit body is `{status: {name: 'CheckIn'}}`.
  - Allowed status transitions: `UPLOADING → CHECKEDIN | UPLOADFAILED`, `CHECKEDIN → CHECKEDOUT`, `CHECKEDOUT → CHECKEDIN`. Delete is not a status; use `DELETE` on the attachment URL (`204`).
  - `resourceName` on upload is a free-text title that must be non-empty.
  - Image fields need a bearer token and `Accept: */*` (`406` otherwise). This is the one place `*/*` is required.
- Verified 2026-09.

## 2. `PUT /items/{id}` replaces the whole record

- **Symptom:** A `PUT` with only the changed field returns `400 "Title is required"`. A `PUT` that omits optional fields returns `204` and clears them.
- **Cause:** `PUT` is a full replace and accepts workspace-scoped links.
- **Fix:** Use `PATCH`. It merges, but demands item-scoped links (`400 "Could not find section {s} in workspace {ws}"` for the workspace form).
- **Derive item-scoped links without a GET:**
  - `/workspaces/{ws}/sections/{s}` → `/workspaces/{ws}/items/{id}/views/{v}/sections/{s}`
  - `/workspaces/{ws}/views/{v}/fields/{F}` → `/workspaces/{ws}/items/{id}/views/{v}/fields/{F}`
- A picklist can be required only on write (`400 "You must select an option for …"`) even when `/fields` validators do not flag it.
- Grid rows differ: `PUT …/rows/{id}` legitimately replaces the row, so build the row payload whole.
- Verified 2026-09.

## 3. Attachment uploads blocked by S3 CORS from web apps

- **Symptom:** Upload works from a Chrome extension, fails from your web app with a CORS error.
- **Cause:** Upload is `POST …/attachments` (returns a signed S3 URL), then `PUT` bytes to S3, then `PATCH {status:{name:"CheckIn"}}`. The S3 bucket answers the preflight with `403 AccessForbidden` for every origin except `https://<tenant>.autodeskplm360.net` (allowed headers: `content-type`, `x-amz-meta-filename`). Extensions with host permission for `*.s3.amazonaws.com` are not subject to CORS.
- **Fix:** A same-origin relay function. See [`playbook/03`](../playbook/03-fm-apps-and-integrations.md).
- **Test without a browser:** send the preflight with `curl -X OPTIONS -H "Origin: …"`.
- A pending attachment cannot be deleted until it is checked in (`403 "File must be checked in"`).
- Verified 2026-10.

## 4. Getting a bearer token from a content script

- **Symptom:** `localStorage['O2AuthToken']` is missing or stale.
- **Cause:** Tokens are no longer reliably stored there. The session cookie (`JSESSIONID`, HttpOnly, on the tenant domain) is not readable by script and is not in `localStorage`.
- **Fix:** From the page origin, `GET /api/v3/token` with `credentials: 'include'`. Response: `{ "accessToken": "…", "expiresIn": 3599, "tokenType": "Bearer" }`. The JWT carries `userid` and scopes (`data:read/write/create/search`, `user:read`). Cache it and refresh about 60 seconds before expiry.
- This is the current user's token. Two-legged client credentials are only needed to act as other users.

## 5. Show an attachment in the APS Viewer without Model Derivative

- **What works:** `GET /api/v3/workspaces/{ws}/items/{id}/attachments/{att}` with `Accept: application/vnd.autodesk.plm.attachment.viewable+json` returns `{fileUrn, fileName, environment, viewerName: 'LMV'|'BOX', status: 'DONE'|'REGISTERING', eventId?}`.
- **The GET itself starts translation** when none exists (DWF around 90 s, DWG around 2 min). Poll the same GET or the manifest. `?force=true` re-registers.
- **Loading:** `Autodesk.Viewing.Initializer({ env: 'AutodeskProduction', api: 'derivativeV2', getAccessToken })` with the user's own 3-legged token. No bucket, no job POST (it fails with "Failed to trigger translation for this file"), no credits.
- `fileUrn` is base64 of `urn:adsk.plm:flc.file.s3:{tenant}.autodeskplm360.net/{bucket}/{key}`.
- `viewerName: BOX` (PDF, DOCX, XLSX) uses Box View; the completed shape was not observed.
- Verified 2026-09.

## 6. Viewer markups API

- `GET …/attachments/{att}/markups` returns a bare array of layers: `{id, selfLink, urn, allowModify, allowDelete, lastModified, createdBy, markupData}`.
- `markupData` is a **JSON string** `{Data, Thumbnail, ThumbnailFullSize}`. Thumbnails are base64 JPEG.
- `Data = {title, is2d, sheetGUID, sheetName, viewerState, markups[]}`. `viewerState` is a verbatim `viewer.getState()`, restored with `viewer.restoreState()`.
- `markups[i] = {type, state, dataModelVersion: 6}`. Types: `freehand | arrow | rectangle | cloud | callout | simple-arrow-callout` (the last is a group; children carry `groupId` and `groupOrder`).
- `POST …/markups {Data, Thumbnail, ThumbnailFullSize}` returns `201` with a `Location` header. `PUT …/markups/{n}` edits and keeps the layer id. `GET …/markups/{n}` is `405`; list only. A Participant license can write markups.
- The API is plain storage. It accepts markups on any file, but FM's own viewer may not display them on images or PDFs (those route to `BOX`).
- **A new file version gets a new attachment id with zero markups.** Old layers stay on the old id. Carry them forward yourself by re-POSTing with `viewerState.seedURN` swapped to the new file's `fileUrn`.
- **2D layers** (`is2d: true`) store anchors in the sheet's world units, not the 3D 0 to 1000 screen space. Open the sheet with `doc.getRoot().findByGuid(Data.sheetGUID)`, call `restoreState`, then map with `viewer.worldToClient`.
- Layer `urn` and the decoded `seedURN` contain the tenant name. Never commit captures.
- Verified 2026-09 to 2026-11.

## 7. System log API

- `GET /api/v3/tenants/{TENANT_UPPERCASE}/system-logs?type=item&offset&limit`. Admin read only (`403` otherwise). **The tenant name must be uppercase.** Newest first, `{totalCount, items}`.
- `type=item` is record activity. `user` and `all` return `400`.
- **Only `filter[description]=<text>` is a useful filter** (case-insensitive contains, no wildcards, `204` when empty). `filter[timestamp]=YYYY-MM-DD` matches one exact day (anything else is `500`). Every other `filter[x]` returns `400 "Invalid filter parameter x"`. `action=` and `query=` are silently ignored. `sort=timestamp` is oldest-first only.
- Filter on description, page newest-first, and cut dates client-side.
- **Short pages are not the end.** A page of 488 out of 500 appeared mid-log. Advance the offset by the page size and stop on an empty page or when offset reaches `totalCount`.
- About 5 s per 500-row page; three in parallel was fine.
- Entry shape: `{action, description, timestamp, item:{link,title,deleted}, user:{title,email}}`. Markup events read "New markup <title> added. Attachment: <file> <version>". The log mangles non-ASCII characters.
- Verified 2026-11.

## 8. Picklists and field types

- `picklist` arrives as a **bare string** path, and only on `/workspaces/{ws}/fields` or a single-field GET. The bulk `/views/{v}/fields` returns a slimmer shape without it.
- Plain and linking picklists both submit as `{link, title}`. A bare label is rejected with `400 Invalid field value`.
- Field type ids that can carry a lookup: 6, 7, 10, 13, 14, 20, 22, 23, 24, 25, 26, 27, 29. Titles are ambiguous ("Single Selection" is both id 20, plain, and id 23, linking), so branch on the type id or `nameKey`, never the title.
- Verified 2026-09.

## 9. Per-tenant API base for the Configuration API

- Sandbox tenants answer under one path (`/plm360-ft`) and production under another (`/plm360`). Hardcoding one breaks on the other.
- Discover it per tenant: `GET /api/v3/configurations/{key}` for `ADSK_API_BASE_URL` and `ADSK_FLC_CONTEXT`.
- Verified 2026-11 on one sandbox and one production tenant.

## 10. Enabling a third-party web app

- FM's API answers CORS for any origin. The only enablement switch is the tenant admin adding your APS client ID to **Allowed client IDs** in general settings.
- An APS app created as a confidential type fails PKCE with `401 invalid_credentials`. Create a public single-page app.
- Verified 2026-09.

## 11. Observed, not fully understood

- An account that appears switched to basic access (every section locked, `ADD_ITEMS denied`) was restored to full access after calling `GET /api/v3/users/@me/licenses`. Reported once. Re-read `/items/{id}/permissions` afterward. Treat this as an observation, not a documented behavior.
