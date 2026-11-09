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
stocks/<TICKER>.svg|.png|.jpg   # Dinari + Bitget Reality tokenized stocks, e.g. AAPL.svg
crypto/<SYMBOL>.svg|.png|.jpg   # Bitget base coins + Hyperliquid coins, e.g. BTC.svg
```

Most files are `.svg`; a few are `.png`/`.jpg` where that was the only
format the source actually served (Bitget's own CDN, mainly) — the app's
resolver needs to try the file extension it finds, not assume `.svg`.

Filename = the exact key the app already looks up by (uppercase ticker for
stocks, uppercase base coin/coin name for crypto). Split into two folders so
a ticker that happens to collide with a coin symbol never resolves to the
wrong asset — confirmed real, not hypothetical: Bitget lists both a `DASH`
*stock* (DoorDash, tokenized as its Reality ticker) and an unrelated `DASH`
*coin* (the Dash cryptocurrency). A resolver that doesn't know which folder
it's looking in will serve the wrong one.

**Bitget Reality/RWA tokens reuse `stocks/` directly, no separate copy
needed.** Bitget's Reality catalog tokenizes almost every US-listed ticker
(3,212 of them, checked 2026-11-09), so any ticker already in `stocks/` —
whether it got there via Dinari or added for Bitget specifically — already
answers both. Only add a file under `stocks/` once per ticker.

**Hyperliquid currently has no gap.** Checked every active perp (179) and
every spot base token with a real market (129) against both of its own live
logo sources (`app.hyperliquid.xyz/coins/`, then the `spothq/
cryptocurrency-icons` CDN fallback — see `CoinIcon.tsx`): all 368 resolve.
Nothing to add here until one of those sources actually goes dark for a
specific coin.

**Bitget's own markets/stocks page (`bitget.com/.../markets/stocks`) is a
first-party source, not a third party.** It's a client-rendered page with no
dedicated public REST docs, but its own JS bundle calls
`www.bitget.com/v1/spot/public/getCoinChainList` (crypto + many RWA tickers,
keyed by `coinName`, with `imgUrl` + `fullName`) and
`www.bitget.com/v1/mix/market/rwa/multiLanAll` (RWA company names, no image).
38 stock logos here came from `getCoinChainList`, filtered to entries whose
`fullName` carries an explicit `RWA`/`STOCK` marker or a real company-name
match against Dinari's own name for that ticker — **never** a bare
ticker-equals-ticker "match" with no corroborating text, which is exactly how
three high-confidence-looking hits turned out wrong during this pass: `GBTC`
→ "Green Bitcoin" (an unrelated meme coin, not Grayscale's trust), `NAKA` →
"Nakamoto Games" (an unrelated gaming token, not the Bitcoin-treasury stock),
`VRTX` → "Vertex Protocol" (an unrelated DeFi project, not Vertex
Pharmaceuticals). All three were dropped. 18 further candidates (`STT`,
`CIFR`, `IBKR`, `BWET`, `VKTX`, `HLT`, `MSCI`, `MDT`, `DHR`, `HUM`, `VLO`,
`CVNA`, `BIRK`, `CWEN`, `VT`, `CRK`, `SYBT`, `JBLU`) passed the same vetting
but their image URLs 403'd on a plain fetch (likely referrer-gated) and were
left unresolved rather than worked around.

**`financialmodelingprep.com/image-stock/<TICKER>.png` is a public, no-key
logo endpoint with very high US-ticker coverage** — 628/632 (99%) of the
Dinari symbols still missing a logo resolved here in one pass. Real 404 for
an unknown ticker (checked against a made-up symbol); for a ticker it does
recognise but has no dedicated artwork for, it serves the fund
sponsor/issuer's own mark instead of the individual fund's (e.g. every
iShares ETF gets the same small iShares "i" mark) — still a real, correct
logo, just shared across a family of funds, not a blank placeholder. Every
file checked during this pass was a real brand mark; none were a generic
fallback graphic. Not a documented/public API product — no terms could be
confirmed (ToS page 403'd automated fetches) — so these are tracked
`UNVERIFIED` same as the Bitget-CDN batch, not CC0.

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

## Confirmed-CC0 sources (prefer these over logoapi.sh)

Checked directly against the upstream project's own licence, not scraped
through a search API:

- [`simple-icons/simple-icons`](https://github.com/simple-icons/simple-icons)
  — CC0-1.0, ~3,470 brand marks, matched here by exact normalized company
  name against its `title` field.
- [`gilbarbara/logos`](https://github.com/gilbarbara/logos) — CC0-1.0,
  ~2,068 brand marks with a `url` (company domain) per entry; matched by
  name and by domain-slug guess (`logos.json`'s `url` field), each candidate
  verified by reading the file's own `<title>` before adding it.
- [`logoapi/logo-api`](https://github.com/logoapi/logo-api)'s own `tickers/`
  and `logos/` folders — CC0-1.0 per that repo's `NOTICE` (its `official/`
  folder is NOT CC0 and explicitly forbids bulk redistribution — never pull
  from there, and never trust its live search API's result shape as a
  licence signal, see the postmortem below).

Both `simple-icons` and `gilbarbara/logos` skew toward tech/SaaS brands but
also carry plenty of large non-tech names (Boeing, Coca-Cola, Goldman Sachs,
Wells Fargo, FedEx, Target, 3M, ...), which is where most of this repo's
stock coverage now comes from. Still skews large-cap/consumer-recognizable —
expect thin coverage for mid-cap industrials, utilities, regional banks, and
essentially none for ETFs (they have no "brand" to match).

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
