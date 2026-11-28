---
name: verify
description: Before a PR - which check proves which change (check:types, build:all, check:links), and the map of site flows to look at on the built pages, with what each must show.
---

# Verify a change

Level: **content site** (static Astro, no server code in this repo). The build is the run: a change is
verified when the checks below pass and the changed pages have been looked at. The PR says how it was
checked ("brak (powód)" is allowed when nothing can be run); the PR template has the section.

## Commands

| Command | Proves |
|---|---|
| `npm run check:types` | `astro check`: types and Astro diagnostics (CI) |
| `npm run build` | main site into `dist/` (17 pages, incl. /panel/; /galeria/ is switched off) |
| `npm run build:all` | main site + `dist-webasto/` + `dist-eberspaecher/` (CI; the production build runs the same) |
| `npm run check:links` | internal links and `#anchors` in all three `dist*` dirs; run it after `build:all` (CI) |
| `npm run check:ads` | ad texts still match the site; needs `private/ads/` (README › Private files), not run in CI |
| `npm run dev` / `dev:webasto` / `dev:eberspaecher` | dev servers on ports 4321 / 4322 / 4323 for looking at a page |

CI (`.github/workflows/pr-checks.yml`) runs `check:types`, `build:all`, `check:links` and the guards for
agent files, comments and the docs budget (`.github/scripts/*.py`, vendored: never edit them here).
Before the PR run the first three locally; for a change in `.agents/`, `AGENTS.md` or `.md` files run the
guard scripts the way the workflow does (`python3 .github/scripts/agents_check.py . --base origin/main`,
same for `docs_check.py` and `check_comments.py` with `--mode enforce`).

Which check for which change:

- `src/pages/`, `src/components/`, `src/styles/`, `src/data/` → `build:all` + `check:links`, then the flows below at 1440 px and 390 px.
- `sites/<brand>/`, `sites/shared/` → `build:all`, then the brand flow; only that brand's photos and products.
- Copy, prices, address, hours, phones, page URLs → also `check:ads` (AGENTS.md), when `private/` is present.
- Types, `astro.config.mjs`, `css-target.mjs`, dependencies → `check:types` + `build:all`.
- `public/contact.php`, `public/panel/` → PHP does not run in dev or CI; say so in the PR and what the owner has to try on the hosting.
- Docs only → the docs and agents guards; no build needed.

## Flows (on the built pages: `npm run preview`, or a dev server)

1. **Home** `/` — h1, tagline "Komfortowy klimat w zasięgu ręki", sections and CTA render; no text overflows its column at 1440 px and 390 px (and ~1024/1366 for grids and bars).
2. **Subpages** (`/cennik/`, `/klimatyzacja/`, `/ogrzewanie-postojowe/`, `/webasto/`, `/eberspacher/`, `/defa/`, `/tunap/`, `/gwarancja/`, `/jak-pracujemy/`, `/o-firmie/`, `/rodo/`, …) — Header › PageHero › content › CTABanner › Footer; text, data and prices exactly as before; "Ceny aktualne na dzień …" shows today's date.
3. **Contact** `/kontakt/` — the form validates on the client (required name and email; phone required when the preferred contact is phone) and the honeypot stays hidden. The send itself needs the PHP handler: only build and client validation can be checked here.
4. **Webasto brand site** `dist-webasto/` (`npm run dev:webasto` or `npx astro preview --config sites/webasto/astro.config.mjs`) — one page, only Webasto photos and products, footer links only to the main site; no link to the other brand, DEFA or air conditioning.
5. **Eberspächer brand site** `dist-eberspaecher/` — the same, with Eberspächer only.
6. **Production build** `npm run build:all` — all three outputs build (17 + 1 + 1 pages); `grep -rl noindex dist dist-webasto dist-eberspaecher` lists only `/panel/` (`PUBLIC_NOINDEX` is set only for the preview build); `check:links` passes.
7. **Panel** `/panel/` — the page builds and loads; login and saving need the server side (`public/panel/api.php`), which does not run here. Never add a `status.json` to the build.

A visual change is looked at in a browser (screenshots at 1440 px and 390 px); a markup-only change at
least through the built HTML.

## When a check fails

- Fix the cause; never skip a check, rerun until green, or edit the vendored guard scripts.
- A broken link or anchor reported by `check:links` is a real defect in the page or in the target.
- Three rounds of fixes and the same check still fails: stop and ask the owner.
- Publishing to the hosting is not part of this repo's checks: it happens only when the owner asks.
