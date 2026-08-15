# oem-price-data

The authored source of EnergyKaki's retailer electricity price plan data,
also published as static JSON on GitHub Pages.

**The apps no longer read from here.** Since 2026-08-10 both iOS and Android
fetch from the Cloudflare Worker at `https://api.energykaki.com/data/...`,
which serves from Workers KV behind a shared-secret header. This repo stays
the place the data is *authored* and the public copy of it; the KV copy is
what users actually get. Edits made here reach users only after the mirror
and publish steps below.

Full pipeline spec: see `retailer-plan-data-sync.md` in the
[`EnergyKaki-specs`](../EnergyKaki-specs) repo.

## Why this repo is separate

Kept independent of either app's codebase so the data lifecycle isn't tied
to an app release cycle, and both platforms can point at the same URLs.

## Files

```
data/
├── current.json          — retailers + active plans, loaded on every app launch
├── history.json          — dated snapshots of plan prices and promos, loaded lazily
└── tariff-override.json  — the SP regulated tariff, current rate + quarterly series
```

Published at (public, no longer read by the apps):

```
https://<username>.github.io/oem-price-data/data/current.json
https://<username>.github.io/oem-price-data/data/history.json
```

## How data reaches users

Three copies, in order:

1. **`oem-price-data/data/`** — authored here, pushed to GitHub Pages
2. **`EnergyKakiData/data/`** — a local-only mirror, copied from (1); the
   directory the publish script reads
3. **Cloudflare Workers KV** — written by
   `EnergyKakiAPI/scripts/publish-price-data.sh`, and what the apps fetch

The daily `energykaki-price-update` scheduled task does all three
unconditionally on every run. Doing it by hand means copying `data/*.json`
into `../EnergyKakiData/data/`, committing there, then running that publish
script — a commit here alone changes nothing for users.

## Updating

Data is entered and verified manually against each retailer's own
price-plan page (not `compare.openelectricitymarket.sg` — see the spec's
"Why not OEM" note) and, for plans with non-trivial rate structures, the
plan's EMA Fact Sheet. Automated scraping was evaluated and ruled out — see
the spec's "Why This Approach" section for why. Until the
`admin-data-entry.html` tool referenced in the spec exists, edit the JSON
files directly, keeping the schema intact, then commit and push:

```
git add data/current.json data/history.json
git commit -m "Update price plans as of <date>"
git push
```

GitHub Pages picks up changes within a few minutes of pushing to `main`.
Remember that this only updates the public copy — see "How data reaches
users" above for the mirror and KV publish that put it in front of the apps.

### `history.json` and promos

Every snapshot entry carries `signupPromo`, mirroring `current.json`. It is
what powers the promo section of the app's What's New screen, so it must be
written on every new snapshot. Two values that look alike but are not:

- `""` — recorded, and the plan had **no promo** that day
- field **absent** — never recorded; the app skips it rather than reporting
  a promo appearing out of nowhere

Never write `null` here. Snapshots predating the field were backfilled from
this repo's own git history, where each day's `current.json` commit carries
that day's promo.
