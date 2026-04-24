# Publishing runbook — Loci llms.txt for lociapp.co.uk (HubSpot CMS)

**Audience:** the agency that manages `lociapp.co.uk` on HubSpot CMS.
**Owner at Loci:** _fill in before handover._
**Estimated time:** 30–45 minutes.
**Pre-requisites:** HubSpot admin access with File Manager and theme-template edit permissions.

---

## 1. Files to publish at the site root

Upload these three files from the bundle to HubSpot and make them available at the site root of `lociapp.co.uk`. Each must be served as **plain text** (or markdown), **not** as HTML. If HubSpot wraps them in a template, the LLM tests will fail.

| Bundle file | Publish at | Expected `Content-Type` |
| --- | --- | --- |
| `llms.txt` | `https://lociapp.co.uk/llms.txt` | `text/plain; charset=utf-8` or `text/markdown; charset=utf-8` |
| `llms-full.txt` | `https://lociapp.co.uk/llms-full.txt` | `text/plain; charset=utf-8` or `text/markdown; charset=utf-8` |
| `robots.txt` | `https://lociapp.co.uk/robots.txt` (replaces the current file) | `text/plain; charset=utf-8` |

### How to upload in HubSpot

1. **Settings → Tools → Marketing → Files and Templates → Files.**
2. Upload `llms.txt`, `llms-full.txt`, and `robots.txt`.
3. For each file, click the file → **Edit file details** → set the URL to the exact root paths in the table above. If HubSpot appends a folder or hash, use the **Change file URL** option (available on Marketing Hub Professional and above) to map the CDN URL to the root path on `lociapp.co.uk`. If that option is not available, the next section describes the HubDB / custom-module fallback.
4. Confirm the response from a shell or browser:
   ```
   curl -I https://lociapp.co.uk/llms.txt
   ```
   Expected: `HTTP/2 200` and `content-type: text/plain; charset=utf-8` (or `text/markdown`). **Do not accept `text/html`.**

### HubSpot fallback if File Manager root-URL mapping is not available

If the Hub plan does not support root-URL file mapping, publish each file as a HubSpot **website page** using a one-block **plain text / raw HTML** template that outputs only the file contents with an explicit `Content-Type` override.
Specifically:

- Create a new Website Page with a blank custom template.
- In the template, emit the file content inside `<pre>{{ content }}</pre>` **is not acceptable** — LLMs must receive the raw bytes, not HTML. Instead, use HubSpot's Serverless Functions (Professional+) or a HubDB-backed redirect to serve the files with the correct content-type. Any solution must pass the `curl -I` check above.

If neither root-URL mapping nor serverless functions are available, escalate to Loci — do not ship HTML-wrapped files.

---

## 2. Add discovery hooks to the main theme

Two small theme edits help LLM crawlers and tooling find the index.

### 2a. `<head>` — on every page

Add the following two lines to the main theme's `<head>` partial (typically `templates/partials/head.html` or equivalent):

```html
<link rel="alternate" type="text/markdown" title="Loci LLM Index" href="/llms.txt">
<link rel="alternate" type="text/markdown" title="Loci LLM Full Context" href="/llms-full.txt">
```

### 2b. Footer — on every page

Add a small "LLM index" link alongside the existing legal footer links. Example in the footer module:

```html
<a href="/llms.txt" rel="alternate" type="text/markdown">LLM index</a>
```

This is functionally important — LLMs with browsing often follow links from the pages they fetch.

---

## 3. Update `sitemap.xml`

HubSpot generates `sitemap.xml` automatically. The bundle file `sitemap.xml` is **not for production** — do not upload it.

Instead, configure HubSpot to include `/llms.txt` and `/llms-full.txt` in the generated sitemap:

- **Marketing → Website → Pages → Sitemap settings.** Add both URLs as additional sitemap entries.
- If HubSpot does not allow adding non-page URLs, add a manual secondary sitemap file (`sitemap-llms.xml`) at the root containing the two entries, and reference it from `robots.txt` via a second `Sitemap:` line. (Requires root-file publishing, same as section 1.)

After publish, verify:

```
curl -s https://lociapp.co.uk/sitemap.xml | grep -E "llms\\.txt|llms-full\\.txt"
```

Both should appear.

---

## 4. Cache considerations

- `llms.txt` and `llms-full.txt` can change when a new council launches or on the quarterly refresh. Set **Cache-Control: public, max-age=3600** (1 hour) or lower on these files — do not inherit HubSpot's longer page cache.
- `robots.txt` should be served with a short cache TTL (1 hour or less) so future updates propagate quickly.

Confirm with:

```
curl -I https://lociapp.co.uk/llms.txt | grep -i cache-control
```

---

## 5. Hand back to Loci

Once all of the above is live in production, let Loci know so we can run `QA-CHECKLIST.md`. If anything in this runbook is blocked by a HubSpot plan limitation, flag it — we'd rather solve it than ship a half-working setup.
