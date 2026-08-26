# Instrument Presets

Starting configurations for four instruments, with the reasoning behind every value.

---

## Read this first

**These are not optimised settings.** This indicator has not been backtested — no walk-forward, no
sample of trades, no equity curve. Nothing below is derived from performance data, because none
exists.

What they *are*: configurations reasoned from two things that can actually be checked — how this
engine's mechanics behave at a given timeframe, and how these instruments behave structurally
(session character, correlation partner, whether they trade 24 hours). That is a useful starting
point and it is honest about being one.

**The last step of every preset is the same: tune the threshold from your own chart.** Nothing here
replaces that, and the engine gives you the number to do it with. See
[Tune the threshold](#tune-the-threshold--last-step-always).

---

## Most settings do not change per instrument

Worth understanding before copying values, because it saves a lot of pointless fiddling.

Every size, distance and tolerance in this engine is a **multiple of ATR**
([why](SETTINGS.md#the-atr-principle)). A `Min FVG Size` of `0.10` is one tenth of current ATR
whether that ATR is 4 points on gold or 140 dollars on Bitcoin. Those settings **self-adjust** — they
do not need an instrument-specific value, and changing them per instrument is usually a mistake.

Only four things genuinely differ between instruments:

| What differs | Why it matters |
|---|---|
| **Bar-count settings** | Pivot lengths, ages and windows are counted in bars, so they follow the *timeframe*, not the instrument |
| **Session relevance** | A 24/7 market and a session-driven market disagree about what a killzone means |
| **SMT partner** | Whether a correlated instrument exists and resolves |
| **Volume quality** | Traded size vs tick count vs nothing |

The presets below change only those. Everything unlisted stays at default.

---

## Universal baseline

Apply to all four before the per-instrument deltas.

| Setting | Value | Why |
|---|---|---|
| Non-Repainting Mode | `on` | Never turn this off for live trading |
| Redraw Every Tick | `off` | Rebuilds twice a bar instead of hundreds of times |
| **Require Valid Structure State** | `on` | The only gate armed to begin with — a reversal with no structure shift is not a setup |
| All other Hard Gates | `off` | Arm them one at a time, later, checking the tally after each |
| A+ Score Threshold | `52` | A starting bar, not a target — you will change this |
| Max Tracked FVGs | `100` | Slightly leaner than default; keeps the chart responsive |
| Show Watch Markers | `on` | These are how you calibrate. Leave them on until tuning is done. |

---

## XAUUSD — 5 minute

Gold on 5m is the engine's best case: violent, clean structure, strong session character, and a
reliable inverse partner in DXY.

| Setting | Value | Default | Reasoning |
|---|---|---|---|
| Swing Pivot Length | `15` | 20 | 20 bars either side on 5m is 100 minutes of confirmation lag *each way*. `15` keeps swings meaningful while cutting a third of the lag on a fast instrument. |
| Internal Pivot Length | `5` | 5 | Unchanged |
| Higher Timeframe | `60` | 60 | 12× the chart — a genuine intermediate term |
| HTF Bias Timeframe | `240` | 240 | Unchanged |
| Min Impulse Leg Size (x ATR) | `0.8` | 0.6 | Gold displaces hard. Demanding more separates real order blocks from ordinary two-candle pairs. |
| Highlight ICT Macros | `on` | on | **Resolves at 5m.** The timing weight can reach its full value here, which it cannot above 15m. This is the single biggest advantage of trading gold on 5m with this engine. |
| SMT | `Auto` → `TVC:DXY`, inverse | Auto | Correctly auto-selected. Gold's cleanest correlation. |
| PMH / PML | `off` | off | Monthly levels are usually far off-screen at 5m |

**Second gate to arm, when you want it tighter:** *Require Liquidity Taken*. Gold sweeps constantly,
and on gold the sweep usually **is** the setup — this gate costs little and removes a lot of noise.

---

## BTCUSD — 15 minute

Crypto breaks one of the engine's assumptions, and needs the only genuine weight change in this
document.

| Setting | Value | Default | Reasoning |
|---|---|---|---|
| Higher Timeframe | `240` | 60 | On a 15m chart, 60 is only 4× — too close to be an intermediate term, and it weakens the ITF gate |
| HTF Bias Timeframe | `D` | 240 | |
| **Timing (Macro / Silver Bullet) weight** | **`6`** | 12 | **The key change.** Killzones, macros and Silver Bullet windows are New York banking-session concepts. Bitcoin trades 24/7 and does not respect them the way FX and metals do. Halving this stops the engine rewarding an accident of the clock. |
| Premium / Discount Half weight | `14` | 12 | With timing demoted, move weight to something crypto genuinely does respect |
| **Require Active Killzone** | **never arm** | off | Would silence a 24/7 market for two thirds of every day |
| Opening Gaps (NDOG / NWOG) | `off` | on | There is no session roll for crypto to gap across |
| Asian Range Box | `off` | on | No meaningful Asia session |
| Judas Swing | `off` | on | Depends on the Asian range, which is not meaningful here |
| SMT | `Auto` → `ETHUSD` | Auto | Positive correlation, resolves reliably |

**Second gate to arm:** *Require Draw on Liquidity in Trade Direction*. Crypto trends hard and runs to
obvious liquidity; refusing trades against the draw suits it.

---

## AUDUSD — 15 minute

A genuinely session-bound pair, and the one where the Asia killzone earns its place.

| Setting | Value | Default | Reasoning |
|---|---|---|---|
| Higher Timeframe | `240` | 60 | Same 4× problem as BTC on 15m |
| HTF Bias Timeframe | `D` | 240 | |
| Volume weight | `5` | 8 | FX "volume" is **tick count, not traded size**. It proxies activity, not conviction, so it deserves less say. |
| SMT | `Auto` → `NZDUSD` | Auto | The tightest correlation in FX; SMT is unusually informative on this pair |
| Asia Killzone | keep `2000-0000` | default | AUD's primary session is Sydney/Tokyo, and the default window covers it. Do not narrow this to the London/NY windows. |

**Second gate to arm:** *Require Active Killzone*. AUDUSD is genuinely thin outside its sessions, and
this is one of the few pairs where that gate removes poor conditions rather than just removing bars.

---

## USDJPY — 15 minute

The most persistently trending of the four, which changes which gate is worth arming.

| Setting | Value | Default | Reasoning |
|---|---|---|---|
| Higher Timeframe | `240` | 60 | |
| HTF Bias Timeframe | `D` | 240 | |
| Volume weight | `5` | 8 | Same tick-volume caveat as AUDUSD |
| SMT | `Auto` → `USDCHF` | Auto | Both USD-base, so they express USD strength together |
| SMT (alternative) | `Manual` → `TVC:DXY`, Positive | — | Worth trying. USDJPY is a USD-strength expression and DXY is the broadest one available, so divergence between them is meaningful. |

**Second gate to arm:** *Require HTF Bias Alignment*. USDJPY trends for weeks at a time, which makes
higher-timeframe alignment a far stronger filter here than on a rangier pair. On something
mean-reverting the same gate mostly just removes signals.

---

## Tune the threshold — last step, always

None of the above is finished until this is done. It takes two minutes.

1. Load the chart with the preset applied and let it compute the full history.
2. Read the **Signal** row of the dashboard:

   ```
   blocked: ▲Structure ▼No tap · 158 fired / 731 setups · best 78 vs 52
                                                           └───┬───┘
                                          the highest score this engine has ever reached
   ```

3. Look at the `watch NN` markers on the chart — each number is that setup's score, so the whole
   distribution can be read directly off the chart.
4. Set **A+ Score Threshold** from what you see:

   | You want | Set threshold to |
   |---|---|
   | Most valid setups | `best − 20` |
   | The better half | `best − 10` |
   | Only the strongest | `best − 5` |

5. If `best` is **below** the threshold, nothing can fire. Lower it until signals appear, then work
   back up.

Repeat per instrument and per timeframe. The `best` figure is not portable — it reflects what that
market actually offers.

---

## What would make these actually "best"

Honest answer: a backtest. Specifically —

- Export signals over 12+ months per instrument (the JSON alert payload is built for this)
- Measure outcome against the stop and the three targets the engine already publishes
- Segment by **model name** — every signal is tagged, so it is possible to find out whether the 2022
  Model genuinely outperforms a bare PD Array Tap on a given instrument, and weight accordingly
- Segment by score band, to find where the threshold actually earns its keep

Until that exists, treat every number in this document as a considered opening position, not an
answer.
