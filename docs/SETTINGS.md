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
| **A+ Score Threshold** | `50` | 40–100 | **The single dial controlling frequency.** Setups clearing every gate but scoring below this become watch markers instead of signals. The scale has a floor — a setup with no supporting evidence still scores above zero — so read the `watch NN` markers on a flat stretch to find where that floor sits in your market before choosing a number. |
| Show Watch Markers | `on` | — | Tags every near-miss with its score. Keep these on — they are how you calibrate the threshold. |
| Min Bars Between Signals | `6` | 0–200 | Stops one setup being re-signalled on every bar price spends inside the same array. Counted **per direction**. |
| **Opposite-Direction Cooldown (x)** | `2.0` | 0–10 | Multiplies the above to mute the *opposite* side after a fire. At `0` the two directions are independent, which lets a BUY and a SELL print on consecutive bars — what a chopping market produces and what no trader would take. |
| Max A+ Signals per Killzone | `3` | 0–20 | A budget, reset when a new killzone opens. The fourth setup inside one NY AM window is usually the market having stopped trending. `0` disables it. |
| **Entry Confirm Window (bars)** | `5` | 1–15 | How long a setup stays armed waiting for its confirmation candle. See below. |
| **Min Confirmation Body (x ATR)** | `0.25` | 0–2 | A confirmation candle has to be a real one. Without this the mean-reclaim trigger accepts any candle closing the right side of the array's 50% line, however small — roughly every other bar on a 3-minute chart. |
| Max Stop Distance (x ATR) | `1.5` | 0.5–10 | The risk ceiling, and the engine's only anti-chase guard. A confirmation is accepted however far it closes from the array provided the stop it implies fits this budget. Also caps how far a purge wick or Quasimodo head may widen the stop. |
| Min Reward:Risk to Fire | `1.8` | 0.5–10 | Measured from the invalidation stop to the objective selected below. A setup that cannot pay this is refused however well it scores. |
| **Reward:Risk Measured To** | `Nearest Pool` | — | `Nearest Pool` measures to the first unswept pool in the trade's direction — the obstacle price must clear before anything beyond it is reachable. `Draw on Liquidity` is the legacy behaviour and measures to the highest-*ranked* pool, which is optimistic whenever something is resting in between: on gold it will name a weekly high thirty points off while an equal-highs cluster sits four points above the entry, crediting a setup with 15R when its realistic objective is 1.5R. |
| Reward:Risk Target Cap (x ATR) | `10.0` | 1–50 | A backstop on the measured objective, so a daily or weekly level tens of ATR away can never hand an intraday setup a fictional reward. Caps the test only — the drawn TP3 still sits on the real level. |

### How an entry is triggered

Price reaching a level and price turning at that level are two different events, usually several
candles apart, and the engine treats them that way.

1. **Arm.** Price trades into a live array's entry band — the far edge to its 50% mean threshold, so
   the lower half of a demand zone or the upper half of a supply zone. The setup is recorded along
   with the array's quality tier and the wick that reached into it. No gate is consulted here; arming
   is only the observation that price got to the level.
2. **Confirm.** Within *Entry Confirm Window* bars, any one of four things says price has turned: a
   close that **reclaims the array's mean**, a **CISD**, a **structure break**, or a **displacement
   leg** in the trade's direction. Arming and confirming on the same candle is allowed — a wick into
   the level closing back through its mean is one candle doing both jobs.
3. **Void.** A close beyond the array's far edge retires the setup immediately: that is the level
   failing, which is the trade's real invalidation. Otherwise the window simply expires.

The stop is measured from **the deepest wick the level survived while armed**, not from the array's
edge and not from a blanket ATR multiple.

The `Entries` row on the dashboard reports this lifecycle directly — `armed N → conf N · exp N ·
void N`. A high `armed` with a near-zero `conf` means price keeps reaching your levels and never
turning at them, which is a market condition, not a filter problem. A high `void` means the arrays
themselves are failing.

**B grade.** A+ is a full cascade pass and stays exactly that rare. A setup that confirmed at a live
array with valid structure behind it and missed **exactly one** armed gate is still a trade — a smaller one
— and is marked separately so a quiet chart stays informative. B markers are small, dim and never get
a triangle; the tooltip names the gate that was missed. They are not tracked as trades and draw no
SL/TP lines.

| Input | Default | Range | What it does |
|---|---|---|---|
| Show B-Grade Setups | `on` | — | Off restores an A+-only chart. **Requires at least two armed gates to ever fire** — see below. |
| B-Grade Score Threshold | `38` | 25–100 | Scored on the identical stack as A+ — the grade reflects the missing gate, never a softer measurement. |
| B-Grade Min Reward:Risk | `1.5` | 0.5–10 | Measured the same way as the A+ figure. |

> **B reads `0` with only one gate armed, and that is correct.** B means "missed exactly
> one armed gate while structure still held". With *Require Valid Structure State* as the
> only armed gate, missing one gate *is* missing structure — and structure is required at
> every grade, because with no shift there is no reversal to trade. The two conditions are
> mutually exclusive, so B is unreachable by construction. Arm a second gate and B starts
> reporting. This is a real limit of the design, not a bug to wait out.

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
| Structure Valid For (bars) | `20` (1–100) | How recently that shift must have happened |
| Require Liquidity Taken | `off` | A purge, Turtle Soup raid or Judas swing in the trade's direction |
| Liquidity Taken Within (bars) | `20` (1–100) | How recently |
| Reject Opposing SMT | `off` | No SMT divergence pointing against the trade |
| Require Draw on Liquidity in Trade Direction | `off` | The draw points the trade's way |
| Min DOL Room (x ATR) | `2.0` (0.5–20) | And has at least this much room to be worth taking |
| **Require Directional Confluence** | **`on`** | A majority of the market's own state variables agree with the trade — see below |
| Min Agreeing Factors | `3` (1–5) | The quorum. The trade must also have more factors for it than against it. |
| **Reject Consolidation (Balance)** | **`on`** | Price is delivering rather than rotating — **in the middle of the range only** |
| Regime Lookback (bars) | `20` (5–100) | The window efficiency is measured over |
| Min Delivery Efficiency | `0.28` (0–1) | Net movement ÷ distance travelled. `0` is pure noise, `1` a straight line; below ~0.25 a market is genuinely rotating. |
| **Range Edge Band** | `0.30` (0–0.5) | The fraction of the dealing range at each end that counts as an extreme. Inside these bands the consolidation gate stands down. `0` makes it unconditional. |

#### Directional confluence — a vote, not a veto

Every other gate here is a veto, and vetoes multiply: arming enough of them to enforce a direction
deadlocks the engine, which is why all but structure ship disarmed — and why, before this gate
existed, **nothing enforced a direction at all**. The engine would take a long at the high of a leg
while its own dashboard reported HTF bias down, chart trend down and the draw above.

Five factors each return *for*, *against*, or abstain when they have no reading:

| Factor | Abstains when |
|---|---|
| HTF bias | Bias has not latched |
| Chart swing trend | Structure is neutral |
| Draw on liquidity | No qualifying pool |
| Dealing-range half | No range resolved |
| Power of 3 phase | Accumulation — the range has not resolved, so it has no opinion |

The trade needs a quorum *and* a plurality, so 3-of-5 with two opposed passes while 2-for-2-against
never does. Because no single factor can block, there is no pair that can deadlock. The bias row
prints the live counts as `vote ▲n ▼n`.

#### Delivery regime

Inside a range every array gets tapped, every tap finds a reclaim candle, and the two directions
confirm alternately — so the engine's signal count **peaks exactly where its edge is worst**. This
gate measures Kaufman's efficiency ratio directly: how much of the distance price actually travelled
went anywhere. It is direction-independent — a balancing market is a bad place to take a reversal
either way round.

**But only in the middle of the range.** Efficiency alone was the wrong test and rejected roughly
two-thirds of every bar, more than structure and confluence combined. Low efficiency *is* the
definition of a range being raided: Turtle Soup, the Judas swing and every purge-and-reverse happen
precisely because price spent the lookback rotating, which is what parks the stops at the edge worth
running. Applied without regard to where in the range price sits, the gate filtered out the core ICT
reversal model and kept only continuation entries in moves already delivered.

`Range Edge Band` fixes that. In the outer 30% at each end of the dealing range the gate stands
down — the range is being tested and the reversal is taken there. Between them it holds. Set it to
`0` to restore the unconditional behaviour.

**A deep pullback inside a trend also reads as low efficiency**, for the same arithmetic reason — the
retracement shrinks net displacement while the path keeps growing. Exempting those on a strong
directional vote was tried and **measurably lost money**: it added 9 signals, none reached TP1, all 9
hit the stop, and expectancy at 1.8R went from `+0.04R` to `−0.16R`.

The flaw is worth recording, because the idea looks right. The vote is a *state* and efficiency is a
*bar-level* measure. HTF bias, chart trend, premium/discount and the draw can all hold the same
reading for hours while price grinds sideways mid-range — which is exactly what this gate exists to
refuse. Overriding a fast measure with a slow one disables the gate almost permanently whenever the
market has a persistent lean, and *persistent lean plus mid-range chop* is the worst thing the engine
can trade. `Range Edge Band` works because it is **positional** and moves with price; a vote is not.

> ⚠️ **Do not arm every gate.** Nine vetoes that must all hold on the same bar multiply out to
> roughly one qualifying bar in several thousand. Worse, *Require Correct Half of Range* and
> *Require Valid Structure State* are anti-correlated in a trend and will deadlock each other. See
> [the gate deadlock trap](TUNING.md#the-gate-deadlock-trap). The two gates armed by default —
> confluence and regime — are exempt from this: neither is a single-factor veto.

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
| Power of 3 Phase | `8` | Manipulation-phase entries — stops harvested, reversal pending — score full. Distribution is a chase, accumulation is a guess. |

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
| FVG Mitigation Depth | `0.50` | 0.0–1.0 | How much of a gap must fill before it is marked used. `0.5` = consequent encroachment, the ICT standard. `1.0` = full fill. This affects freshness scoring and drawing only — **the entry engine keeps a gap eligible until its far edge is closed through**, because retiring an array at the depth the entry needs price to reach would destroy it with the very move that qualified it. |
| Maximum FVG Age (bars) | `150` | ≥1 | Gaps older than this are dropped. |
| **Max Tracked FVGs (per side)** | `120` | 20–300 | The working set walked every bar. See [Counts and caps](#counts-and-caps). |
| Show Mitigated FVGs | `off` | — | Keeps filled gaps visible. |
| Extend FVG Zones | `on` | — | Stretches gap boxes to the current bar. |
| Multi-Timeframe Analysis | `on` | — | Detects gaps on the higher timeframe too. |
| **Higher Timeframe** | `60` | 60/240/D/W | **Must be above your chart timeframe**, or HTF gaps are tagged with the chart's own timeframe and the intermediate-array gate can never pass. |
| **Non-Repainting Mode** | `on` | — | Evaluate only on confirmed closes. Turning it off lets everything move intrabar — a preview convenience, not a trading mode. |

---

## 3. Structure and arrays

### Market Structure (ICT)

| Input | Default | Range | Notes |
|---|---|---|---|
| Swing Pivot Length | `12` | 3–50 | Drives the primary trend and dealing range. Higher = fewer, more significant swings, but the confirmation lag is paid on **both** sides: `20` on a 5-minute chart puts major structure over three hours behind price, far too late for the structure gate's validity window. |
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
| **Dealing Range Source** | `HTF Range` | Where the two edges come from. `Chart Swings` is the legacy behaviour and is **timeframe-dependent by construction**: at the default pivot length a 3-minute chart spans 36 minutes and a 5-minute chart an hour, so the two report *opposite halves* for the same instrument at the same moment — and each then feeds a 12-point stack weight and an optional veto off that contradiction. `HTF Range` resolves the same range on every chart timeframe. `Session Range` uses the current day's high and low. |
| Dealing Range Timeframe | `60` | Used by `HTF Range` only. Should sit well above your chart: `60` for 3–5m, `240` for 15m–1h. |
| Dealing Range Length (bars) | `20` (5–100) | How many bars of that timeframe define the range. 20 bars of 1H is a little under a trading day. |
| Show OTE Band | `on` | 0.62 / 0.705 / 0.79 retracement |
| Show Standard Deviation Projections | `on` | −0.5 / −1 / −2 standard deviations |
| Use SD Projections as TP2 / TP3 | `on` | Replaces R-multiples with the leg's own measured objectives |
| Min Leg Size (x ATR) | `1.0` (0.2–10) | Legs smaller than this produce no OTE band |

The OTE band and the standard-deviation projections are measured from the **chart-local impulse
leg**, which is tracked separately from the dealing range above. The two used to share one pair of
numbers, which was only correct while the range was built from chart swings — anchoring the range to
a higher timeframe would otherwise have turned OTE into an HTF retracement and measured the
projections off a leg the chart never delivered.

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
| Show FVG Consequent Encroachment | `on` | The 50% line of every live gap. This is the level the entry engine arms on and the level its confirmation has to reclaim. |

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

This is an indicator. It cannot hold a position, size one, or enforce a daily loss cap, and it no
longer offers inputs implying otherwise. What is left is the level set drawn against a fired signal.

| Input | Default | Range | Notes |
|---|---|---|---|
| Show Risk Management Levels | `on` | — | Draws SL, TP1, TP2 and the final objective against each signal |
| Stop Loss Padding (x ATR) | `0.25` | 0–2 | Buffer beyond the setup's invalidation wick |
| **Min Stop Distance (x ATR)** | `0.75` | 0.1–3 | A floor on how close the stop may sit to the entry, whatever the setup's own invalidation says. Applied last, so it only ever widens a stop and never pulls one inside the level that would prove the trade wrong. Without it the whole stop is often barely more than the padding — a quarter of one ATR on gold, where a single 5-minute wick routinely runs half of one, so trades are stopped out by noise rather than by being wrong. A stop that tight also inflates the reward:risk it is measured against, so those setups pass a test they should fail. Raise it if the **Result** row shows a high stop-out rate; lower it only if R:R starts refusing setups you would have taken. |
| **Max Tracked Trades** | `12` | 2–40 | Drawing budget — see below |

**How the levels are derived.**

- **Stop** — the deepest wick the array survived while the setup was armed, padded. Widened to a
  model's own swing invalidation (the Turtle Soup raid extreme, the Judas level, the Quasimodo head,
  or the purge wick on a full ICT 2022 sequence) only when *that model* is what named the setup, and
  never past *Max Stop Distance*.
- **TP1 / TP2** — the current impulse leg's 0.5 and 1.0 standard-deviation projections when the leg
  runs the trade's way; 1R and 2R when it does not.
- **Final objective** — the best liquidity pool lying in the trade's own direction, whether or not
  it won the draw-on-liquidity ranking. This is also what *Min Reward:Risk to Fire* is measured to.
  When that pool sits nearer than TP1 or TP2, those are compressed inside it rather than left beyond
  a level the trade will never reach.

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
and 15 watch markers, and the HTF gap tags are the first thing sacrificed when labels get tight.

---

## 6. Display, alerts and colours

| Input | Default | Notes |
|---|---|---|
| Show Status Table | `on` | The dashboard — five rows, plus three for diagnostics and one for results |
| **Show Gate Diagnostics** | `on` | Adds three rows. **Gates** — how many bars each armed gate has rejected across the loaded history, plus which gates are armed. **Entries** — the armed-setup lifecycle (`armed → conf · exp · void`), which says what became of every level price actually reached. **Rejects** — what happened to setups that confirmed and cleared every gate and still did not fire (`score` / `RR` / `cool` / `budget`), followed after a slash by `floor`, which is *not* a rejection: it counts confirmed setups whose stop came from the noise floor rather than from their own invalidation. A low `floor` share means stops are already wide, so widening the floor further would change almost nothing — see [the recipe](TUNING.md#signals-fire-but-lose). Read these rows in order; a gate holding a five-figure count while the setup tally sits in the dozens is the thing to disarm, and no threshold tuning substitutes for that. |
| **Show Signal Performance** | `on` | Adds a **Result** row counting what fired signals actually reached: `n · TP1% · TP2% · TP3% · SL%`. First-touch, with no assumed management, so it measures the engine's edge rather than a position-sizing policy — and a bar that touches both a target and the stop is scored as the stop, which keeps the number pessimistic rather than flattering. This is the row that turns tuning from an argument into a measurement: if `SL` dominates while the setup tally is healthy, the engine is picking the wrong *direction*, and raising the threshold will not fix it. |
| **Redraw Every Tick** | `off` | Off rebuilds drawings twice per bar — once on open, once on close. On rebuilds on every tick, which is the dominant cost in real time. Only needed with Non-Repainting Mode off. |
| Enable Alerts | `on` | Master switch |

### Colours

Every drawn object has a colour input, grouped under **Colors** — FVGs and their variants, risk
lines, structure events, sessions and killzones, each zone kind, each PD array, the liquidity pools,
HTF levels, DOL, named patterns and the sniper signals. All are cosmetic; none affect detection.

FVG boxes are coloured by **variant** — Perfect, Breakaway, Rejection — and a gap the intermediate
timeframe also left gets a heavier border. There is no separate confluence score: the sniper stack
is the only scorer in the script, and a second scoring model that disagreed with it was worse than
no second model at all.
