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









## 🧠 اصل ماجرا: EMAها برای چی استفاده شدن؟

ما از EMAها **برای تشخیص جهت بازار (Context / Trend)** استفاده کردیم، نه برای ورود دقیق.

همه EMAها:

* روی **تایم‌فریم 15 دقیقه** محاسبه می‌شن
* ولی روی **هر تایم‌فریمی که باشی** نمایش داده می‌شن

---

## 📐 تعریف هر EMA

| EMA     | نقش                  |
| ------- | -------------------- |
| EMA 200 | فیلتر جهت اصلی بازار |
| EMA 50  | روند میان‌مدت        |
| EMA 20  | مومنتوم کوتاه‌مدت    |

---

## ✅ سناریوی LONG (سناریوی صعودی)

سناریو می‌گه:

> «بازار در یک ساختار صعودی قابل‌اعتماد هست»

### شرط‌ها:

```text
1️⃣ قیمت بالای EMA 200 باشد
2️⃣ EMA 20 بالای EMA 50 باشد
```

### یعنی چه از دید تریدر؟

* EMA200 زیر قیمت → بازار **Bullish Context**
* EMA20 بالای EMA50 → مومنتوم صعودی فعاله

📌 نتیجه:

```text
Scenario: LONG
```

> 🔥 یعنی فقط دنبال لانگ باش، شورت‌ها مشکوکن

---

## ❌ سناریوی SHORT (سناریوی نزولی)

### شرط‌ها:

```text
1️⃣ قیمت زیر EMA 200 باشد
2️⃣ EMA 20 زیر EMA 50 باشد
```

### تفسیر:

* قیمت زیر EMA200 → بازار نزولیه
* EMA20 زیر EMA50 → فشار فروش فعاله

📌 نتیجه:

```text
Scenario: SHORT
```

> 🔥 یعنی فقط دنبال شورت باش

---

## ⚠️ سناریوی NEUTRAL (خنثی / گیج)

اگر **هیچ‌کدوم از دو حالت بالا نباشه**:

مثلاً:

* قیمت بالای EMA200 ولی EMA20 و EMA50 قاطی‌ان
* یا قیمت زیر EMA200 ولی EMA20 > EMA50
* یا بازار فلت

📌 نتیجه:

```text
Scenario: NEUTRAL
```

> ⚠️ یعنی بازار تصمیم نگرفته → سیگنال‌محور معامله نکن

---

## 🔗 نقش Bias (اگر روشن باشد)

اگر این گزینه روشن باشه:

```text
Use Bias filter in Scenario = ON
```

آن‌وقت:

### LONG فقط وقتی معتبر است که:

* Scenario = LONG
* Bias = LONG (کلوز روز قبل بالای High دو روز قبل)

### SHORT فقط وقتی معتبر است که:

* Scenario = SHORT
* Bias = SHORT

اگر هم‌راستا نباشن → سیگنال حذف می‌شود.

---

## 🎯 خروجی نهایی در پنل

پنل بهت اینا رو می‌گه:

* SIGNAL: LONG / SHORT / WAIT
* Scenario: LONG / SHORT / NEUTRAL
* Bias: LONG / SHORT / INSIDE
* Score (از ۵)
* Zone (High Probability / Conflict)

---

## 🧠 خلاصه خیلی کوتاه (حفظی)

```text
EMA200 = جهت بازار
EMA20 vs EMA50 = قدرت حرکت
```

| وضعیت                        | نتیجه          |
| ---------------------------- | -------------- |
| قیمت بالای 200 + 20 بالای 50 | Scenario LONG  |
| قیمت زیر 200 + 20 زیر 50     | Scenario SHORT |
| بقیه حالات                   | NEUTRAL        |

---
👌


```
```
