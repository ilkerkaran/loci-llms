# QA checklist — post-publish verification

Run through this after the agency confirms the files are live on `lociapp.co.uk`. Copy this file to a fresh issue or doc and tick the boxes as you go.

## Mechanical checks

- [ ] `curl -I https://lociapp.co.uk/llms.txt` returns **200** and `content-type: text/plain; charset=utf-8` (or `text/markdown`). **Not** `text/html`.
- [ ] `curl -I https://lociapp.co.uk/llms-full.txt` — same expectations as above.
- [ ] `curl -I https://lociapp.co.uk/robots.txt` returns 200 and `text/plain`.
- [ ] `curl -s https://lociapp.co.uk/llms.txt | head -20` — output is raw markdown starting with `# Loci`, no HTML wrapper, no cookie-banner noise.
- [ ] `curl -s https://lociapp.co.uk/llms.txt | wc -l` — line count roughly matches the bundle file (within a few lines for trailing newlines).
- [ ] `diff <(curl -s https://lociapp.co.uk/llms.txt) ./llms.txt` — byte-identical (or ignorable whitespace diff only).
- [ ] Same `diff` for `llms-full.txt` and `robots.txt`.
- [ ] `curl -s https://lociapp.co.uk/robots.txt | grep -i "sitemap:"` — returns the Sitemap line.
- [ ] `curl -s https://lociapp.co.uk/sitemap.xml | grep -E "llms\\.txt|llms-full\\.txt"` — both entries present.
- [ ] View-source of the lociapp.co.uk homepage — the two `<link rel="alternate" type="text/markdown" …>` tags are present in `<head>`.
- [ ] Homepage footer contains a visible "LLM index" link pointing at `/llms.txt`.

## Content sanity checks

- [ ] The 12 councils named in `llms.txt` match the current PostHog `council-launched` flag. Re-run the DB query if uncertain.
- [ ] No private URLs, internal Jira references, staging hostnames, or test email addresses leaked into either file.
- [ ] All links in `llms.txt` resolve to 200 (spot-check 3–4 of them).
- [ ] The British English / American English split matches the rest of the site (British everywhere except product identifiers).

## Three-tier LLM test

Test against the live production URLs (not the Netlify / GitHub Pages preview) once published. Record the LLM's answer for each test so we can track behaviour over time.

### T1 — Control (explicit URL). Must pass.

Prompt (paste into ChatGPT with browsing, Claude.ai with web search, Perplexity):

> Summarise Loci using the content at `https://lociapp.co.uk/llms.txt`. What councils is Loci live with? Who is it aimed at? What are the core commercial propositions?

- [ ] ChatGPT — answer accurately names councils, identifies the tri-stakeholder model, and mentions B2B cost savings.
- [ ] Claude — same pass criteria.
- [ ] Perplexity — same pass criteria.

### T2 — Semi-assisted. Must pass.

Prompt:

> Tell me about `https://lociapp.co.uk`. If the site has an `llms.txt` or AI index available at the root, use it.

- [ ] ChatGPT — fetches `/llms.txt` and uses it; answer is accurate.
- [ ] Claude — same.
- [ ] Perplexity — same.

### T3 — Pure auto-discovery. Informational only.

Prompt:

> What does `https://lociapp.co.uk` do?

- [ ] Note which LLMs (if any) auto-discovered `/llms.txt` without being told. Not blocking — this is evidence for our own records.

## Negative tests

- [ ] Fetching `https://lociapp.co.uk/llms.txt?hs_preview=something` — does the Disallow rule in robots.txt still behave correctly for crawlers? (Spot-check robots.txt parse via [https://technicalseo.com/tools/robots-txt/](https://technicalseo.com/tools/robots-txt/) or similar.)
- [ ] View the file on mobile — HubSpot sometimes serves a different template on mobile UAs. Confirm plain text both ways.

## Sign-off

- [ ] Loci internal owner has reviewed the live files end-to-end.
- [ ] Agency has been told what (if anything) to fix.
- [ ] A reminder is scheduled for the quarterly content refresh.
