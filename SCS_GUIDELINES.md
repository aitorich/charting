# Sweep Closure Sequencer [SCS] — Guidelines

Visual style modeled after `fractal_candle_closure.pine` from the
`feat/smt-quarter-sequences` branch. Same drawing idiom, narrower scope:
just the closure line and the equilibrium box.

## What it draws

1. **Closure line** — a short red horizontal line at the swept extreme of C1, drawn on the C2 bar (the bar that swept C1).
2. **Equilibrium box** — a translucent blue rectangle, drawn on the C3 bar (one bar after C2), spanning from `open(C3)` to the midpoint of `[open(C3), swept extreme of C1]`.

No text. No labels. Lines and boxes only.

## Logic

Immediate C1/C2 detection — every bar is potentially a C2 of the bar before it.

- **Bullish reversal**: `low < low[1] and close > low[1]` → C2 wicked C1's low and closed back above. Draw closure line at `low[1]`. On the next bar, draw the equilibrium box from `open` down to the midpoint.
- **Bearish reversal**: `high > high[1] and close < high[1]` → C2 wicked C1's high and closed back below. Mirror image.

This matches the rolling-reference rule at depth 1: each bar is treated as a fresh C1 candidate for the next bar's evaluation. If the current bar isn't a reversal, the next bar gets evaluated against this one.

## Install

1. TradingView → Pine Editor.
2. Paste `SweepClosureSequencer.pine`.
3. Save → Add to chart.

## Settings

| Group | Input | Default | Purpose |
|---|---|---|---|
| Display | Show Last N Days | 3 | Limit drawings to the most recent N days |
| Closure Line | Show Closure Line | true | Toggle the line |
| Closure Line | Line Color | red `#ff1744` | Line color |
| Closure Line | Line Width | 1 | 1–5 |
| Closure Line | Line Style | Solid | Solid / Dashed / Dotted |
| Closure Line | Line Extra Bars | 1 | How far past C2 the line extends |
| Equilibrium Box | Show Equilibrium Box | true | Toggle the box |
| Equilibrium Box | Bull / Bear Box Color | translucent blue `#2196f3` | Fill color |
| Equilibrium Box | Bull / Bear Border | semi-translucent blue | Border color |
| Equilibrium Box | Border Width | 1 | 0–3 |
| Equilibrium Box | Box Extra Bars | 2 | Right extension of the box |

## Verification

Apply on a liquid pair (EURUSD, GBPUSD, NQ, ES) on 1H or 4H. You should see:

- Red horizontal lines bridging two adjacent candles whenever the right candle wicked the left candle's high or low and closed back inside.
- Blue translucent rectangles appearing on the bar AFTER each red line, sized to the upper or lower half of `open(C3)` ↔ swept extreme.

If you want denser signals: lower the timeframe (15m, 5m). If you want fewer: increase TF or shorten `Show Last N Days`.

## Tuning tips

- **Cleaner chart on trending pairs**: increase `Line Width` to 2, set `Line Extra Bars` to 0 so the lines don't overlap subsequent candles.
- **Wider eq boxes**: bump `Box Extra Bars` to 5–10.
- **Different colors per direction**: split bull and bear box colors (already exposed as separate inputs).

## Limitations / known behaviour

- Pure 2-bar comparison. No deeper rolling reference, no ATR filters, no trend gating.
- The equilibrium box appears one bar after the reversal because it needs `open(C3)`. On the live (developing) bar that just produced a reversal, the box won't render until the next bar opens.
- `max_lines_count` and `max_boxes_count` are 500 each; the engine auto-prunes oldest drawings beyond that.
