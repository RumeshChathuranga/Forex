# Settings Reference

Every input, by group, with its default, range and when you would change it.

**Jump to:** [The ATR principle](#the-atr-principle) · [Signal controls](#1-signal-controls) ·
[Calibration](#2-universal-calibration) · [Structure & arrays](#3-structure-and-arrays) ·
[Time & liquidity](#4-time-liquidity-and-correlation) · [Risk](#5-risk-management) ·
[Counts and caps](#counts-and-caps) · [Display](#6-display-alerts-and-colours)

---

## The ATR principle

**Read this before touching any slider.**

There are no point, pip or dollar settings anywhere in this indicator. Every size, distance and
tolerance is expressed as a **multiple of a 14-period ATR** — the volatility unit the whole engine is
calibrated against.

This is why the same defaults work unchanged across instruments:

| Instrument | Price | ATR(14) on 5m | `Min FVG Size = 0.10` means |
|---|---|---|---|
| XAUUSD | 4,632 | ~4.0 | 0.40 points |
| EURUSD | 1.0850 | ~0.0004 | 0.4 pips |
| NAS100 | 20,400 | ~18 | 1.8 points |
| BTCUSD | 92,000 | ~140 | 14 dollars |

A setting of `0.10` is **one tenth of current ATR**, not ten points. It also means every threshold
adapts automatically as volatility changes — a gap that qualifies in a quiet Asian session would be
noise during NFP, and the engine knows that without being told.

The only settings expressed in absolute units are **bar counts** (lookbacks, ages, windows) and
**percentages** (risk, trailing triggers), both of which are instrument-independent by nature.

---

## 1. Signal controls

These decide what fires. Start here.

### Sniper Engine (A+ Setups)

| Input | Default | Range | What it does |
|---|---|---|---|
| Enable Sniper Engine | `on` | — | The primary signal path. Off leaves the map drawn but silent. |
| **A+ Score Threshold** | `45` | 40–100 | **The single dial controlling frequency.** Setups clearing every gate but scoring below this become watch markers instead of signals. |
| Show Watch Markers | `on` | — | Tags every near-miss with its score. Keep these on — they are how you calibrate the threshold. |
| Min Bars Between Signals | `10` | 0–200 | Stops one setup being re-signalled on every bar price spends inside the same array. |
| Max Stop Distance (x ATR) | `2.0` | 0.5–10 | How far the stop may widen past the array edge to reach the setup's swing invalidation. Beyond this the array edge stands alone — a stop that far out is a different trade at a different size. |
| Min Reward:Risk to Fire | `2.0` | 0.5–10 | Measured from the invalidation stop to the draw on liquidity. A setup that cannot pay this is refused however well it scores. |
| Also Show Legacy FVG Confluence Signals | `off` | — | The simpler FVG-tap signal with its own confluence score. Fires far more often and on a much lower bar. |

**Tuning the threshold.** Do not guess. The dashboard Signal row shows `best NN vs NN` — the highest
score the engine has ever reached against the bar it must clear. Set the threshold just under `best`.
The number on each `watch NN` marker is that setup's score, so you can also read the distribution
straight off the chart. See [TUNING.md](TUNING.md#too-few-signals).

### Sniper Hard Gates

Each gate is a **veto**: fail it and the setup is not scored at all. Only *Require Valid Structure
State* is armed by default.

| Input | Default | What it requires |
|---|---|---|
| Require HTF Bias Alignment | `off` | Higher-timeframe bias points the trade's way |
| HTF Bias Timeframe | `240` | Which timeframe that bias is read from |
| HTF Structure Length (bars) | `20` (5–100) | A close beyond the extreme of this many HTF bars is a break of structure up there; the bias latches until the opposing extreme goes |
| Require Intermediate PD Array | `off` | A live unmitigated gap on the MTF higher timeframe, same direction. **Stands down automatically if the Higher Timeframe is not actually above the chart.** |
| Require Correct Half of Dealing Range | `off` | Buy only in discount, sell only in premium |
| Require Active Killzone | `off` | Price is inside a killzone. **Stands down above intraday** — a daily bar is stamped at its session open and matches no window. |
| **Require Valid Structure State** | **`on`** | Post-CISD, post-MSS/CHoCH, or a BOS continuation in the trade's direction |
| Structure Valid For (bars) | `15` (1–100) | How recently that shift must have happened |
| Require Liquidity Taken | `off` | A purge, Turtle Soup raid or Judas swing in the trade's direction |
| Liquidity Taken Within (bars) | `20` (1–100) | How recently |
| Reject Opposing SMT | `off` | No SMT divergence pointing against the trade |
| Require Draw on Liquidity in Trade Direction | `off` | The draw points the trade's way |
| Min DOL Room (x ATR) | `2.0` (0.5–20) | And has at least this much room to be worth taking |

> ⚠️ **Do not arm every gate.** Nine conditions that must all hold on the same bar multiply out to
> roughly one qualifying bar in several thousand. Worse, *Require Correct Half of Range* and
> *Require Valid Structure State* are anti-correlated in a trend and will deadlock each other. See
> [the gate deadlock trap](TUNING.md#the-gate-deadlock-trap).

The disarmed gates are **not discarded** — HTF bias, killzone timing, liquidity, SMT, DOL room and
premium/discount are all scored by the stack below. They still decide whether a setup is good enough
to fire; they simply no longer each hold a veto.

### Sniper Stack Weights

Applied only to setups that already cleared every armed gate. These decide *how good* a valid setup
is, never whether it is valid. All range 0–50.

| Input | Default | Rewards |
|---|---|---|
| PD Array Quality | `20` | Unicorn > BPR / Breaker > OB / IFVG > Mitigation / Rejection > bare gap |
| Premium / Discount Half | `12` | Right half of the dealing range; equilibrium scores half |
| Timing (Macro / Silver Bullet) | `12` | Macro > Silver Bullet > killzone > off-hours |
| CISD + MSS Agreement | `12` | The scout fired *and* the confirmation arrived behind it |
| OTE Overlap | `10` | Entry sits inside the 0.62–0.79 band |
| Displacement Strength | `10` | Recent displacement in the trade's direction |
| SMT Confirmation | `10` | Partner confirms; "no divergence" scores three-quarters |
| DOL Room & Alignment | `10` | Draw points the trade's way, with room |
| Zone Freshness | `8` | Untouched array |
| Volume | `8` | Volume above its 20-period average |

The score is `sum of applicable weights earned ÷ sum of applicable weights × 100`. A factor that
**cannot apply** — the OTE band when the leg points the other way, SMT when no partner resolves —
leaves the scale entirely rather than scoring zero against the setup.

---

## 2. Universal Calibration

The ATR multipliers everything else is measured in.

| Input | Default | Range | What it does |
|---|---|---|---|
| ATR Period | `14` | 5–50 | The volatility unit itself. Lower reacts faster; higher is steadier. |
| Min FVG Size (x ATR) | `0.10` | 0.01–2.0 | Gaps smaller than this fraction of ATR are noise. Raise to `0.15–0.25` on noisy instruments. |
| Zone Tap Tolerance (x ATR) | `0.15` | 0.0–1.0 | How far outside an array price may trade and still count as a valid tap. |
| FVG Mitigation Depth | `0.50` | 0.0–1.0 | How much of a gap must fill to retire it. `0.5` = consequent encroachment, the ICT standard. `1.0` = full fill. |
| Maximum FVG Age (bars) | `150` | ≥1 | Gaps older than this are dropped. |
| **Max Tracked FVGs (per side)** | `120` | 20–300 | The working set walked every bar. See [Counts and caps](#counts-and-caps). |
| Show Mitigated FVGs | `off` | — | Keeps filled gaps visible. |
| Extend FVG Zones | `on` | — | Stretches gap boxes to the current bar. |
| Multi-Timeframe Analysis | `on` | — | Detects gaps on the higher timeframe too. |
| **Higher Timeframe** | `60` | 60/240/D/W | **Must be above your chart timeframe**, or HTF gaps are tagged with the chart's own timeframe and the intermediate-array gate can never pass. |
| Show Entry Signals | `on` | — | Master switch for signal drawing. |
| Entry Strategy | `Sharp Turn + Confirmation` | — | Legacy path only: whether a tap alone triggers, or a tap plus a confirming close. |
| Only Show Trend-Aligned Signals | `on` | — | Legacy path only. |
| **Non-Repainting Mode** | `on` | — | Evaluate only on confirmed closes. Turning it off lets everything move intrabar — a preview convenience, not a trading mode. |

---

## 3. Structure and arrays

### Market Structure (ICT)

| Input | Default | Range | Notes |
|---|---|---|---|
| Swing Pivot Length | `20` | 3–50 | Drives the primary trend and dealing range. Higher = fewer, more significant swings, but a longer confirmation lag. |
| Internal Pivot Length | `5` | 2–20 | Faster shifts for entry timing. |
| Show Swing Pivots (H/L) | `on` | — | |
| Show Swing BOS / CHoCH / MSS | `on` | — | |
| Show Internal BOS / CHoCH | `off` | — | Busy — turn on only while studying structure. |
| Displacement Threshold (x ATR) | `1.5` | 0.3–5.0 | Minimum body for a move to count as displacement. |
| Displacement Lookback (bars) | `3` | 1–10 | How recently displacement must have occurred to promote a CHoCH to an MSS. |
| Max Structure Drawings | `30` | 5–120 | Caps structure lines and labels. |

### Dealing Range · OTE

| Input | Default | Notes |
|---|---|---|
| Show Dealing Range | `on` | The high/low/equilibrium frame |
| Shade Premium / Discount Zones | `on` | The coloured halves |
| Show OTE Band | `on` | 0.62 / 0.705 / 0.79 retracement |
| Show Standard Deviation Projections | `on` | −0.5 / −1 / −2 standard deviations |
| Use SD Projections as TP2 / TP3 | `on` | Replaces R-multiples with the leg's own measured objectives |
| Min Leg Size (x ATR) | `1.0` (0.2–10) | Legs smaller than this produce no OTE band |

### Zones (OB / Breaker / Mitigation)

| Input | Default | Range | Notes |
|---|---|---|---|
| Show Zones | `on` | — | |
| Use Wick Boundaries | `off` | — | Off = body-based, which is the edge that actually holds |
| Show Mean Threshold (50%) | `on` | — | The consequent encroachment line on every zone |
| Also Build Zones on Internal Breaks | `on` | — | Off = only major swing breaks produce zones. Very few, very high quality. |
| Origin Candle Search (bars) | `12` | 3–50 | How far back to look for the last opposite-colour candle |
| **Min Impulse Leg Size (x ATR)** | `0.6` | 0.1–5.0 | What separates a real order block from any two-candle pair. Raise for fewer, stronger blocks. |
| Liquidity Check Window (bars) | `10` | 3–50 | A leg taking this window's extreme is an OB; one that reversed without it is a Mitigation block |
| Retire Zone After N Touches | `3` | 1–10 | Fresh (0 touches) is highest quality |
| Zone Max Age (bars) | `200` | 10–1000 | |
| Show Breaker Blocks | `on` | — | |
| Show Mitigation Blocks | `on` | — | |
| Breaker Structure Window (bars) | `10` | 1–50 | A violated zone only becomes a breaker if structure confirmed the new direction this recently |
| **Max Zones Drawn** | `18` | 4–60 | See [Counts and caps](#counts-and-caps) |

### PD Arrays

| Input | Default | Notes |
|---|---|---|
| Inversion FVGs (IFVG) | `on` | |
| IFVG Close-Through Depth (x ATR) | `0.05` (0–1) | How far past the far edge the body must close to count as inverted rather than merely filled |
| Balanced Price Ranges (BPR) | `on` | |
| BPR Pairing Window (bars) | `30` (3–200) | How far apart the two gaps may form |
| Min BPR Overlap (x ATR) | `0.04` (0–1) | |
| Unicorn Model | `on` | Breaker + FVG overlap — the top tier |
| Unicorn Pairing Window (bars) | `30` (3–200) | |
| Propulsion Blocks | `on` | |
| Rejection Blocks | `on` | |
| Rejection Wick / Range Ratio | `0.55` (0.3–0.95) | How much of the candle must be wick |
| Min Rejection Wick (x ATR) | `0.35` (0.05–3.0) | |
| Volume Imbalances | `off` | Numerous; price revisits them constantly |
| Min Volume Imbalance (x ATR) | `0.10` (0.01–1) | |
| Opening Gaps (NDOG / NWOG) | `on` | Reference levels, never retired by touches |
| Min Opening Gap (x ATR) | `0.02` (0–1) | |
| Show FVG Consequent Encroachment | `on` | The 50% line of every live gap |
| **Require CE Tap for Entry** | `on` | Entry triggers only once price reaches the gap's 50% line. Fewer signals, materially better fills. Pair with a Mitigation Depth of 0.5 or deeper. |

### CISD · Purges · Quasimodo

| Input | Default | Range | Notes |
|---|---|---|---|
| Show CISD Triggers | `on` | — | |
| Min Delivery Run Length | `2` | 1–10 | `1` fires on every single-candle flip; `2–3` keeps only real delivery legs |
| Run Must Own Extreme Of (bars) | `12` | 3–60 | Without this, CISD fires on every pullback inside a trend |
| Require Liquidity Purge First | `off` | — | Far stricter; closer to the 2022 Model sequence |
| CISD Max Age (bars) | `40` | 5–200 | |
| Show Liquidity Purges | `on` | — | |
| Purge Confirm Window (bars) | `3` | 1–10 | Price must close back inside within this many bars, or the level was genuinely broken |
| Min Pierce Depth (x ATR) | `0.02` | 0–1 | Ignores a one-tick graze |
| Track Internal Pivots as Liquidity | `on` | — | |
| Level Cluster Tolerance (x ATR) | `0.15` | 0–1 | Pivots within this distance merge into one pool — this is what builds EQH/EQL |
| Purge Max Age (bars) | `60` | 10–300 | |
| Untouched Level Max Age (bars) | `400` | 50–1000 | |
| Show Quasimodo Patterns | `on` | — | |
| Min Head Sweep Depth (x ATR) | `0.10` | 0–2 | How far the Head must run beyond the Left Shoulder |
| QM Max Age (bars) | `80` | 10–400 | |

---

## 4. Time, liquidity and correlation

### Sessions & Killzones

All windows are session strings evaluated in the reference timezone, so DST is handled by the
calendar.

| Input | Default | Notes |
|---|---|---|
| Show Trading Sessions / Background | `on` / `on` | |
| **Reference Timezone** | `America/New_York` | ICT's clock. The macro times were derived in it. |
| London Killzone | `0200-0500` | |
| New York AM Killzone | `0700-1000` | |
| London Close Killzone | `1000-1200` | |
| Asia Killzone | `2000-0000` | |
| Highlight ICT Macros | `on` | **Only resolves at 15m and below** |
| Highlight Silver Bullet Windows | `on` | 03:00–04:00, 10:00–11:00, 14:00–15:00 |
| True Day Open (NY Midnight) | `on` | |
| Classify Power of 3 (AMD) | `on` | |
| Judas Swing | `on` | |

### Liquidity Pools & HTF Levels

| Input | Default | Range | Notes |
|---|---|---|---|
| Equal Highs / Lows (EQH / EQL) | `on` | — | |
| **Min Pivots per Pool** | `2` | 2–8 | How many clustered pivots make a pool. Raise to `3` for only the heaviest. |
| Max Pools Drawn | `8` | 2–20 | |
| PDH / PDL (Previous Day) | `on` | — | |
| PWH / PWL (Previous Week) | `on` | — | |
| PMH / PML (Previous Month) | `off` | — | Often far off-screen intraday |
| Asian Range Box | `on` | — | |
| Turtle Soup | `on` | — | |
| Turtle Soup Lookback (bars) | `20` | 5–200 | The N-period extreme being raided |
| Turtle Soup Confirm Window (bars) | `5` | 1–20 | How long the reversal has to confirm |

### Draw on Liquidity

| Input | Default | Range | Notes |
|---|---|---|---|
| Show Draw on Liquidity | `on` | — | |
| **Min Distance to Qualify (x ATR)** | `0.75` | 0–10 | A pool closer than this is already in reach and is not an objective worth a trade |
| Max Distance Considered (x ATR) | `25` | 2–200 | Beyond this is scenery |
| Directional Bias Weight | `12` | 0–50 | How far premium/discount and trend tilt the ranking. `0` ranks purely on magnetism. |

### SMT Divergence

| Input | Default | Notes |
|---|---|---|
| Correlated Instrument | `Auto` | Auto / Manual / Off. Auto picks by asset class — see the [pairing table](GLOSSARY.md#smt-divergence). |
| Manual Symbol | `TVC:DXY` | Used when set to Manual |
| Correlation | `Auto` | Auto / Positive / Inverse. Inverse partners are normalised by negating their series. |
| SMT Pivot Length | `5` (2–30) | |
| Show SMT Labels | `on` | |

> Setting SMT to **Off** stops the engine consuming the data but does not remove the request — the
> feed still loads.

---

## 5. Risk Management

Applies to the SL/TP lines drawn with each signal.

| Input | Default | Range | Notes |
|---|---|---|---|
| Show Risk Management Levels | `on` | — | |
| Risk:Reward Ratio | `2.0` | ≥0.5 | Fallback target when SD projections do not apply |
| Stop Loss Padding (x ATR) | `0.25` | 0–2 | Buffer beyond the array edge |
| Use Trailing Stop | `off` | — | |
| Trail Trigger (%) | `50` | 10–90 | % of TP reached before trailing activates |
| Show Partial TP Levels | `off` | — | |
| Partial TP (%) | `50` | 10–90 | |
| Use Breakeven Stop | `on` | — | |
| Breakeven Trigger (%) | `30` | 10–90 | % of TP reached before SL moves to entry |
| Max Daily Loss (%) | `5.0` | 0.5–20 | |
| Volatility-Based Position Sizing | `on` | — | Scales size inversely with the ATR/average-ATR ratio |
| Time-Based Exit | `off` | — | |
| Max Trade Duration (hours) | `24` | 1–168 | |
| Max Concurrent Positions | `3` | 1–10 | New signals are suppressed while this many are open |
| **Max Tracked Trades** | `12` | 2–40 | Drawing budget — see below |
| Risk Per Trade (%) | `1.0` | 0.1–5.0 | |

**Smart ATR SL/TP** — an alternative stop model for the legacy path only. The sniper engine always
derives its stop from the setup's own invalidation.

| Input | Default | Range |
|---|---|---|
| Use Smart ATR SL/TP | `on` | — |
| ATR SL Multiplier | `1.5` | 0.5–5.0 |
| ATR TP Multiplier | `3.0` | 1.0–10.0 |

---

## Counts and caps

TradingView enforces hard limits of **500 boxes, 500 lines and 500 labels** per script. These
settings divide that budget, and they also bound how much work happens each bar — every one of them
is a collection the engine walks.

| Input | Default | Bounds | Raise if | Lower if |
|---|---|---|---|---|
| **Max Tracked FVGs (per side)** | `120` | The gap working set — re-examined every bar and scanned again by the entry trigger. **The single biggest per-bar cost.** | Your market leaves more live gaps than this inside the max age | The chart feels heavy |
| Max Zones Drawn | `18` | Zone boxes on screen. Working memory is six times this. | You want the full array map visible | The chart is cluttered or slow |
| Max Pools Drawn | `8` | EQH/EQL pool lines | You trade pools heavily | — |
| Max Structure Drawings | `30` | BOS/CHoCH/MSS lines and pivot tags | Studying structure history | — |
| Max Tracked Trades | `12` | Concurrent SL/TP line sets — **five lines each** | You want a longer trade history on screen | You run many signals |
| Maximum FVG Age (bars) | `150` | How long a gap survives | — | Old gaps clutter |
| Zone Max Age (bars) | `200` | How long a zone survives | — | — |
| Untouched Level Max Age | `400` | How long a resting level stays in the registry | — | — |

At default settings the tracked label total is about 300 of the 500 available. Pushing *Max
Structure Drawings* and *Max Zones Drawn* to their maximums at the same time can exceed it; the
oldest drawings are simply dropped, which is the intended degradation rather than an error.

The budget is allocated with **signal output taking priority** — the engine retains 60 signal flags
and 15 watch markers, and FVG score tags are the first thing sacrificed when labels get tight.

---

## 6. Display, alerts and colours

| Input | Default | Notes |
|---|---|---|
| Show Status Table | `on` | The five-row dashboard |
| **Redraw Every Tick** | `off` | Off rebuilds drawings twice per bar — once on open, once on close. On rebuilds on every tick, which is the dominant cost in real time. Only needed with Non-Repainting Mode off. |
| Enable Alerts | `on` | Master switch |

### Confluence Scoring

Used by the **legacy** FVG signal path only. If *Also Show Legacy FVG Signals* is off, none of this
affects anything.

| Input | Default | Range |
|---|---|---|
| Minimum Confluence Score | `40` | 0–100 |
| Show Score Labels on FVGs | `on` | — |
| Color-Code FVGs by Score | `on` | — |
| Weight: Trend / RSI / Volume / Variant / Session / Volatility | `25` / `15` / `15` / `20` / `15` / `10` | 0–100 each |

### Colours

Every drawn object has a colour input, grouped under **Colors** — FVGs and their variants, score
tiers, risk lines, structure events, sessions and killzones, each zone kind, each PD array, the
liquidity pools, HTF levels, DOL, named patterns and the sniper signals. All are cosmetic; none
affect detection.
