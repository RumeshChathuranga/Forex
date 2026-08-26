# Advanced RC Indicator v3.4 — ICT Sniper

[![Pine Script](https://img.shields.io/badge/Pine%20Script-v6-2962FF)](https://www.tradingview.com/pine-script-docs/)
[![License: MPL 2.0](https://img.shields.io/badge/License-MPL%202.0-brightgreen)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-TradingView-131722)](https://www.tradingview.com/)

A non-repainting ICT / Smart Money Concepts engine for TradingView. It maps market structure,
price-delivery arrays, liquidity and the algorithmic clock, then fires only when a named model has
actually formed — and tells you which one.

---

## What it is

Most SMC indicators draw boxes and leave the reading to you. This one does the reading: it runs a
top-down cascade of hard gates, scores what survives against a weighted stack, and emits a signal
carrying **the model's name, its invalidation, and the liquidity it is targeting**.

```
ICT 2022 Model | Unicorn fresh + OTE in Discount | MACRO 09:50 NY | SMT confirmed | DOL: PDH
```

**Who it's for.** Traders already working in the ICT / SMC framework who want the bookkeeping
automated. It assumes you know what an order block and a liquidity sweep are — if you don't, start
with the [Glossary](docs/GLOSSARY.md), which defines every term the engine uses.

**Who it isn't for.** Anyone looking for a beginner's buy/sell arrow indicator. It fires rarely by
design, and its ~168 functional inputs (plus 90 colour controls) exist because
something needed tuning.


---

## Features

| Layer | What it does |
|---|---|
| **Market structure** | True BOS / CHoCH / MSS from `ta.pivothigh` / `ta.pivotlow` at two resolutions, with displacement detection |
| **Dealing range** | Premium / discount / equilibrium, OTE band (0.62 / 0.705 / 0.79) and standard-deviation objectives |
| **PD arrays** | FVG, IFVG, BPR, Unicorn, order blocks, breakers, mitigation, propulsion and rejection blocks, volume imbalances, NDOG / NWOG — one shared model with freshness, touch count and a quality tier |
| **Liquidity** | Real purge detection (trade beyond, close back inside), EQH / EQL pools, PDH / PDL, PWH / PWL, PMH / PML, Asian range |
| **Draw on Liquidity** | Ranks every unswept pool by magnetism and names the one price is being delivered towards |
| **Time** | DST-correct killzones, 20-minute ICT macros, Silver Bullet windows, True Day Open, Judas swing, Power of 3 |
| **Correlation** | SMT divergence against an auto-selected partner (XAU↔DXY, EURUSD↔GBPUSD, NQ↔SPX, BTC↔ETH) |
| **Signal engine** | Nine-gate cascade → weighted stack score → named model, invalidation-based stop, SD/DOL targets |

Every threshold is expressed as a **multiple of ATR**, never in points or pips, so the same defaults
behave identically on gold, FX, indices and crypto. See
[The ATR principle](docs/SETTINGS.md#the-atr-principle).

---

## Install

1. Open TradingView → **Pine Editor** (bottom panel)
2. Paste the contents of [`indicator/advanced-rc-ict-sniper.pine`](indicator/advanced-rc-ict-sniper.pine)
3. **Save**, then **Add to chart**

Requires a chart timeframe of 15 minutes or below for the ICT macro windows to resolve; everything
else works on any timeframe.

---

## Quick start

Four settings matter on day one. Leave the rest alone until you have read
[SETTINGS.md](docs/SETTINGS.md). For a per-instrument starting point, see
[PRESETS.md](docs/PRESETS.md).

| Setting | Group | Start at | Why |
|---|---|---|---|
| **A+ Score Threshold** | Sniper Engine | `45` | The single dial controlling signal frequency. Tune it from the dashboard — see below. |
| **Sniper Hard Gates** | Sniper Hard Gates | only *Require Valid Structure State* armed | Arming more tightens the engine. Arming all of them will silence it — see the [deadlock warning](docs/TUNING.md#the-gate-deadlock-trap). |
| **Higher Timeframe** | top of the list | one step above your chart | Must actually be *higher* than the chart, or the intermediate-array gate can never pass. |
| **Redraw Every Tick** | Display Settings | `off` | Off rebuilds the chart twice a bar instead of hundreds of times. Leave it off unless you need intrabar zones. |

---

## Reading the chart

**Only two things are signals.** Everything else is a map annotation.

| Marker | Meaning |
|---|---|
| ▲ / ▼ **triangle on the price scale** | **A signal.** Buy below the candle, sell above it. |
| **`▲ BUY 60`** flag (2 lines) | **A signal.** Score and model name; hover for the full narrative, R and stop. |
| `watch 47` | Every gate passed, the score fell short. The number *is* that setup's score. |
| `OB ▼ ★★★★ ✦fresh \| i-CHoCH` | A zone. **The ▲/▼ is the array's direction, not an entry.** Stars are the quality tier, `✦fresh` means untouched. |
| `IFVG ▼ ★★★ 2t \| inv 5` | An inverted gap; `2t` = touched twice. |
| `▲ purge` / `▼ PURGE ✕` | Liquidity taken. Uppercase = major swing level, `✕` = reclaimed. |
| `CISD ▲ ✦ ⓟ` | Change in state of delivery. `✦` displaced, `ⓟ` post-purge. |
| `EQH ×2` / `EQL ×3` | An equal-highs / equal-lows pool; `×N` = how many pivots built it. |
| `◀ DOL PWH 4632.07` | Where the engine believes price is being delivered to. |
| `SMT ▲` / `SMT ▼` | Divergence against the correlated instrument. |
| `Judas ▼`, `Turtle Soup ▲`, `QM ▲ QML` | Named patterns. |
| `TDO ▲`, `OTE .705`, `Asian Range` | Reference levels. |
| Background shading | Killzone; brighter gold = an ICT macro window. |

---

## The dashboard

Five rows, top right. The last one is the important one.

| Row | Shows |
|---|---|
| **Header** | Symbol, chart structure, higher-timeframe bias |
| **Killzone** | The most precise live window — macro beats Silver Bullet beats killzone |
| **Range** | Premium (sell side) or discount (buy side) |
| **Draw on Liq** | The magnetic target, its price, and distance in ATR |
| **Signal** | The engine's verdict — **and its tally** |

The Signal row reads one of:

```
▲ BUY  ICT 2022 Model  74  ·  3.1R          ← fired on this bar
▲ watch Unicorn  68/52  ·  158 fired / 731 setups · best 78 vs 52
blocked: ▲Structure ▼No tap  ·  last ▼132b  ·  158 fired / 731 setups · best 78 vs 52
```

**`best 78 vs 52`** is your tuning instrument: the highest score the engine has ever reached, next to
the threshold it must clear. If `best` is below your threshold, nothing can fire. Set the threshold
just under it. Full recipes in [TUNING.md](docs/TUNING.md).

---

## Alerts

One consolidated alert fires on every sniper signal, carrying a JSON payload suitable for a webhook
or a trading bot:

```json
{
  "symbol": "XAUUSD",
  "timeframe": "5",
  "model": "ICT 2022 Model",
  "direction": "long",
  "entry": 4632.07,
  "sl": 4628.11,
  "tp1": 4636.03,
  "tp2": 4639.99,
  "tp3": 4651.18,
  "rr": 3.12,
  "score": 74,
  "session": "MACRO 09:50 NY",
  "dol": "PDH",
  "narrative": "Unicorn fresh + OTE in Discount | MACRO 09:50 NY | SMT confirmed | DOL: PDH"
}
```

Individual alerts for CISD, purges, Quasimodo, Unicorn, Turtle Soup, Judas and SMT are also
available. Create them with **Alert → Condition → Advanced RC Indicator**.

---

## Non-repainting

Every detector is gated on `barstate.isconfirmed`, so nothing is evaluated on a forming candle.
Signals appear at a bar's close and never vanish afterwards.

- Pivots come from `ta.pivothigh` / `ta.pivotlow`, which confirm only after `length` bars have closed
  on both sides — inherently non-repainting.
- Previous day / week / month levels use `lookahead_on` paired with a `[1]` offset. This is the
  documented safe idiom: the value read belongs to a period that has already completed, so there is
  nothing to look ahead into, and it is available from the first bar of the new period rather than
  lagging one period behind.
- Turning off **Non-Repainting Mode** enables intrabar evaluation. Signals will then move while a
  candle is forming. This is a preview convenience, not a trading mode.

---

## Documentation

| Document | Contents |
|---|---|
| **[Presets](docs/PRESETS.md)** | Starting configurations for XAUUSD 5m, BTC / AUDUSD / USDJPY 15m, with reasoning |
| **[Glossary](docs/GLOSSARY.md)** | Every ICT term the engine implements — definition, how it is detected, what you see |
| **[Settings](docs/SETTINGS.md)** | All ~170 inputs by group, with defaults, ranges and when to change them |
| **[Tuning](docs/TUNING.md)** | Symptom → cause → fix, timeframe presets, instrument notes |

---

## Performance

The script respects TradingView's hard limits of 500 boxes, 500 lines and 500 labels, and stays
inside them at every slider's maximum. Drawings rebuild twice per bar rather than on every tick, the
liquidity registry is cached once per bar, and the signal cascade short-circuits before its only
expensive step. If your chart still feels heavy, see
[Chart is slow](docs/TUNING.md#chart-is-slow).

---

## Disclaimer

This indicator is a technical analysis tool for research and education. It is **not** financial
advice, and it does not predict future price. Trading leveraged instruments carries substantial risk
of loss. Signal counts, scores and reward:risk figures shown on the chart are descriptive of the
setup at the moment it formed, not a claim about outcomes. Test any configuration thoroughly before
risking capital, and never trade money you cannot afford to lose.

## License

[Mozilla Public License 2.0](LICENSE) — the convention for published TradingView Pine scripts. You
may use, modify and redistribute it; modified source files must remain under the same license.
