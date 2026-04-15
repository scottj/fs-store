# fs-store

A single-page static site hosted via GitHub Pages at `store.full-speed.org` (see `CNAME`). Its sole job is to redirect visitors to Amazon — either to a fixed affiliate short link, or, when given an ASIN in the query string, to a `amazon.com/dp/{ASIN}?tag=20-4444-20` affiliate URL.

## Layout

- `src/index.html` — authoring source. Edit this, not the root file.
- `index.html` — minified build output, committed to the repo so GitHub Pages serves it. Do **not** hand-edit; regenerate via the minify script.
- `scripts/minify.sh` / `scripts/minify.ps1` — build scripts. Both run `bunx html-minifier-terser` with identical flags; keep them in sync when changing flags.
- `CNAME` — GitHub Pages custom domain (`store.full-speed.org`).

## Redirect behavior

Implemented in `src/index.html`:

- **No query string / invalid ASIN / JS disabled** → `https://amzn.to/3G9L2lG` (the JS fallback and the `<noscript><meta refresh></noscript>` both point here).
- **Valid ASIN** — `location.search.slice(1)` matches `/^[A-Za-z0-9]{10}$/` → `https://www.amazon.com/dp/{ASIN}?tag=20-4444-20` (uppercased).
- ASIN is the *entire* query string with no key, e.g. `?B0ABCDEFGH` — not `?asin=...`.
- Redirect uses `location.replace()` to avoid history pollution.

The affiliate tag `20-4444-20` and the fallback short link `amzn.to/3G9L2lG` are load-bearing constants — don't change them without the user's say-so.

## Build

From the repo root:

```sh
bash scripts/minify.sh        # or: pwsh scripts/minify.ps1
```

This reads `src/index.html` and writes `index.html`. Requires `bun` on PATH.

Every edit to `src/index.html` must be followed by running the minify script and committing **both** files together — GitHub Pages serves the root `index.html`, so forgetting to rebuild ships stale content.

## Testing changes

Local verification (open the built `index.html` in a browser or serve it):

1. No query string → lands at `amzn.to/3G9L2lG`.
2. `?B0ABCDEFGH` → lands at `amazon.com/dp/B0ABCDEFGH?tag=20-4444-20`.
3. `?bogus` (wrong length/chars) → falls through to `amzn.to/3G9L2lG`.
4. JS disabled in devtools → `<noscript>` meta refresh fires to `amzn.to/3G9L2lG`.
