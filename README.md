# Gold SMC Analyzer (Android)

An Android app that analyzes **XAU/USD** with **Smart Money Concepts (SMC)** and gives
trade setups with an entry, stop loss and two take-profit levels. It also shows a live
TradingView chart.

## Features

| Tab | What it shows |
|---|---|
| **Live Chart** | Real-time TradingView chart (default `OANDA:XAUUSD`) with all of TradingView's drawing tools and indicators |
| **SMC Chart** | Candles with the SMC analysis drawn on top: order blocks, fair value gaps, BOS/CHoCH, liquidity (EQH/EQL), sweeps, premium/discount/EQ, and the entry/SL/TP lines. You can switch between the entry timeframe and the bias timeframe. |
| **Signals** | A live TradingView quote, the **primary setup** and an **alternate setup** (Buy/Sell limit or market, Entry, SL, TP1, TP2, R:R, a 0–100 confluence score with an A+/A/B/C grade, and the list of confluences), a written market summary, structure, points of interest, liquidity, and TradingView's technical rating |
| **Settings** | Data source, API key, price offset, timeframes, swing length, entry mode, stop buffer, TradingView symbol, alerts |

When a new setup scores above your minimum, or when price enters the setup's zone, the app
sends an Android notification. Alerts only run while the app is open.

## The SMC engine (`app/src/main/assets/js/smc.js`)

1. **Swings**: fractal pivot highs and lows (N candles on each side, set in Settings).
2. **Market structure**: when a candle closes through the last confirmed swing, the app marks
   a **BOS** (the trend continues) or a **CHoCH** (the trend reverses).
3. **Order blocks**: the last opposite-colored candle before the move that broke structure.
   The app tracks whether each block has been touched (mitigated) or broken (invalidated).
   ★ marks a fresh block that hasn't been touched.
4. **Fair value gaps**: 3-candle gaps of at least 0.15 × ATR, tracked until they fill.
5. **Liquidity**: equal highs (buy-side liquidity) and equal lows (sell-side liquidity), plus
   **sweeps**, where a wick runs a swing and the candle closes back inside.
6. **Dealing range**: the range between the major swings, split into premium, discount and
   equilibrium (EQ).
7. **Setups (multi-timeframe)**: the higher timeframe (4H by default) sets the bias. On the
   entry timeframe (15m by default), each active order block or FVG gets a score:
   - HTF trend agrees: +25
   - LTF break agrees: +15
   - Zone is in discount (buys) or premium (sells): +15
   - Zone is also in HTF discount or premium: +5
   - Order block overlaps an FVG: +10
   - Zone is an order block: +5
   - Zone is fresh (untouched): +10
   - Liquidity was swept just before: +10
   - Zone sits inside an HTF zone: +10
   - Price is in the London or New York kill zone: +5
   - Final R:R is 3 or better: +5
   - Points come off the further the zone is from price.
   - **Entry**: the edge of the zone, or 50% of the order block if you choose that in Settings.
   - **SL**: beyond the zone plus 0.3 × ATR. It is never tighter than 0.6 × ATR.
   - **TP1 / TP2**: the next internal liquidity (swing highs or lows, equal highs or lows),
     then the external range liquidity. Each TP must be at least 1.5R away.

## Data sources: important

TradingView **does not provide a public data API**. Its widgets are sealed iframes, so no app
can read their prices. That's why the app works like this:

* The **Live Chart** and the quote/rating widgets are real-time **TradingView** data.
* The **SMC engine** needs raw candles, so it uses one of these:
  * **PAXG/USDT on Binance** (default). It's free, needs no key, and streams in real time over a
    WebSocket. 1 PAXG = 1 troy ounce of gold, so it follows spot gold closely, usually within a
    few dollars. Use **Settings → Price offset** to match your broker or the TradingView symbol
    exactly. If Binance is blocked where you live, the app falls back to `binance.vision`.
  * **XAU/USD spot from Twelve Data**. Get a free API key at https://twelvedata.com and paste
    it in Settings. The free plan allows 800 requests a day, so the app refreshes every 2.5 min.

If you have a paid feed (for example OANDA v20 or Polygon), add another loader in `js/feed.js`
that returns candles as `{time, open, high, low, close}`.

## Build the APK

### Option A: GitHub Actions (no Android Studio needed)
1. Create a new GitHub repository and push this folder to it.
2. Open the **Actions** tab. The **Build APK** workflow runs on its own; you can also start it
   with *Run workflow*.
3. When it finishes, download the **GoldSMC-apk** artifact, unzip it, copy
   `app-release.apk` to your phone and install it. You need to allow "install unknown apps".

### Option B: Android Studio
1. **File → Open** and pick this folder. Let Gradle sync; it downloads the SDK parts it needs.
2. Run on a device, or use **Build → Build APK(s)**.

### Option C: Command line (Android SDK installed)
```bash
./gradlew assembleRelease     # APK: app/build/outputs/apk/release/app-release.apk
```

Requirements: JDK 17, Android SDK 34, and a device running Android 8.0 or newer.
The release build is signed with the debug key so it installs directly. Set up your own
signing key before you publish it to Google Play.

## Project layout
```
app/src/main/
  java/com/goldsmc/analyzer/   MainActivity (WebView host), JsBridge, Notifier
  assets/index.html            UI
  assets/js/smc.js             SMC analysis engine (pure JS, unit-testable in Node)
  assets/js/feed.js            Real-time data feed (Binance WS / Twelve Data)
  assets/js/app.js             UI controller, charts, alerts
  assets/js/lightweight-charts.standalone.production.js  (TradingView Lightweight Charts, Apache-2.0)
```

## Disclaimer
This is an educational analysis tool, **not financial advice**. SMC setups are probabilities,
not guarantees. Trading gold with leverage carries a high risk of loss. Always use your own
judgement and proper risk management.
