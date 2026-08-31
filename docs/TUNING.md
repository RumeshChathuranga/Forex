# Tuning Guide

Symptom → cause → fix. Every recipe here comes from a problem this engine actually had.

**Jump to:** [No signals at all](#no-signals-at-all) · [Too few](#too-few-signals) ·
[Too many](#too-many-signals) · [Deadlock trap](#the-gate-deadlock-trap) ·
[Chart is slow](#chart-is-slow) · [Chart is cluttered](#chart-is-cluttered) ·
[Timeframe presets](#timeframe-presets) · [Instrument notes](#instrument-notes)

---

## Start with the dashboard

Before changing anything, read the **Signal**, **Gates**, **Entries** and **Rejects** rows. Together
they are the diagnostic, and they tell you which of the problems below you actually have.

```
blocked: ▲Structure ▼No entry ·  last ▼132b  ·  158 fired / 731 setups · best 78 vs 52
         └─────────┬────────┘                   └──┬──┘   └────┬────┘   └────┬────┘
                   │                               │           │             │
    first gate that said no, per side     signals fired   cascade passed   ceiling reached
                                                                          vs your threshold

Gates    Struct 5969  ·  Entry 402
         └────────────┬───────────┘
          every armed gate and what it has rejected, both
          directions, whole history. A gate listed here is
          a gate that is armed.

Entries  armed 1204 → conf 388 · exp 690 · void 126
         └──────────────────┬──────────────────────┘
          what became of every level price actually reached.
          armed = price got there. conf = price turned there.
          exp = it never turned. void = the array failed.

Rejects  score 6 · RR 4 · cool 5 · budget 2 / floor 3  |  B 41  |  cont 22 / rev 16
         └───────────────┬──────────────┘   └──┬──┘     └─┬─┘     └────────┬───────┘
          what happened to setups that      not a      B-grade    which family the
          confirmed, cleared every gate     reject:    setups     *fired* signals
          and still did not fire            the stop   marked     came from
                                            came from
                                            the noise
                                            floor

Result   n 38 · TP1 61% · TP2 34% · TP3 18% · SL 39%
         └──────────────────┬───────────────────────┘
          what fired signals actually reached. First-touch,
          no assumed management. A bar touching both a target
          and the stop is scored as the stop.
```

| What you see | What it means |
|---|---|
| `best 78 vs 52` | Healthy. The engine reaches 78; your bar is 52. |
| `best 48 vs 60` | **Nothing can ever fire.** The ceiling is below the threshold. |
| `144 fired / 290 setups` | **Half of everything armed became a signal.** That is not a sniper engine — see [Too many signals](#too-many-signals). |
| `Result` SL% above ~50 | The engine is firing, and the signals are wrong. Filters, not thresholds — start with confluence and the regime gate. |
| `0 setups` | The cascade never passed. A gate is blocking — the **Gates** row names which. |
| `731 setups / 0 fired` | Gates fine. The **Rejects** row says whether it is score, R:R, cooldown or stop. |
| One Gates count dwarfing the rest | That gate is the constraint. Disarm it before touching anything else. |
| `(armed: …)` listing a gate you did not mean to arm | That is your answer. Only *Require Valid Structure State* is armed by default. |
| `cont 0 / rev n` on a trending day | Every continuation is being refused. Pre-v4.2 this was structural — see [the continuation blind spot](#the-continuation-blind-spot-fixed-in-v42). |

The Signal row explains one candle; the Gates row explains the chart. When they disagree, trust
Gates — the Signal row can only ever name the *first* gate that failed, so a gate sitting downstream
of a permanently-failing one never gets to report itself.

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
| `Liquidity` + `Structure` on opposite sides | **The same deadlock, different pair.** See [below](#the-gate-deadlock-trap). | Disarm *Require Liquidity Taken* — it is off by default and scored by the stack |
| `HTF Bias` | Bias has not latched yet, or points the other way | Disarm, or shorten HTF Structure Length |
| `No entry` | Gates pass; no armed setup confirmed on this bar | Read the **Entries** row — it says whether price is failing to reach your levels or failing to turn at them |

### `N setups / 0 fired` — the score cannot reach the threshold

Read `best NN vs NN`. If `best` is below your threshold, lower the threshold to just under `best`.
See [Too few signals](#too-few-signals).

---

## Too few signals

**Read the Gates row first. Do not touch the threshold until you have.**

The Signal row names the gate blocking *this one candle*. The **Gates** row counts what every armed
gate has rejected across the whole loaded history, and that is the number that identifies the real
blocker. A gate holding a five-figure count while the setup tally sits in the dozens is the thing to
disarm — no amount of threshold tuning substitutes for that, because a vetoed setup is never scored
at all.

The Gates row also prints `(armed: …)`. If that list contains a gate you did not mean to arm, that is
your answer. Only *Require Valid Structure State* is armed by default.

Then read the **Entries** row, which says whether the entry trigger is even producing candidates —
see [the table below](#read-the-entries-row-first). Only if it is do the **Rejects** counts mean
anything.

Then read the **Rejects** row — `score / RR / cool / stop` — which says what happened to setups that
confirmed *and* cleared every gate. If `score` dominates, the threshold is genuinely the constraint
and the steps below apply. If `RR` dominates, the setups are pointing against the dominant draw. If
`stop` dominates, the invalidation is landing inside the noise floor.

**Only once `score` is the dominant reject: lower the A+ Score Threshold, using evidence.**

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
| Min Reward:Risk to Fire | `1.8`, measured to the nearest pool in the trade's own direction. Setups fighting the dominant draw legitimately fail this. Try `1.5` before blaming it. |
| Max Stop Distance (x ATR) | `1.5`. This is the risk ceiling a confirmation has to fit inside, so lowering it refuses confirmations that close far from the array. Raise toward `2.0` on a fast market. |
| Min Bars Between Signals | `6`, counted per direction. Suppresses re-entries into the same array. |
| **Entry Confirm Window (bars)** | `5`. How long a setup stays armed waiting for its confirmation. Shortening it to `1` demands the tap and the turn on one candle, which is what the engine used to require and what made it silent. |

### Read the Entries row first

`armed N → conf N · exp N · void N` tells you which half of the entry is failing, and the two halves
need opposite fixes.

| Pattern | Meaning | Do |
|---|---|---|
| `armed` low | Price is not reaching your arrays at all | Loosen *Min FVG Size*, raise *Max Zones*, or accept the market is trending away from its levels |
| `armed` high, `exp` high, `conf` low | Price reaches the levels and never turns there | Lengthen *Entry Confirm Window*; if that does not help, this is a market condition and no setting will fix it |
| `void` high | The arrays themselves keep failing | Your zones are stale or too small — raise *Min Impulse Leg Size*, lower *Retire Zone After N Touches* |
| `conf` healthy, still no signals | The entry is working; something downstream is refusing | Read the **Rejects** row |

---

## Too many signals

**First, know what the floor is.** The stack is a normalised ratio and every factor has a floor for
"the evidence is absent" — a setup with no displacement, no SMT divergence, a re-tapped bare gap and
no draw still scores well above zero. Setting the threshold below that floor filters nothing, and
the symptom is a fire rate near 50% of the setup tally.

Read the `watch NN` markers on a flat, directionless stretch of chart. That number is the floor in
your market. **The threshold has to sit well above it** — as a rule, at least halfway between the
floor and `best`.

Then, in order:

1. **Require Directional Confluence** (armed by default, `3` of 5). The single most effective
   filter, and the reason the others can stay disarmed. It is a vote across HTF bias, chart trend,
   draw on liquidity, dealing-range half and Power of 3, so it cannot deadlock the way stacked
   vetoes do. Raise to `4` to trade only when the whole picture agrees.
2. **Reject Consolidation** (armed by default, min efficiency `0.28`). Inside a range every array
   gets tapped and both directions confirm in turn, so the engine's signal count *peaks* exactly
   where its edge is worst. Raise toward `0.4` to trade only clean expansion.
3. **Max A+ Signals per Killzone** — `4`. Past a handful, setups in one NY AM window are usually the
   market having stopped trending.
4. **Opposite-Direction Cooldown** — `2`×. Stops a BUY and a SELL printing back to back.
5. **Min Confirmation Body (x ATR)** — `0.25`. Raise it if setups are confirming on doji-ish candles.
6. **Min Impulse Leg Size (x ATR)** — `0.6` → `1.0`, which produces far fewer but much stronger
   order blocks.

Only then arm the hard-veto gates, **one at a time**, checking the tally after each — and re-read
[the deadlock trap](#the-gate-deadlock-trap) first:

1. **Require Liquidity Taken** — reversals that took nothing on the way are the ones that fail
2. **Require Active Killzone** — removes off-hours noise entirely
3. **Require DOL in Trade Direction** — refuses trades against the draw
4. **Reject Opposing SMT** — cheap, rarely blocks
5. **Require HTF Bias Alignment** — the strongest filter, and the most restrictive

---

## Signals fire but lose

**This is a different problem from too many signals, and the fixes are different.** Filtering harder
does not repair a losing trade — it delivers fewer of them. Read the **Result** row and work down:

### 1. Is the stop inside the noise? (check this first)

`Result n 11 · TP1 27% · SL 73%` with `Rejects … stop 0` is the signature. A `stop` reject count
that never moves off zero means no setup has ever been refused for having an unusable stop, which
usually means the floor is too low to refuse anything.

Hover a fired signal and compare its `SL` distance to one ATR on your instrument. The **Draw on Liq**
row gives you ATR for free: it prints the distance to the draw in both price and ATR, so
`PWH 4632.07 · 10 ATR` at a price of 4590 means ATR is 4.2 points.

| Stop distance | Verdict |
|---|---|
| Under `0.5` ATR | Inside the noise. Raise **Min Stop Distance**. |
| `0.75`–`1.0` ATR | Healthy for an intraday reversal engine |
| Over `1.5` ATR | The invalidation is far away; the setup may not be worth its risk |

A stop that is too tight does double damage: it gets hit by ordinary wicks, *and* it inflates the
reward:risk the setup is measured against, so bad trades pass a test they should have failed.

**Then read `floor N` on the Rejects row before you touch Min Stop Distance again.** It counts how
many confirmed setups had their stop set by the floor rather than by their own invalidation, and it
tells the two problems apart:

| `floor` share | What it means | Do |
|---|---|---|
| High (over ~40%) | Most stops are the floor, so the arrays are too small for the instrument | Raise **Min Impulse Leg Size**; the floor is already doing its job |
| Low (under ~15%) | Stops are already wide from the setup's own invalidation | **Do not raise the floor — it would touch almost nothing.** A wide stop means the entry closed far from the level it is measured to, which is chasing. Lower **Max Stop Distance**. |

### 2. What does the arithmetic actually require?

The win rate you need is set by the R you take. At `Min Reward:Risk 1.8`:

| TP1 rate | Expectancy |
|---|---|
| 27% | **−0.24R** per trade |
| 36% | break-even |
| 45% | +0.26R |

So `TP1 36%` is the bar. Below it the engine is unprofitable however few signals it prints, and no
threshold change fixes that. Widening the stop trades R for win rate, which is the right direction
whenever the stop is inside the noise.

**Do not judge this on fewer than ~30 closed trades.** At `n = 11` the number is noise itself.

### 3. `vote ▲1 ▼4` with `blocked: ▼Balance` is not a bug

It reads like one — four of five factors back the short and the consolidation gate refuses it anyway.
Exempting that case was tried and **it lost money**: 9 extra signals, none reached TP1, all 9 stopped
out, expectancy `+0.04R → −0.16R`.

The vote is a *state* that holds for hours; efficiency is a *bar-level* measure. A market can carry a
persistent lean while grinding sideways mid-range, and that combination is the worst thing this
engine can trade. If the pairing genuinely blocks too much, move the gate's own dials — lower **Min
Delivery Efficiency** or raise **Range Edge Band**, both of which stay tied to price — and read the
Gates row first to confirm `Balance` is actually the dominant count.

### 4. Is it picking the wrong direction?

Only once the stop is sane. If `SL` still dominates while `setups` is healthy, the engine is choosing
the wrong side rather than the wrong quality — that is what the confluence vote and the regime gate
exist to fix. Check the `vote ▲n ▼n` counts on the bias row: if they are close to even, the market is
genuinely conflicted and the honest output is no trade. Raise **Min Agreeing Factors** to 4.

### 5. Is it entering late?

**Max Stop Distance (x ATR)** is how far a confirmation candle may close from the level and still be
accepted. At `1.5` on gold that is over six points of chase. Lowering it to `1.0`–`1.2` refuses
entries that closed too far from the array — fewer signals, better fills.

Raising the threshold is the *last* thing to try, not the first: a bad trade with a high score is
still a bad trade.

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

**The same trap catches *Require Liquidity Taken*.** Armed alongside the structure gate in a trend it
deadlocks the same way: the trending direction has structure but the counter-trend direction is the
one that just swept a level, so longs fail liquidity while shorts fail structure. The signature is
`blocked: ▲Liquidity ▼Structure`. Liquidity is scored by the stack by default for exactly this
reason — arming it converts a weighted factor into a veto, and vetoes multiply.

The Gates row is the general cure: any gate whose block count dwarfs the others while the setup tally
stays near zero is deadlocking, whichever pair is involved.

More generally: nine gates that must all hold on the same bar multiply out to roughly **one
qualifying bar in several thousand**. That is not strict, it is off. Premium/discount is scored by
the stack by default precisely so it can be outvoted rather than holding a veto.

### Why confluence is a vote and not a gate

The trap above is the reason the engine shipped with almost every gate disarmed — and the reason
nothing then enforced a direction at all. It would take a long at the high of a leg while its own
dashboard reported HTF bias down, chart trend down and the draw above: four readings, no two
agreeing, and no mechanism that noticed.

**Require Directional Confluence** solves that without joining the deadlock. Each of the five
factors returns *for*, *against*, or abstains when it has no reading, and the trade needs both a
quorum (`Min Agreeing Factors`, default 3) and a plurality. No single factor can block, so there is
no pair that can lock the engine — but a setup the market is genuinely split on cannot pass either.
The bias row prints the live counts as `vote ▲n ▼n`, so when the engine is quiet you can see whether
it is being blocked or whether the market simply has no opinion.

#### The continuation blind spot (fixed in v4.2)

The vote had a structural flaw worth recording, because it is invisible from the settings and it made
the gate refuse the *best* trades on any trending day.

Two of the five voters describe **where price sits**, and both read backwards on a continuation. In a
delivered downtrend price is by definition in **discount**, so the dealing-range voter returned "buy"
and opposed every short. The draw sat on the buy-side pool above, so it opposed too. HTF bias and
chart trend voted short, Power of 3 abstained in Accumulation — **2 for, 2 against, one short of the
quorum.** The engine could not take a continuation in the direction its own bias, trend and structure
all agreed on, and Confluence was consequently the largest blocker in the whole cascade.

The fix is not a lower quorum. `Min Agreeing Factors` stays at `3`; the voters were wrong, not the
bar. A continuation now measures premium/discount against the **impulse leg it is retracing** rather
than a range price already broke out of, and reads the draw from the pool in its **own** direction —
with a `Min DOL Room` requirement, so that pool is not simply a free vote. Reversals are unchanged.

Watch the `cont n / rev n` split on the **Rejects** row against the **Result** row. If continuations
are the ones losing, raise `Min Agreeing Factors` to `4` or lift **Model Quality** — do not
re-disable the family wholesale.

---

## Chart is slow

In order of impact:

1. **Leave `Redraw Every Tick` off.** On, the entire chart — several hundred boxes, lines and labels
   — is deleted and recreated on every tick. Off rebuilds twice per bar.
2. **Lower `Max Tracked FVGs (per side)`** from `120`. This is the largest per-bar cost: every gap in
   the set is re-examined each bar and scanned again by the entry trigger. `60–80` is comfortable.
3. **Lower `Max Zones Drawn`** from `18`. Working memory is six times whatever this is set to.
4. **Turn off layers you do not read** — Volume Imbalances (already off), Rejection Blocks, Internal
   Structure, Opening Gaps.
5. **Shorten ages** — Maximum FVG Age, Zone Max Age, Untouched Level Max Age.

Setting SMT to `Off` does **not** save the data request; the feed still loads.

---

## Chart is cluttered

The map is dense by design. Switch off, in this order, until it reads:

| Turn off | Removes |
|---|---|
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

Starting points, not gospel. Tune the threshold from `best` in every case. For four specific
instruments with per-market reasoning, see [PRESETS.md](PRESETS.md).

### Scalping — 1m to 5m

| Setting | Value |
|---|---|
| Swing Pivot Length | `10–12` |
| Internal Pivot Length | `5` |
| Higher Timeframe | `60` |
| HTF Bias Timeframe | `240` |
| Dealing Range Source / TF / Length | `HTF Range` / `60` / `20` |
| A+ Score Threshold | `50` |
| Min Stop Distance (x ATR) | `0.75` |
| Min Agreeing Factors | `3` |
| Min Delivery Efficiency / Range Edge Band | `0.28` / `0.30` |
| Entry Confirm Window | `4–6` |
| Min Bars Between Signals | `10` (5m) · `16` (3m) |
| Structure Valid For (bars) | `12` (5m) · `10` (3m) |
| Max A+ Signals per Killzone | `4` |
| Min Impulse Leg Size | `1.0` |
| Min Reward:Risk | `1.8`, measured to `Nearest Pool` |

Macros resolve here — leave them on, and let the timing weight do its work.

**The threshold is deliberately low here, and that is not a mistake.** With confluence and the
regime gate armed, the cascade is doing the filtering and the score is only ranking what survives
it. Pushing the threshold into the 60s on top of them stacks two filters on the same job and takes
the engine silent. If you are getting too many signals, read the Gates row and tighten a gate; the
threshold is the last dial, not the first.

**Run signals off one chart.** A 3-minute and a 5-minute chart of the same instrument see different
structure and will not agree about it, and taking signals from both means taking twice as many for
no extra information. Pick the slower one as the signal chart and use the faster only to place the
entry. Leaving *Dealing Range Source* on `HTF Range` is what stops the two disagreeing about
premium and discount, which they will do at any chart-swing setting.

Keep *Swing Pivot Length* short. The confirmation lag is paid on both sides of the pivot, so `20` on
a 5-minute chart is over three hours of delay before a major swing is even acknowledged — long past
the point the structure gate still considers it fresh.

### Intraday — 15m to 1h

| Setting | Value |
|---|---|
| Swing Pivot Length | `12–16` |
| Internal Pivot Length | `5–8` |
| Higher Timeframe | `240` (on 15m–1h) |
| HTF Bias Timeframe | `D` |
| A+ Score Threshold | `55–65` |
| Entry Confirm Window | `5` |
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
