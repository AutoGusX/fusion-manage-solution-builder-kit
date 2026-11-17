# Web app and extension gotchas

## FM's modern admin navigation is React MUI

- **Symptom:** Your injected button disappears after a moment, or "the page mutations changed". The selector still matches.
- **Cause:** FM's modern admin (`/plm/admin/system-configuration/`) navigation is a React MUI `Drawer` (`MuiDrawer-root`, emotion `css-*` classes) wrapped in an Angular `<navigation-panel>` directive. Anything appended inside the React subtree is wiped on React's next render.
- **Fix:** Append your UI as a **sibling** to the Angular-owned container that wraps the React root. Priority order: `.navigation-panel-wrapper`, then `.system-configuration-container`, then `.admin-page-content`. Add a self-healing `MutationObserver` as backup.

## Extensions bypass CORS; web apps do not

An extension with host permissions for `*.s3.amazonaws.com` can upload to FM's S3 bucket. The same code in a web app is blocked. See [`fm-api.md`](fm-api.md) entry 3 and the relay pattern in the playbook.

## Render FM HTML safely

Rich text from FM must go through a sanitizer that returns React elements. Do not inject it with `dangerouslySetInnerHTML`.

## JavaScript trap: `"text"?.link`

On a string, `.link` is `String.prototype.link`, a function. `value?.link` on a field that may be a plain string yields that function, not a URL. Test `typeof v === 'object'` before reading `.link`.

## Astro and Starlight links on GitHub Pages project sites

- **Symptom:** Navigation works, but links inside page content 404.
- **Cause:** On a site deployed with a `base` (for example `/<repo>/`), Astro does not base-prefix links written in markdown or frontmatter. A link written `/api/v3/items/` ships literally. Starlight's own sidebar and nav links are prefixed, which is why this is easy to miss. Hero `actions[].link` values are used verbatim too.
- **Fix:** A small dependency-free rehype plugin in `astro.config.mjs` that prefixes internal root-relative `href`s at build time, so `base` is defined once.
- **Verify:** Build it and grep the emitted HTML. Add a link checker that validates both `href` and `src` against the files in `dist/`. Checking only `href` once let a missing `/favicon.svg` ship broken on every page.
