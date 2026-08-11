# Prompt: repopulate the portfolio scene deck

Give this to an AI assistant together with `portfolio-deck.html` (the blank template in
this repo) and your own holdings data. Everything from `---` down is the prompt — paste
it as-is.

---

You are editing a single self-contained HTML file, `portfolio-deck.html`. It is an
animated five-scene portfolio deck plus a standalone exposure-map page. All of the
code is already written and working. **Your only job is to replace the `DATA` object
near the top of the file with my numbers.** Do not restructure the deck, rename keys,
add libraries, or touch any code below the `ENGINE` banner.

## Rules that matter more than anything else

1. **Never invent a figure.** If I haven't given you a number, leave the placeholder
   in `[BRACKETS]`. The deck renders unfilled values as an em dash and counts them in
   a corner card — that is the intended behaviour, and it is always better than a
   plausible-looking guess. If something important is missing, finish the edit and
   then tell me what you left blank and where to find it.
2. **Do not compute derived numbers I can give you directly.** Weights, gain on cost
   and the local-currency value should come from me. If I ask you to derive a weight
   from a value, show the arithmetic in your reply so I can check it.
3. **Flag inconsistencies rather than silently reconciling them.** If asset-class
   values don't sum to the total, or weights don't sum to ~100%, or the direct
   stocks plus ETFs don't equal the listed-equity sleeve, say so and ask. Don't
   quietly adjust a number to make the arithmetic close.

## The data contract

Replace the `DATA` block with this shape. Rows are examples of shape only — add or
delete as many as I actually hold.

- `title`, `asOf` — free text.
- `home` — `{ name, iso3 }`. **iso3 must be ISO 3166-1 alpha-3**: `GBR`, `PAK`, `USA`,
  `IND`, `DEU`. This drives the lit country on the globe and the map label.
- `currency` — `{ symbol, code }` for the currency the book is priced in.
- `local` — `{ symbol, code, value, grouping }`. `grouping` is `"western"`
  (1,284,930) or `"indian"` (12,84,930, lakh/crore).
- `totals` — `value`, `gainAbs` (may be negative), `gainOnCost` (percent, so `34.2`,
  not `0.342`), and optionally `outsideHomeCurrency`. Leave that last one bracketed
  and the deck estimates it from country exposure and labels it as an estimate.
- `assetClasses[]` — `{ name, value, weight, iso3 }`. `weight` is percent of the
  portfolio. **Only set `iso3` on sleeves that are NOT made up of the stocks and
  funds below** — cash, property, bonds. Tagging the equity sleeve double-counts.
- `liabilities[]` — `{ name, value }`. Delete the array contents if I have none.
- `directStocks[]` — `{ name, ticker, value, iso3, sector }`.
- `etfs[]` — `{ name, ticker, value, constituents[], rest{} }` where:
  - `constituents[]` is `{ name, ticker, weight, iso3, sector }` and **`weight` is
    percent OF THE FUND**, exactly as the factsheet publishes it — not percent of my
    portfolio. Only the names I care about need itemising.
  - `rest` is `{ countries: { ISO3: pct }, sectors: { name: pct } }`, spreading the
    part of the fund I didn't itemise. These are also percent of the fund. Aim for
    constituents plus rest to land near 100% per fund; whatever is left over shows
    honestly as unattributed rather than being spread around.
- `universes` — three arrays: `"Asset class"`, `"Geography"`, `"Sector"`. This is the
  board coverage is measured against; anything listed that I don't touch is drawn
  dark and labelled "none". **Order matters** — "biggest gap" is simply the first
  absent item, so order each list by how much I'd mind not owning it.
- `traceThreshold` — countries below this percent of the book shade as trace rather
  than full exposure on the map. `1.0` is a sensible default.

## Validation pass — run every one of these before you hand the file back

- **Numbers are bare digits.** `1284930`, never `"1.28M"`, `"£1.28m"` or `"1,284,930"`.
  A string with a `[` in it is treated as unfilled; any other unparseable string
  becomes a dash.
- **Tickers match exactly** between `directStocks` and `etfs[].constituents`. This is
  the single most common silent failure: a mismatch means Scene 3 won't detect the
  overlap and the "names you own twice" payoff reads zero. Normalise case and strip
  exchange suffixes consistently across both.
- **Sector strings match** the `"Sector"` universe, and **asset-class names match**
  the `"Asset class"` universe. Matching is case-insensitive but otherwise literal,
  so "Tech" will not light up an "Information technology" cell.
- **Every `iso3` exists in the embedded map data** — Natural Earth 110m, 177
  countries. Notable absences: Singapore, Hong Kong, Malta, Bahrain, Luxembourg and
  most small island states. Exposure tagged to a missing code won't shade the map or
  appear in the ranked list. Tell me which codes you dropped and either roll them
  into a neighbour or leave them unattributed — my call, not yours.
- **Asset-class weights sum to roughly 100.**
- **Direct stocks plus ETF values equal the listed-equity sleeve**, if I have one.
- **Per-fund constituent weights plus rest weights are ≤ 100.**

## Finishing

Save the edited file — same filename unless I say otherwise — and reply with:

1. A short table of what you filled in versus what you left bracketed.
2. Any inconsistency you found, quoted with the numbers involved.
3. The values the deck will now derive, so I can sanity-check them without opening
   it: largest single-country exposure and its percent, count and names of the
   doubled-up holdings, percent of equity sitting in those names, and the held/missing
   counts for each of the three coverage lenses.

If you want to check your work, open the file in a browser: the corner card shows
the number of figures still unfilled, and it should be zero if I gave you everything.

### Optional, only if I ask

- **Theme.** `--accent` and `--ink` are declared once per mode near the top of the
  `<style>` block, under `.app[data-theme="dark"]` and `.app[data-theme="light"]`.
  Every other colour — the donut ramp, map shading, trace tones — derives from those
  two, so change nothing else.
- **Copy.** The one-line commentary in each scene is generated from my data in the
  scene definitions. Edit the strings if you must, but keep any figure inside them
  bound to the computed variable rather than typing the number in.
