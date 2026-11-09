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

- **CC0** — confirmed by matching the file against
  [`logoapi/logo-api`](https://github.com/logoapi/logo-api)'s `tickers/` or
  `logos/` folder, which its `NOTICE` file dedicates to CC0 1.0 Universal.
  Safe to redistribute. This is the **only** tier that should go in this
  repo going forward — see below.
- **Official brand site / media center / Wikimedia** — a company's own
  trademark asset, or an individually-licensed Commons file. **Not safe to
  self-host in bulk**: `logoapi/logo-api`'s own `official/` folder (not CC0,
  separately licensed) says explicitly *"do not redistribute them in bulk as
  a logo collection"* — the exact thing a repo like this one is. Flagged
  `UNVERIFIED` in `sources.json`, kept only as a record of what was tried;
  **do not wire these into the client**, and prefer removing a file once a
  genuine CC0 equivalent is found for it (see `ABBV`/`ACN`/`BAC`/`GE`, removed
  after `logoapi/logo-api` confirmed they come from its restricted
  `official/tickers/`, not `tickers/`).

A structural heuristic used early on ("single flat `<path>`, a `<title>`, no
embedded raster → CC0") turned out to be **unreliable** — `official/`'s own
files share that exact shape. Don't use it to infer a licence; only
`logoapi/logo-api`'s `tickers/`/`logos/` folders (or another source with an
explicit CC0/public-domain grant) count as confirmed.

`logoapi.sh`'s **search API is capped at 500 requests/day** (separate from
the "2M requests/month" figure on their homepage, which is for fetching a
logo you already have the URL for — search is the bottleneck). A batch that
needs more than ~500 lookups must detect `{"error":"daily_limit"}` and stop
rather than silently miscounting a rate-limited response as "not found".

All trademarks remain the property of their respective owners. This repo
exists to display them for Eldora's own product (ticker identification in a
trading UI), not to claim any ownership over them.
