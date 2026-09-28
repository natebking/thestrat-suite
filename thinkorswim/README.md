# TheStrat Suite Lite for thinkorswim

A thinkScript port of TheStrat Suite v3.1.1, scoped to what thinkorswim can draw natively.
Free and open source under MPL-2.0, like the Pine original.

**Status: v0.1.0, layer 1 of 3. Not yet loaded in thinkorswim.** The classifier is checked
against the grammar by a Python test (below); everything else still needs the in-app
verification pass at the bottom of this file.

## Install

1. In thinkorswim: Charts → Studies → Edit Studies → Create.
2. Name it `TheStratSuiteLite`, delete the placeholder code, and paste the contents of
   `TheStratSuite_Lite_TOS_v0.1.0.txt`.
3. OK → Apply. Pick a preset in the study's settings; colors are under the Globals tab.

## What "Lite" means

Everything that runs off the same classifier on each timeframe and fits thinkorswim's drawing
primitives. The scope is frozen so the port stays cheap to keep in sync with the Pine.

| Layer | Contents | Status |
|---|---|---|
| 1 | Classifier (1u/1d, 2u/2d, F2u/F2d, 3u/3d), Strat Candles and FTFC Candles bar coloring, timeframe label strip, FTFC label, the six Suite presets | **this version** |
| 2 | Signal combo bubbles (Inside Reversals, 2-2 Reversals, Inside Continuations; Failing 2s and the off-by-default signals as options), hammer/shooter filter | next |
| 3 | Trigger lines and magnitude targets for the chart timeframe plus the two nearest higher timeframes, chart-timeframe alerts | after 2 |

Not planned for Lite: exhaustion targets (pivot scan is slow without arrays; first add-on after
Lite), stops, Take Action Windows, Lead filter, Domino, Universal labels, full table mode,
debug panel, preview mode.

## How it differs from the Suite on TradingView

- **Timeframes below the chart's are not read.** thinkorswim rejects a secondary aggregation
  below the chart timeframe, so a lower slot is clamped, shown gray, and left out of FTFC. The
  Suite also grays these cells.
- **No 12H.** thinkorswim has no 12-hour aggregation. The Futures/Crypto preset runs 1H, 4H, D, W.
- **Label strip instead of a table.** One colored label per enabled timeframe, highest first,
  reading `<TF> <current candle>`, then the FTFC label. It follows the Compact table's Bar State
  coloring.
- **FTFC label ignores exhaustion.** The Suite's table FTFC cell can drop a timeframe that has
  already hit exhaustion. Lite has no exhaustion levels, so its FTFC label always uses every
  monitored timeframe, the same as the Suite's FTFC Candles paint and signal filters.
- **No Failing 2 flip highlight** on FTFC Candles yet.
- **Higher-timeframe bars are built by thinkorswim**, not by the Suite's straddle/glue/+12h
  logic. They must be checked against TradingView (checklist below) before anything is trusted.

## Classifier parity

`grammar/tests/test_tos_parity.py` reads the `StratState` script straight out of the study file,
evaluates it with a small interpreter, and compares it to `thestrat_grammar` across 45,000
random bars with ties on every boundary, all four Failing 2 methods, detection on and off.

```bash
cd grammar && python3 -m pytest -q tests/test_tos_parity.py
```

## Verification checklist (in thinkorswim)

Run these before calling layer 1 done. Record symbol, chart timeframe, preset and result.

- [ ] The study compiles and applies with no red error in the editor.
- [ ] Computed aggregations work: switching presets changes the strip without a
      "secondary period" error, on a 5m, 15m, 1H and D chart.
- [ ] Strat Candles on SPY 15m match the Suite's Strat Candles on TradingView bar for bar for
      one full session, including at least one F2u/F2d and one 3.
- [ ] Label strip on SPY 15m (TheStrat Classic) matches the Suite's Compact table (Bar State)
      live during RTH, including an inside bar on one timeframe.
- [ ] FTFC label matches the Suite's FTFC row at the same moment.
- [ ] ES 1H chart, Futures/Crypto preset: 4H boundaries land at 3PM/7PM/11PM/3AM/7AM/11AM PST
      (18/22/02/06/10/14 ET), and the D bar opens at 3PM PST (18:00 ET).
- [ ] A CME holiday week on ES (for example July 3 to 6, 2026): D and W candles match TradingView.
- [ ] SPY with extended hours on and off: D candles match TradingView's RTH daily bars.
- [ ] Reload test: note the Strat Candles and FTFC Candles on the last 20 bars, reload the
      chart, confirm nothing changed.
- [ ] Label text is readable on each state color (thinkorswim picks label text color itself).
- [ ] Mobile: note which parts render in the thinkorswim mobile app.
