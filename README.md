````md
# Prev Day H/L + Bias + US Session (Yesterday NY, 15m)

A TradingView Pine Script v5 indicator for Forex intraday trading that plots:

- **Previous day High/Low (PDH/PDL)** from the Daily timeframe (visible on all timeframes)
- **Daily Bias** based on yesterday’s close vs. the prior day’s range (D1[1] close vs D1[2] high/low)
- **US Session levels for *Yesterday* (NY time)** computed from **15m** data:
  - Session **High/Low** (green)
  - Session **Open/Close** (light pink)
- **EMA 20/50/200 (15m)** and a **Scenario** label (LONG/SHORT/NEUTRAL), optionally filtered by Bias

---

## Table of Contents

- [What This Indicator Does](#what-this-indicator-does)
- [How Bias Works](#how-bias-works)
- [How Scenario Works (EMA + optional Bias filter)](#how-scenario-works-ema--optional-bias-filter)
- [How US Session Levels Are Calculated](#how-us-session-levels-are-calculated)
- [Inputs / Settings](#inputs--settings)
- [How to Install](#how-to-install)
- [How to Use (Practical Workflow)](#how-to-use-practical-workflow)
- [Notes (Forex / Timezone / Data)](#notes-forex--timezone--data)
- [License](#license)

---

## What This Indicator Does

This indicator is designed to give you **intraday context** with minimal noise:

1. **PDH/PDL** from **Daily[1]**  
   These are horizontal extended lines that remain visible on any chart timeframe.

2. **Bias (Daily context)**  
   A simple 3-state bias computed from daily candles.

3. **US Session levels for Yesterday (NY time)**  
   Built from intraday 15m session data and plotted as extended levels.

4. **15m EMAs + Scenario label**  
   You get trend structure from EMAs and an on-chart Scenario label.

---

## How Bias Works

Bias uses **Daily candles** as source of truth:

- `candle 0` = today (current daily candle)
- `candle 1` = previous day
- `candle 2` = two days ago

The logic:

- **Bias: LONG** if `D1 close[1] > D1 high[2]`
- **Bias: SHORT** if `D1 close[1] < D1 low[2]`
- Otherwise **Bias: INSIDE**

In code (conceptually):

```text
c1 = close of daily candle[1]
h2 = high of daily candle[2]
l2 = low of daily candle[2]

if c1 > h2  => LONG
if c1 < l2  => SHORT
else        => INSIDE
````

---

## How Scenario Works (EMA + optional Bias filter)

Scenario is based on **15m EMA structure**, visible on any chart timeframe.

### EMAs used

* EMA 20
* EMA 50
* EMA 200

All computed on `15m`:

```text
ema20_15  = EMA(20) on 15m
ema50_15  = EMA(50) on 15m
ema200_15 = EMA(200) on 15m
```

### EMA-only Scenario

* **Scenario LONG** if:

  * `price > EMA200` AND `EMA20 > EMA50`
* **Scenario SHORT** if:

  * `price < EMA200` AND `EMA20 < EMA50`
* Otherwise **Scenario NEUTRAL**

### Optional Bias filter (`useBiasInScenario`)

You can force Scenario to only print LONG/SHORT when it aligns with Bias:

* If `useBiasInScenario = ON`:

  * LONG only when `(EMA_LONG AND Bias_LONG)`
  * SHORT only when `(EMA_SHORT AND Bias_SHORT)`
  * otherwise NEUTRAL

* If `useBiasInScenario = OFF`:

  * Scenario is EMA-only
  * Bias is still displayed, but does not filter Scenario

The on-chart scenario label prints:

```text
Scenario: LONG/SHORT/NEUTRAL | Bias: LONG/SHORT/INSIDE | BiasFilter: ON/OFF
```

---

## How US Session Levels Are Calculated

### Goal

Plot **US Session levels for yesterday** (not today), using **New York time** and built from **15-minute** data.

### Session definition

Default:

* `0930-1600` in `America/New_York`

### Computation

The script runs a session engine on 15m:

* Detect session start/end
* Track session:

  * open (first candle)
  * high (max)
  * low (min)
  * close (last candle)
* Store the last completed sessions (buffer of 6)
* Select the one matching **Yesterday’s NY date**
* Draw:

  * **High/Low** in green
  * **Open/Close** in light pink

### Why "Yesterday NY" (and not Daily[1] date)?

For **Forex**, the broker’s daily candle roll time can differ (often around NY 5pm).
To avoid mismatches/flashing, the script targets **yesterday by NY calendar day**, not by daily candle identity.

---

## Inputs / Settings

### Core toggles

* `Show Prev Day High/Low (Blue)`
* `Show Bias Labels on Blue Lines`
* `Show US Session Levels for YESTERDAY (NY), calc on 15m`

### US session controls

* `US Session (NY time)` — default `0930-1600`
* `US Session Timezone` — default `America/New_York`

### EMA controls

* `Show EMAs (20/50/200) from 15m`
* `EMA 20 Length`
* `EMA 50 Length`
* `EMA 200 Length`

### Scenario/Bias control

* `Use Bias filter in Scenario (EMA + Bias must align)`
  If ON: scenario requires alignment with Bias.

### Styling

* Width controls for blue/green/pink
* Label colors

---

## How to Install

1. Open **TradingView**
2. Go to **Pine Editor**
3. Paste the script code
4. Click **Add to chart**
5. Optional: open indicator settings and adjust session/EMA options

---

## How to Use (Practical Workflow)

A clean workflow for Forex intraday:

1. **Use Bias as daily context**

   * LONG bias favors buys
   * SHORT bias favors sells
   * INSIDE bias favors range/mean-reversion behavior

2. **Use Scenario as intraday trend filter**

   * Scenario LONG: look for longs only (unless you purposely trade counter-trend)
   * Scenario SHORT: same for shorts
   * NEUTRAL: reduce aggressiveness / wait for clarity

3. **Use PDH/PDL and US levels as structure**

   * PDH/PDL are common liquidity / reaction levels
   * US session H/L often acts as intraday boundaries
   * Session O/C provides additional reference

4. **Combine with your entry trigger**
   The indicator is designed to provide context, not force an entry.
   Use your own trigger (market structure, pattern, orderflow, etc.).

---

## Notes (Forex / Timezone / Data)

* **Forex daily candle roll time** varies by broker and symbol.
  This script uses **NY calendar day** for the US session selection to remain stable.

* **Session levels depend on 15m availability**.
  If the chart does not have enough historical 15m data loaded, you may need to scroll back a bit to let TradingView load more.

* On weekend/holidays, the script keeps a buffer of several sessions (6) so yesterday can still resolve correctly.

---

## License

This script references the **Mozilla Public License 2.0** as included in the original header.
See: [https://mozilla.org/MPL/2.0/](https://mozilla.org/MPL/2.0/)

```
```
