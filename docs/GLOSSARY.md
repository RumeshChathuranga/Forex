# Glossary

Every ICT / Smart Money Concepts term this indicator implements. Each entry gives the concept, how
this engine detects it, and what you see on the chart.

Terms are grouped by subsystem. If you are new to the methodology, read
[Market structure](#market-structure) and [Price-delivery arrays](#price-delivery-arrays) first —
everything else builds on them.

---

## Market structure

### Swing vs internal pivots
Two resolutions of the same thing. **Swing pivots** (default length 20) confirm the primary trend and
anchor the dealing range. **Internal pivots** (default 5) give the faster shifts used for entry
timing. Both come from `ta.pivothigh` / `ta.pivotlow`, which only confirm once `length` bars have
closed on either side — which is what makes them non-repainting, at the cost of a `length`-bar
confirmation lag.

*Chart:* `H` and `L` tags at confirmed swing pivots.

### BOS — Break of Structure
A close beyond the prior swing extreme **in the direction of the existing trend**. Continuation, not
reversal: the trend did what it was already doing.

*Chart:* dashed line back to the broken pivot, tagged `BOS ▲` / `BOS ▼`.

### CHoCH — Change of Character
The **first** close beyond the *opposing* swing extreme. This flips the trend state. A CHoCH is the
earliest structural evidence that control has changed hands.

*Chart:* `CHoCH ▲` / `CHoCH ▼`.

### MSS — Market Structure Shift
A CHoCH **accompanied by displacement**. This is the high-quality reversal trigger: structure broke
*and* it broke with force. The engine promotes a CHoCH to an MSS when displacement occurred within
the last few bars.

*Chart:* `MSS ▲` / `MSS ▼`, in a distinct colour.

### Displacement
An energetic candle that leaves an imbalance behind it. Detected strictly: the body must span at
least `dispATRMult` × ATR **and** the candle must gap past the range two bars back
(`low > high[2]` for an up move). Both conditions — energy *and* inefficiency. This strictness is
why displacement is genuinely rare and why its absence is not heavily penalised in scoring.

---

## Ranges and retracement

### Dealing range
The band from the most recent confirmed swing low to the most recent confirmed swing high, expanding
with price while it trades beyond either edge. Everything premium/discount is measured against it.

### Premium / Discount / Equilibrium
**Equilibrium** is the 50% mark of the dealing range. Above it is **premium** (expensive — the sell
side), below it is **discount** (cheap — the buy side). The core ICT pricing rule: never buy in
premium, never sell in discount.

*Chart:* shaded upper and lower halves with an `EQ` line between them; the dashboard **Range** row.

### OTE — Optimal Trade Entry
The 0.62 / 0.705 / 0.79 retracement band of the current impulse leg. The zone where a retracement is
deep enough to offer a good price but not so deep that the leg has failed. 0.705 is the reference
line.

*Chart:* shaded band with the `OTE .705` level marked.

### Standard deviation projections
The impulse leg measured *forward* past its own extreme at 0.5, 1 and 2 standard deviations of the
leg itself. These are objectives the market defined, and they replace arbitrary R-multiples as the
engine's TP1 / TP2.

*Chart:* `-1 SD` and `-2 SD` levels.

---

## Price-delivery arrays

A **PD array** is any price band the algorithm is expected to react to. This engine models all of
them with one shared object, so every array carries a direction, a **quality tier** (1–5), a **touch
count** and a **freshness** state.

### FVG — Fair Value Gap
A three-candle imbalance where the middle candle moved so fast that candle 1's wick and candle 3's
wick never overlapped. Price tends to return and rebalance it.

- **BISI** — Buy-side Imbalance, Sell-side Inefficiency. A bullish gap.
- **SIBI** — Sell-side Imbalance, Buy-side Inefficiency. A bearish gap.

Variants the engine classifies: **Perfect** (clean), **Breakaway** (formed on an expansion out of a
range), **Rejection** (closed back into itself).

### CE — Consequent Encroachment
The **50% line of a gap**. ICT entries are taken at the CE, not at the far edge — a half-filled gap
is a better price with the same invalidation. The engine can require a CE tap before triggering.

*Chart:* dotted line through the middle of each live gap.

### IFVG — Inversion FVG
A gap that price closed clean *through* has not been filled, it has **failed**. It flips polarity and
becomes support-turned-resistance, or the reverse. One of the strongest arrays for scalping.

### BPR — Balanced Price Range
A bullish and a bearish FVG that **overlap**. Price delivered inefficiently in both directions
through the same band, which makes that overlap unusually reactive.

### Unicorn Model
A **breaker block overlapping a same-direction FVG**. The breaker says structure failed and reversed;
the gap says the reversal was delivered with displacement. The band where both statements are true is
the highest-tier array this engine recognises (tier 5).

### Order Block
The last opposite-colour candle before a displacement leg **that breaks structure** — and whose leg
**took prior liquidity** on the way. Not any down-candle-then-up-candle pair. Boundaries are
body-based by default, which is the edge that actually holds.

### Mitigation Block
Structurally identical to an order block, but its leg **reversed without taking liquidity first**.
Lower tier, because nothing was harvested to fuel the move.

### Breaker Block
An order block that price **violated**, and which structure then confirmed against. Failed supply
becomes demand, and vice versa. Only order blocks and propulsion blocks make this transition — an
IFVG or BPR that fails is finished, not reborn.

### Propulsion Block
An order block that forms **inside a live block of the same direction**: the algorithm returning to
its own level to continue delivery rather than to reverse off it.

### Rejection Block
The **wick alone** of a pivot candle that speared liquidity and was thrown straight back — body
extreme to wick extreme. The body is the range the algorithm was comfortable delivering in; the wick
is the range it was not.

### Volume Imbalance
A gap between two candle **bodies** whose ranges still overlap — price crossed the area on a wick but
never opened or closed inside it. The smallest array in the methodology. Off by default: price
returns to them constantly and they bury the chart.

### NDOG / NWOG — New Day / New Week Opening Gap
The gap between the prior session's close and the new session's open. These are **reference levels**,
not tradeable blocks: price passes back and forth through an NWOG all week and the level still draws
price. They age out rather than being retired by touches.

---

## Liquidity

### Purge vs break
A **purge** (or sweep) is price trading beyond a resting level and then **closing back inside** it —
stops were taken and price rejected. A **break** is price trading beyond and staying there. The
distinction is everything, and a wick-length heuristic cannot make it. This engine tracks every
confirmed pivot as a resting level and requires the two-part sequence.

*Chart:* `▲ purge` / `▼ PURGE` at the wick that did it; uppercase for major swing levels, `✕` when
price later reclaimed the wick and voided it.

### EQH / EQL — Equal Highs / Equal Lows
Pivots that cluster within an ATR tolerance are **one pool of resting stops**, not several separate
levels. The more pivots in the cluster, the more liquidity is parked there and the harder it pulls
price.

*Chart:* dashed line tagged `EQH ×2` / `EQL ×3` — the count is how many pivots merged.

### PDH / PDL, PWH / PWL, PMH / PML
Previous **day**, **week** and **month** high and low. The levels every desk on the planet can see,
which is exactly why price is drawn to them.

### Asian range
The high and low built during the Asia killzone, held after the session closes. London and New York
trade *against* the range Asia left behind, which is what makes it the reference for the Judas swing.

### DOL — Draw on Liquidity
**The piece most indicators miss, and this engine's headline output.**

Rather than assuming a fixed reward:risk, the engine ranks every unswept pool above and below price
by how strongly it should attract delivery, and names the winner. Magnetism is scored from five
independent terms:

| Term | Reasoning |
|---|---|
| **base** | What kind of level it is — a previous-month high outranks an internal pivot by a wide margin |
| **cluster** | How many pivots built it: an equal-highs pool is a stack of stops, not a single order |
| **age** | Liquidity that has rested untouched for longer is worth more than something printed twenty bars ago |
| **proximity** | Near levels pull harder, decaying smoothly with distance |
| **bias** | Premium/discount position and swing trend tilt the ranking — this is what stops the engine naming a target behind price |

A level price has already traded through is **spent** — the orders resting there are filled — so it
never enters the ranking. The winner becomes the target every signal is measured to, and the headline
row on the dashboard.

*Chart:* thick line tagged `◀ DOL PWH 4632.07`, with the runner-up on the opposite side drawn faintly
as the counter-objective.

---

## Delivery

### Delivery run
A series of consecutive same-direction closes — one leg of algorithmic delivery. Dojis extend the
current run rather than ending it: a candle with no body has not delivered anything.

### CISD — Change in State of Delivery
The earliest reversal trigger in the engine. Where a CHoCH waits for a swing point to break, **CISD
fires the moment a body closes through the opening price of the consecutive opposing-direction candle
run that delivered price into the extreme.**

A bullish CISD is a close above the opening of the down-close run that made the low; bearish is the
mirror. It typically fires 2–4 candles before the CHoCH, which means a materially tighter stop —
the run's extreme is only a few candles away.

On one reversal all three fire in sequence: **CISD → CHoCH → MSS**. The engine carries all three, so
it can offer a scout trigger and a confirmation trigger from a single pass.

*Chart:* line at the broken opening level, tagged `CISD ▲`; `✦` if the breaking close itself carried
displacement, `ⓟ` if liquidity was purged into the extreme first.

---

## Time

ICT's clock is **New York local time**. Every window below is evaluated in a named timezone, so the
DST shift is handled by the calendar rather than drifting the whole map by an hour twice a year.

### Killzones
The windows in which the algorithm is most active:

| Killzone | New York time |
|---|---|
| London | 02:00 – 05:00 |
| New York AM | 07:00 – 10:00 |
| London Close | 10:00 – 12:00 |
| Asia | 20:00 – 00:00 |

### ICT Macros
The **20-minute windows** either side of the hour where the algorithm reprices toward liquidity — far
more precise than a three-hour killzone block. The engine tracks eight: 02:33, 04:03, 08:50, 09:50,
10:50, 11:50, 13:10 and 15:15. A 20-minute window cannot be resolved by a bar longer than it is, so
**macros do not apply above the 15-minute chart**.

### Silver Bullet
One hour, one model: take the FVG left by the displacement inside the window and target the liquidity
it was delivered away from. Three windows — 03:00–04:00 London, **10:00–11:00 New York** (the
flagship), and 14:00–15:00 PM.

### True Day Open
**Midnight New York** — where the daily dealing range actually opens, not the exchange session open
your chart happens to use. Price delivering above it is bullish for the day, below it bearish.

*Chart:* `TDO ▲` / `TDO ▼` level.

### Judas Swing
The **first sweep of the Asian range inside a killzone, reversed on the same bar**. The move
engineered to position traders the wrong way before the real expansion. Taken once per day — only the
first one counts.

### Power of 3 (AMD) — Accumulation, Manipulation, Distribution
The day's story in three acts, anchored on the True Day Open. Price **accumulates** in a range, is
**manipulated** through one side of it to harvest the stops resting there, then **distributes** in the
other direction.

The phase tells you whether a setup is early, on time, or late: a distribution-phase entry is chasing,
an accumulation-phase entry is guessing, and the manipulation leg is where you want to be filled.

---

## Correlation

### SMT Divergence
Correlated instruments deliver the same story. When one makes a higher high and its partner fails to,
the move is not being supported and that high is a liquidity raid rather than an expansion.

The engine auto-selects a partner by asset class, and **normalises inverse pairs by negating the
partner's series**, so one set of higher-high / lower-low comparisons covers both correlation types.

| Instrument | Partner | Correlation |
|---|---|---|
| XAUUSD / Gold | DXY | Inverse |
| XAGUSD / Silver | Gold | Positive |
| EURUSD | GBPUSD | Positive |
| GBPUSD | EURUSD | Positive |
| USDJPY / USDCAD | USDCHF | Positive |
| AUDUSD | NZDUSD | Positive |
| NQ / NAS100 / NDX | SPX | Positive |
| ES / SPX / US500 | NDX | Positive |
| BTC | ETHUSD | Positive |

SMT is a **state at the most recent turn**, not a permanent flag — a pivot pair that agrees clears it.

*Chart:* `SMT ▲` / `SMT ▼` at the diverging pivot; the dashboard names which side blinked.

---

## Named models

The engine recognises the specific setup that has formed rather than emitting a bare score, because
"87" tells you nothing about what to do when it goes wrong. In priority order:

### ICT 2022 Model
The flagship, and the most mechanically defined setup in the methodology:

```
HTF bias → liquidity sweep → CISD/MSS with displacement →
FVG in discount (long) or premium (short) → entry →
stop beyond the sweep wick → target the Draw on Liquidity
```

It outranks everything else. When a Unicorn or a Silver Bullet window is part of that sequence, it
appears in the narrative rather than replacing the name.

### Silver Bullet
An FVG entry taken inside one of the three Silver Bullet hours.

### Judas Swing
Entry in the direction of the reversal that followed the day's Judas sweep.

### Turtle Soup
A raid of the N-period extreme that **closes back inside** it and is then confirmed by CISD or a
structure shift. The original Turtle system bought N-period breakouts; the soup is the trade against
them. Without the second leg it is only a long wick.

### Quasimodo (QM)
Left Shoulder → Leg → **Head sweeping beyond the Left Shoulder** → break through the Leg. The entry is
the retrace back to the shoulder price (the **QML**), the stop sits beyond the Head, and the first
objective is the broken Leg.

### Unicorn
Entry from a breaker/FVG overlap where no full 2022 sequence was present.

### CISD Scout
The early, lower-conviction tier: a CISD has fired but the structural confirmation behind it has not
arrived yet.
