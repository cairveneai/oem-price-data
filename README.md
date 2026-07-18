# oem-price-data

Static JSON hosting for EnergyKaki's retailer electricity price plan data.
Served via GitHub Pages and fetched directly by the iOS and Android apps —
no backend server involved.

Full pipeline spec: see `retailer-plan-data-sync.md` in the
[`EnergyKaki-specs`](../EnergyKaki-specs) repo.

## Why this repo is separate

Kept independent of either app's codebase so the data lifecycle isn't tied
to an app release cycle, and both platforms can point at the same URLs.

## Files

```
data/
├── current.json   — retailers + active plans, loaded on every app launch
└── history.json   — dated snapshots of plan prices, loaded lazily
```

Served at:

```
https://<username>.github.io/oem-price-data/data/current.json
https://<username>.github.io/oem-price-data/data/history.json
```

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
