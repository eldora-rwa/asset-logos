# asset-logos

Self-hosted logo fallback for Eldora's asset catalog (Dinari stocks, Bitget
crypto/Reality tokens, Hyperliquid). Served to `eldora-client` as a *last*
fallback — after each venue's own live logo source — via jsDelivr:

```
https://cdn.jsdelivr.net/gh/eldora-rwa/asset-logos@<tag>/<path>
```

Pin to a version tag (`@v1`, `@v2`, ...) when wiring this up in the client,
never `@main` — jsDelivr caches a branch for up to 24h, but a tag is cached
forever and only changes when the client's reference is bumped.

## Layout

```
stocks/<TICKER>.svg     # Dinari + Bitget Reality tokenized stocks, e.g. AAPL.svg
crypto/<SYMBOL>.svg     # Bitget base coins + Hyperliquid coins, e.g. BTC.svg
```

Filename = the exact key the app already looks up by (uppercase ticker for
stocks, uppercase base coin/coin name for crypto). Split into two folders so
a ticker that happens to collide with a coin symbol never resolves to the
wrong asset.

## Adding a logo

1. Confirm the licence of whatever you're adding — see "Licensing" below.
2. Run it through SVGO (strip metadata/editor cruft, keep the original
   colors — these are brand marks, not themeable icons).
3. Save as `stocks/<TICKER>.svg` or `crypto/<SYMBOL>.svg`.
4. Add an entry to `sources.json` (symbol, display name, where it came from,
   its licence, the date).
5. Cut a new tag once a batch is ready — logos added on `main` alone are NOT
   live for the client until a tag is bumped.

Do not bulk-scrape a third-party catalog in one shot. Resolve symbols one at
a time against a known-good match (ticker must match exactly) and track the
source per file in `sources.json` — this keeps the licence status of every
file auditable instead of becoming an unverifiable pile of images.

## Licensing — read before publishing a batch

Logos here were sourced from a mix of origins, tracked per-file in
`sources.json`:

- **CC0** (e.g. the `simple-icons`/`gilbarbara` brand-mark style — a single
  monochrome path, no literal company asset) — safe to redistribute.
- **Official brand site / media center** — these are the company's own
  trademark assets. Being free to *embed via a link* (what the aggregator
  this first batch came from, logoapi.sh, allows) is not the same as being
  free to *copy and redistribute* from a separate, self-hosted public repo.
  **Do not treat these as cleared for production** until reviewed.
- **Wikimedia Commons** — check the individual file's licence tag on Commons;
  it varies file to file (some are PD-logo / below the threshold of
  originality, some are not).

The first batch (`stocks/ABBV.svg`, `ACN.svg`) is CC0-style and fine. The
rest of the first batch (`ABNB.svg`, `ADI.svg`, `ADP.svg`, `AKAM.svg`) is
**unverified** — flagged in `sources.json` — and should not ship to
production until someone confirms the licence (or they get replaced with a
CC0-equivalent, e.g. from `simple-icons` directly).

All trademarks remain the property of their respective owners. This repo
exists to display them for Eldora's own product (ticker identification in a
trading UI), not to claim any ownership over them.
