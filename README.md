# weather-data-mirror

Latest publishable mirror of high-frequency Himalayan Atlas data, written by [Himalayan Atlas's](https://github.com/SushantChalise/Weather) GitHub Actions cron and served to end users via the [jsDelivr](https://www.jsdelivr.com/) public CDN.

## Scope — read this first

This repo holds the **latest publishable state** of high-frequency datasets plus a short rolling history (typically 24–48 frames). It is **not a permanent archive**.

If you want historical reconstruction beyond the rolling window:

| Dataset | Authoritative source |
|---|---|
| Himawari-9 cloud imagery | [Japan Meteorological Agency](https://www.data.jma.go.jp/mscweb/data/himawari/) |
| Glacier outlines | [ICIMOD RDS portal](https://rds.icimod.org/), [GLIMS](https://www.glims.org/), [RGI](https://www.glims.org/RGI/) |
| Reanalysis weather | [Copernicus CDS](https://cds.climate.copernicus.eu/), [Open-Meteo](https://open-meteo.com/) |

This repo is regenerable from those sources at any time. Treat it as a CDN-friendly cache, not a database of record.

## How it works

1. The Himalayan Atlas's [`scripts/ingestion/himawari/`](https://github.com/SushantChalise/Weather/tree/main/scripts/ingestion/himawari) cron runs every 30 min.
2. It downloads the latest Himawari-9 B13 frame from JMA, renders WebP tiles, and pushes them to the `data-mirror` branch of this repo.
3. `manifest.json` is updated alongside; jsDelivr's manifest URL is purged so frontends pick up the new frame within ~1–2 min.
4. Tiles are commit-pinned in jsDelivr URLs (cached forever); only the manifest is purged on each push.

## URLs

**Manifest** (branch-aliased, purged on each push):

```
https://cdn.jsdelivr.net/gh/SushantChalise/weather-data-mirror@data-mirror/himawari/latest/manifest.json
```

**Tiles** (commit-pinned, immutable):

```
https://cdn.jsdelivr.net/gh/SushantChalise/weather-data-mirror@<commit_sha>/himawari/latest/tiles/{z}/{x}/{y}.webp
```

The current commit SHA is exposed in the manifest's `commit_sha` field; clients should always use it to construct tile URLs.

## Repository layout

```
main (this branch)
├── README.md
├── LICENSE
└── .github/workflows/compact.yml      ← monthly orphan-rebase

data-mirror (live branch — orphan, force-pushable)
└── himawari/
    └── latest/
        ├── manifest.json              ← live manifest (purged after each push)
        ├── preview.webp               ← single-image preview of the latest frame
        ├── metadata.json              ← provenance + render parameters
        └── tiles/{z}/{x}/{y}.webp     ← tile pyramid, z=0..2 (Nepal bbox)
```

## Maintenance

- **Repo size:** kept under jsDelivr's 150 MB working-tree recommendation by storing only the latest frame plus a short rolling history on the `data-mirror` branch.
- **Compaction:** [`compact.yml`](.github/workflows/compact.yml) runs monthly to orphan-rebase `data-mirror` (drops history; keeps the working tree). This also runs manually via `gh workflow run compact.yml`.
- **Hard limits respected:**
  - Per-file: ≤ 5 MB (jsDelivr supports up to 20 MB but we stay well under)
  - Working tree: ≤ 100 MB (jsDelivr docs recommend < 150 MB)
  - File count: bounded by the tile pyramid (~21 tiles per frame at z=0..2)

## License

[CC0 1.0 Universal](LICENSE) — public domain dedication. The original sources (JMA Himawari, ICIMOD, etc.) are themselves free to redistribute. Mirroring rather than re-licensing.

## Issues / PRs

Open issues and PRs against [SushantChalise/Weather](https://github.com/SushantChalise/Weather/issues), not this repo. This repo holds generated data; the code that generates it lives upstream.
