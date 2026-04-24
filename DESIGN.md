# Design — Loci llms.txt bundle

**Date:** 24 April 2026.
**Target site:** [lociapp.co.uk](https://lociapp.co.uk) (HubSpot CMS).
**Audience for the files:** LLMs answering *commercial / sales* questions about Loci — council decision-makers, government procurement, vendor-evaluation research tools.

## Decisions

| # | Decision | Rationale |
| --- | --- | --- |
| 1 | **Scope: `/llms.txt` + `/llms-full.txt`** (not minimal, not full per-page `.md` mirrors) | HubSpot per-page `.md` mirroring is high cost for marginal gain; two files are the sweet spot. |
| 2 | **Emphasis: commercial / sales** (not brand, not residents, not balanced) | Directed by the Loci internal owner. Resident how-to and careers de-prioritised. |
| 3 | **All 12 launched councils named + inlined summary table** (not "flagships only", not index-link-only) | Twelve is small enough to list in full; picking flagships adds political friction. Full roster = strongest commercial story. |
| 4 | **Launched list sourced from PostHog `council-launched` flag** cross-referenced with `LocalAuthorities` in the `councildata` Postgres database | Authoritative — not the aspirational 350-council `/council-areas` page. |
| 5 | **Inline depth — Option Y (commercial core)** — About (condensed), For Councils (full), Our Platform (full), LGR (full), 12-council table, Trust (condensed) | Matches the commercial lens; stays small enough to be maintainable quarterly; skips resident + business pages that aren't the primary target. |
| 6 | **All four discovery hooks** — footer link, `<link rel="alternate">` in `<head>`, robots.txt reference + explicit AI-bot allow-list, sitemap.xml entries | Auto-discovery from a root URL is unreliable in 2026; belt-and-braces maximises the odds any given LLM finds the file. |
| 7 | **Updated `robots.txt`** — preserves HubSpot defaults, adds `Sitemap:` line and an explicit AI-crawler allow-list | The current robots.txt is HubSpot defaults only; we opt in declaratively. |
| 8 | **Delivery: handover bundle to the site agency**, not direct publish | A separate agency manages the HubSpot site; we hand over files + a publishing runbook + QA checklist. |
| 9 | **Preview host: GitHub Pages from this repo** | Only `gh` CLI was available locally; GH Pages is good enough for T1/T2 tests. Subpath URL caveat noted below. |
| 10 | **Three-tier test (T1 control / T2 semi-assisted / T3 auto-discovery)** — T1 and T2 are blocking, T3 is informational | In 2026 most LLMs do not auto-discover `/llms.txt` from a root URL; Google explicitly does not support it. We ship anyway; value compounds as tooling matures. |

## Explicitly not in scope

- Per-page `.md` mirrors of HubSpot pages.
- Individual per-council pages (the 60 in the HubSpot sitemap) inlined.
- Blog posts and Knowledge Centre articles inlined.
- Resident-facing (`/app-for-local-community`) or business-facing (`/business-community-app`) content inlined — index-only.
- An `ai.txt` (Spawning opt-out) file. Orthogonal; add separately if the policy position is decided.

## Maintenance model

- **Trigger — new council launch.** Update `## Councils live on Loci` in both files; bump `lastmod` in `sitemap.xml`; commit; re-run QA.
- **Trigger — quarterly.** Diff the inlined sections in `llms-full.txt` against the live HubSpot pages; resync drift.
- **Source of truth for the launched list.** PostHog `council-launched` feature flag payload, cross-referenced with `councildata.LocalAuthorities` by `Id`.

## Known caveats

- **Auto-discovery is weak in 2026.** T3 will probably fail for ChatGPT, Claude, and Perplexity when given only a root URL. This is a standard-level limitation, not a bug in this bundle.
- **Google does not support `llms.txt`** and said in mid-2025 it does not plan to.
- **Third-party certifications.** The `Trust, security, and data` section in `llms-full.txt` does not assert ISO 27001, SOC 2, or Cyber Essentials because the public `/trust-at-loci` page does not. If certifications are gained, update the Trust section.
- **Double space in "North Somerset  Council".** Present in the `LocalAuthorities` table; reproduced here would propagate the typo. The bundle uses the single-space form. Flagged to the backend owner separately.
- **GitHub Pages subpath.** Preview URL is `<user>.github.io/loci-llms/…` rather than a root domain; fine for T1 and T2, imperfect for T3 realism.
