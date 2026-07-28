# Tourism, Hajj & Umrah — Saudi Arabia in open data

Built by [Samer Ajlawi](https://www.linkedin.com/in/ajlawi/). MIT licensed — issues and PRs welcome.

A public dashboard over Saudi government open data:

- **Ministry of Hajj & Umrah** open-data library — pilgrim volumes, seasonality, permits,
  accommodation class, agent networks
- **Ministry of Tourism** series via the National Open Data Platform — inbound and domestic
  visitors, spending by purpose, hotel and apartment occupancy, licensed facility stock

**Static. No backend, no database, no build step, no dependencies.** Three files ship:
`index.html`, `data.js`, `data.json`. Total under 40 KB.

---

## Run it locally

```bash
cd hajj-umrah-dashboard
python3 -m http.server 8899
# open http://127.0.0.1:8899
```

It also opens straight from the filesystem — `data.js` sets `window.DATA`, so no `fetch()`
and no CORS problem.

## Refresh the data

```bash
pip install openpyxl
python3 build_data.py           # reads the vault mirror, rewrites data.js + data.json
```

Point it elsewhere with `MOHU_FILES=/path/to/xlsx python3 build_data.py`.
Defaults: `MOHU_FILES=~/knowledgevalute/hajj-umrah-open-data/files` and
`TOURISM_FILES=~/knowledgevalute/hajj-umrah-open-data/national-platform/tourism`.

**2025 tourism data is a part year (January–June)** and is flagged `partial` in the same way as
the 1447 pilgrimage months — never summed against a complete year.

To re-pull the source library itself from the Ministry, or the wider national platform,
see `hajj-umrah-open-data/README.md` and `national-platform/README.md` in the vault —
both record the API recipe.

## Deploy

Any static host. No configuration.

### GitHub Pages (recommended — free, and the repo is already set up)

The repo is initialised with a workflow at `.github/workflows/deploy.yml` that publishes on
every push to `main`. To go live:

```bash
gh repo create umrah-open-data --public --source=. --push
# then: Settings -> Pages -> Build and deployment -> Source: GitHub Actions
```

Or with an existing remote:

```bash
git remote add origin git@github.com:<you>/<repo>.git
git push -u origin main
```

The site then serves at `https://<you>.github.io/<repo>/`. A custom domain goes in
Settings → Pages → Custom domain (add a `CNAME` file to the repo root).

`.nojekyll` is present so GitHub serves the files as-is rather than running them through Jekyll.

### Other hosts

```bash
npx wrangler pages deploy . --project-name umrah-open-data   # Cloudflare Pages
npx netlify deploy --prod --dir .                            # Netlify
```

### Keeping it current

The Ministry updates most datasets annually, a few quarterly. A monthly GitHub Action that
runs `build_data.py` and commits the result is enough:

```yaml
on:
  schedule: [{ cron: "0 3 1 * *" }]
  workflow_dispatch:
```

---

## ⚠️ The one rule that must survive any change

**All 1447 source files were published 2026-03-05, mid-Ramadan 1447.** Ramadan is the last
month present in those files and it is a **partial month**.

Comparing it year-on-year produces a false **−44% "Ramadan collapse"**. Two independent
analysis passes over this data both published that claim before it was caught.

`build_data.py` enforces the rule rather than leaving it to the reader:

- the last 1447 month is flagged `partial: true`
- `yoy_by_month` and `growth` are computed over **complete months only**
- the chart draws the partial month faded, with a dashed connector and a "partial" tick label
- the banner at the top of the page states the rule in plain language

It also drops trailing near-zero months as reporting artifacts (1446 entrants ends with a
month of `2`).

**If you change the pipeline, keep this.** The failure mode is silent and the headline it
produces is dramatic and wrong.

## Other source-data cautions

- **Six dataset pairs in the library are byte-identical duplicates.** The file named for
  transport companies is a copy of the Hajj-companies file — there is no transport data.
- The Nusuk regional-users file has region uncaptured on **91% of rows**; only its total is safe.
- The accommodation-classification file carries **no year label** — the hotel mix is undated.
- Season **1442 recorded 13,300 visas** (COVID). Including 1441–1443 in any trend or CAGR
  produces nonsense.

---

## Privacy

`build_data.py` reads only aggregate files. The source library contains files with individual
agent names and email addresses (many of them personal Gmail accounts) — those are **never
read here and must not be published**. Analysing them privately is fine; putting a searchable
contact list on a public site is not.

## Licence and attribution

Data: Ministry of Hajj & Umrah open data <https://haj.gov.sa/en/Open-Data>, and Ministry of
Tourism series via the Saudi National Open Data Platform
<https://open.data.gov.sa/en/datasets?category=tourism>. Both reused under the Saudi Open Data
License. The licence requires **attribution with a link back to the source
portal** — that link is in the page footer and must stay there.

The footer also carries the required disclaimers: that this is an independent analysis, not
affiliated with or endorsed by any government body, and that responsibility for the
interpretation rests with the authors rather than the source.

## Design notes

- Colours come from a validated palette — categorical slots, a single-hue sequential ramp for
  magnitude, and a blue↔red diverging pair for growth. The set passes CVD separation,
  normal-vision separation, lightness-band and chroma checks in **both** light and dark.
  Dark mode is a separately-stepped palette, not an inverted one.
- Three light-mode series sit below 3:1 against the surface, so every chart ships **direct
  labels and a data-table view** — identity is never carried by colour alone.
- Charts are hand-rolled SVG: no chart library, nothing loaded from a CDN, so the page works
  offline and under a strict CSP.
