# TheStrat Suite for thinkorswim

A thinkScript port of TheStrat Suite v3.1.1. Free and open source under MPL-2.0, like the Pine
original. The goal is a **full port** of every Suite feature; `FULL_PORT_PLAN.md` maps each one
to a thinkScript approach, lists the hard platform limits and their equivalents, and sets the
build order. The study shipped today is layer 1 of that plan (named "Lite" while it is partial).

**Status: v0.1.0, layer 1. Not yet loaded in thinkorswim.** The classifier is checked
against the grammar by a Python test (below); everything else still needs the in-app
verification pass at the bottom of this file.

## Install

1. In thinkorswim: Charts → Studies → Edit Studies → Create.
2. Name it `TheStratSuiteLite`, delete the placeholder code, and paste the contents of
   `TheStratSuite_Lite_TOS_v0.1.0.txt`.
3. OK → Apply. Pick a preset in the study's settings; colors are under the Globals tab.

## Roadmap

Layer 1 (this version): classifier (1u/1d, 2u/2d, F2u/F2d, 3u/3d), Strat Candles and FTFC Candles
bar coloring, timeframe label strip, FTFC label, the six Suite presets.

Layers 2 to 12 (probes and a code generator, HTF data core, detection engine and debug panel,
table, targets, levels and labels, stops and Take Action Windows, F2 flip highlight, preview,
alerts, packaging) are laid out in `FULL_PORT_PLAN.md`.

## How it differs from the Suite on TradingView

- **Timeframes below the chart's are not read.** thinkorswim rejects a secondary aggregation
  below the chart timeframe, so a lower slot is clamped, shown gray, and left out of FTFC. The
  Suite also grays these cells.
- **No 12H.** thinkorswim has no 12-hour aggregation. The Futures/Crypto preset runs 1H, 4H, D, W.
  The full port synthesizes 12H from chart bars (see `FULL_PORT_PLAN.md`).
- **Futures 4H is likely misaligned in v0.1.0.** thinkorswim appears to anchor time bars at
  midnight CT, so its native 4H on futures would run 10P/2A/6A/10A/2P/6P PT (01/05/09/13/17/21 ET)
  instead of the Suite's 3P/7P/11P/3A/7A/11A PT (18/22/02/06/10/14 ET). This affects the Day Trade
  and Futures/Crypto presets on futures. The checklist item below confirms it; the full port builds
  futures 4H from chart bars.
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
