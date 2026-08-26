# Tuning Guide

Symptom → cause → fix. Every recipe here comes from a problem this engine actually had.

**Jump to:** [No signals at all](#no-signals-at-all) · [Too few](#too-few-signals) ·
[Too many](#too-many-signals) · [Deadlock trap](#the-gate-deadlock-trap) ·
[Chart is slow](#chart-is-slow) · [Chart is cluttered](#chart-is-cluttered) ·
[Timeframe presets](#timeframe-presets) · [Instrument notes](#instrument-notes)

---

## Start with the dashboard

Before changing anything, read the **Signal** row. It is the diagnostic, and it tells you which of
the problems below you actually have.

```
blocked: ▲Structure ▼No tap  ·  last ▼132b  ·  158 fired / 731 setups · best 78 vs 52
         └─────────┬───────┘                   └──┬──┘   └────┬────┘   └────┬────┘
                   │                              │           │             │
    first gate that said no, per side    signals fired   cascade passed   ceiling reached
                                                                          vs your threshold
```

| What you see | What it means |
|---|---|
| `best 78 vs 52` | Healthy. The engine reaches 78; your bar is 52. |
| `best 48 vs 60` | **Nothing can ever fire.** The ceiling is below the threshold. |
| `0 setups` | The cascade never passed. A gate is blocking — the `blocked:` text names it. |
| `731 setups / 0 fired` | Gates fine, score too low. Threshold problem, not a gate problem. |

---

## No signals at all

**Check `setups` in the tally first.** It splits the diagnosis cleanly.

### `0 setups` — a gate is blocking

The `blocked:` text names the first gate that failed, per direction. Common causes:

| Blocker | Likely cause | Fix |
|---|---|---|
| `ITF Array` | **Higher Timeframe is not above your chart.** On a 1H chart with the default `60`, every HTF gap is tagged with the chart's own timeframe and the gate can never pass. | Set Higher Timeframe one step above your chart, or disarm the gate |
| `Killzone` | You are on a daily or higher chart. A daily bar is stamped at its session open, which matches no killzone window. | The gate stands down automatically above intraday — if you still see this, check your Reference Timezone |
| `PD Range` + `Structure` on opposite sides | **The deadlock.** See [below](#the-gate-deadlock-trap). | Disarm one of them |
| `HTF Bias` | Bias has not latched yet, or points the other way | Disarm, or shorten HTF Structure Length |
| `No tap` | Gates pass; price simply is not touching a live array right now | Nothing to fix — this is the engine waiting |

### `N setups / 0 fired` — the score cannot reach the threshold

Read `best NN vs NN`. If `best` is below your threshold, lower the threshold to just under `best`.
See [Too few signals](#too-few-signals).

---

## Too few signals

**Lower the A+ Score Threshold, using evidence rather than guesswork.**

1. Read `best NN` from the Signal row — the highest score the engine has ever reached.
2. Look at the `watch NN` markers on the chart. Each number is that setup's score, so you can see
   the whole distribution.
3. Set the threshold just below the cluster you want to catch.

| Threshold | Roughly |
|---|---|
| `best − 20` | Most valid setups |
| `best − 10` | The better half |
| `best − 5` | The top few |

A conflicted market genuinely tops out in the 50s — that is the engine reporting honestly, not a
fault. In a market where range, draw and structure all agree, scores reach the high 70s.

**Other things that suppress count:**

| Setting | Effect |
|---|---|
| Min Reward:Risk to Fire | `2.0` refuses setups whose draw is too close. Try `1.5`. |
| Max Stop Distance (x ATR) | If the setup's swing invalidation is far away, the stop widens and R:R fails. `2.0` is the cap; lowering it keeps stops tight and R:R achievable. |
| Min Bars Between Signals | `10` suppresses re-entries into the same array. |
| Max Concurrent Positions | `3` — new signals are blocked while three are open. |
| Require CE Tap for Entry | On, price must reach a gap's 50% line. Turning it off roughly doubles triggers at worse fills. |

---

## Too many signals

Raise the threshold first — it is one dial and it is reversible.

Then arm gates **one at a time**, checking the tally after each. In rough order of value:

1. **Require Liquidity Taken** — reversals that took nothing on the way are the ones that fail
2. **Require Active Killzone** — removes off-hours noise entirely
3. **Require DOL in Trade Direction** — refuses trades against the draw
4. **Reject Opposing SMT** — cheap, rarely blocks
5. **Require HTF Bias Alignment** — the strongest filter, and the most restrictive

Also consider raising **Min Impulse Leg Size (x ATR)** from `0.6` to `1.0`, which produces far fewer
but much stronger order blocks.

---

## The gate deadlock trap

⚠️ **Do not arm both *Require Correct Half of Dealing Range* and *Require Valid Structure State*
without understanding this.**

They are anti-correlated in any trending market:

- In a **downtrend**, only shorts satisfy the structure gate — but price spends most of a downtrend
  in **discount**, and the PD gate refuses shorts there.
- In an **uptrend**, the mirror: only longs satisfy structure, but price sits in **premium**, where
  the PD gate refuses longs.

Between them they reject both directions on nearly every bar. The dashboard makes it obvious —
`blocked: ▲Structure ▼PD Range` is the signature.

More generally: nine gates that must all hold on the same bar multiply out to roughly **one
qualifying bar in several thousand**. That is not strict, it is off. Premium/discount is scored by
the stack by default precisely so it can be outvoted rather than holding a veto.

---

## Chart is slow

In order of impact:

1. **Leave `Redraw Every Tick` off.** On, the entire chart — several hundred boxes, lines and labels
   — is deleted and recreated on every tick. Off rebuilds twice per bar.
2. **Lower `Max Tracked FVGs (per side)`** from `120`. This is the largest per-bar cost: every gap in
   the set is re-examined each bar and scanned again by the entry trigger. `60–80` is comfortable.
3. **Lower `Max Zones Drawn`** from `18`. Working memory is six times whatever this is set to.
4. **Turn off layers you do not read** — Volume Imbalances (already off), Rejection Blocks, Internal
   Structure, FVG Score Labels, Opening Gaps.
5. **Shorten ages** — Maximum FVG Age, Zone Max Age, Untouched Level Max Age.

Setting SMT to `Off` does **not** save the data request; the feed still loads.

---

## Chart is cluttered

The map is dense by design. Switch off, in this order, until it reads:

| Turn off | Removes |
|---|---|
| Show Score Labels on FVGs | The `72 [A]` tags on every gap |
| Show FVG Consequent Encroachment | The dotted 50% line through each gap |
| Show Internal BOS / CHoCH | The faster structure breaks |
| Show Mitigation Blocks | The lowest-tier zones |
| Rejection Blocks | Wick-based zones |
| Opening Gaps (NDOG / NWOG) | The reference bands |
| Show Watch Markers | The `watch NN` near-miss tags |
| Shade Premium / Discount Zones | The two big coloured halves |

Remember that **only the price-scale triangle and the `▲ BUY` flag are signals**. Every ▲/▼ on a box
is that array's direction. If the map still competes with the signals, thin the map — not the
signals.

---

## Timeframe presets

Starting points, not gospel. Tune the threshold from `best` in every case.

### Scalping — 1m to 5m

| Setting | Value |
|---|---|
| Swing Pivot Length | `15–20` |
| Internal Pivot Length | `5` |
| Higher Timeframe | `60` |
| HTF Bias Timeframe | `240` |
| A+ Score Threshold | `50–55` |
| Min Reward:Risk | `1.5–2.0` |

Macros resolve here — leave them on, and let the timing weight do its work.

### Intraday — 15m to 1h

| Setting | Value |
|---|---|
| Swing Pivot Length | `20` |
| Internal Pivot Length | `5–8` |
| Higher Timeframe | `240` (on 15m–1h) |
| HTF Bias Timeframe | `D` |
| A+ Score Threshold | `55–65` |
| Min Impulse Leg Size | `0.8` |

Macros stop resolving above 15m; the timing weight falls back to killzone level.

### Swing — 4h to Daily

| Setting | Value |
|---|---|
| Swing Pivot Length | `10–15` |
| Internal Pivot Length | `4–5` |
| Higher Timeframe | `D` or `W` |
| HTF Bias Timeframe | `W` |
| A+ Score Threshold | `55–65` |
| Require Active Killzone | stands down automatically |
| PMH / PML | turn **on** — monthly levels matter at this range |

Fewer bars means fewer pivots, so shorten the pivot lengths or structure will barely confirm.

---

## Instrument notes

### SMT pairing

Auto-selected by asset class. Set **Correlated Instrument** to `Manual` to override.

| Your chart | Partner | Correlation |
|---|---|---|
| XAUUSD / Gold | `TVC:DXY` | **Inverse** |
| XAGUSD / Silver | `TVC:GOLD` | Positive |
| EURUSD ↔ GBPUSD | each other | Positive |
| USDJPY, USDCAD | `USDCHF` | Positive |
| AUDUSD ↔ NZDUSD | each other | Positive |
| NQ / NAS100 / NDX | `SPX` | Positive |
| ES / SPX / US500 | `NDX` | Positive |
| BTC ↔ ETH | each other | Positive |

If no partner resolves, SMT reports itself unavailable and **leaves the score scale entirely** rather
than penalising the setup.

### Session-dependent instruments

The killzone map assumes a 24-hour instrument. On something that trades a single session — a cash
index, a stock — most killzones will never be live. Either widen the killzone strings to your
instrument's hours, or leave the killzone gate disarmed and let the timing weight handle it.

### Low-volume instruments

The **Volume** weight (`8`) uses volume against its 20-period average. On instruments where your feed
reports no volume, that factor contributes a flat mid-score and is effectively neutral. Nothing
breaks; you simply lose 8 points of discrimination.
