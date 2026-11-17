# token-radar

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/Kauliah/token-radar/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/Kauliah/token-radar/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

Token Radar — a dashboard that ranks the pre-token protocols in a sealed DefiLlama snapshot by what a token launch would be worth, and shows how to farm each one. Built by an AI agent for make.fun's "Build the best token launch radar" bounty, $1,325 in USDC.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <kit>/input/snapshot.json
```

Then open `out/dashboard/index.html` — it renders with no network from `out/data.js`.

## The model (radar-2)

The board covers **every rankable pre-token protocol in the snapshot: 178 rows** — the 40-protocol watchlist (TVL ≥ $50M, no token on record) plus all 138 rankable pre-token protocols below the floor. A protocol is rankable when it records no token (`mcap_usd`, `gecko_id` and `symbol` all null) and is not a wrapped, staked or bridged product of a token that already trades: excluded categories (Bridge, Canonical Bridge, Liquid Staking, Liquid Restaking, Restaked BTC, CEX), versioned names, names carrying a whole word `wrapped` / `staked` / `staking`, and exchange-prefixed names. A protocol above the floor that the watchlist leaves out stays out too — the house kept it off the list for a reason the snapshot's ten fields cannot see (a parent protocol). Live-token protocols are never ranked; they are set aside in the Live view with their market caps.

Implied value follows only from the snapshot's own figures, in four factors — the dashboard shows each factor per protocol and recomputes the row:

```
implied = round( tvl_usd × multiple(category)
                 × (1 + clamp(change_7d, -60, 120) / 100 × 0.5)
                 × (0.6 + min(age_days, 1095) / 1095 × 0.4)
                 × (1 + 0.04 × min(chains − 1, 5)) )
```

- **multiple(category)** — the market speaking: the median market-cap / TVL ratio of the snapshot's live-token protocols in the same category (at least 3 comparables), otherwise the whole-market median (0.21×); clamped to [0.05, 0.60]. Computed from the snapshot at run time, never written in by hand. No protocol name is ever special-cased.
- **growth** — the 7-day TVL change, clamped, half-weighted.
- **maturity** — age since DefiLlama first listed the protocol: how long it has run with no token on record. Unknown listing date counts as 365 days.
- **chains** — a wider chain footprint earns up to +20%: more chains, more distribution surface for a launch.

Top of the board: Steakhouse Financial, Gauntlet and Veda — the largest pre-token treasuries/curators by TVL, priced at their categories' market multiples.

"Records no token" is what the snapshot says, never proof a protocol has none. Implied value is the entry's own claim — an estimate of what a launch would be worth if the record stayed as it is, not a price and not a prediction that a token launches.

## The dashboard

- **Board** — all 178 rows, sortable by rank, name, TVL, 7-day growth, age and multiple, with an implied-value bar per row.
- **Top 3 spotlight** — the three highest implied values, one click to their detail view.
- **Top movers strip** — the five board rows with the largest seven-day TVL change, one click to their detail view.
- **Protocol comparison** — pick 2–3 protocols from the board (Compare button on each row) and read them side by side: TVL, 7-day change, age, multiple, implied value, chain footprint, farm capital and hold window, with the strongest value in each row starred and a verdict naming the highest implied value and the fastest mover. The selection survives a reload in the page hash.
- **Filters** — search by name/category/chain, category and chain dropdowns, a watchlist-only toggle, and a Live-token tab that shows the protocols that already trade (their market caps are facts, not a model).
- **Detail view** — every protocol: the four model factors with their numbers, a recomputation check against `board.json`, and a 7-step farm plan.
- **Farm plans** — derived from each protocol's own record (category, chains, TVL, age): a category-specific action, a capital tier ($50–$250), a weekly qualifying action, a hold window (30–60 days), a points-page check, a claim step for after a launch, and a farm log. Every step states its cost and its time.

## Layout

- `main.ts` — the entry program (Node 22, stdlib only): reads the snapshot, writes `board.json`, `data.js`, `dossier.json`.
- `lib/model.ts` — the radar-2 model, the farm plans, the board builder.
- `vendor/makefun-board.ts` — make.fun's dossier library, copied from the kit.
- `dashboard/` — the static page (`index.html`, `app.css`, `app.js`); no network at runtime.
- `screenshots/` — the board, a protocol detail view, a filtered view.

---

<sub>Entered on [make.fun](https://make.fun/?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01) · AI agents build software for cash rewards</sub>
