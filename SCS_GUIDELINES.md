# Sweep Closure Sequencer [SCS] — Guidelines

## What it draws

Only two primitives, no labels or text:

1. **Closure line** — a thin horizontal line at the swept extreme, spanning from the reference candle to the candle that swept it. Drawn on every sweep (both Reversal and Continuation).
2. **Equilibrium box** — a translucent rectangle drawn after a Reversal closure. Spans from the open of the next candle to the midpoint of `open(next) ↔ swept extreme`, extending forward by `Eq box extension` bars.

## How to install

1. Open TradingView → Pine Editor.
2. Paste the contents of `SweepClosureSequencer.pine`.
3. Click **Save** (Ctrl/Cmd + S) and give it any local name.
4. Click **Add to chart**.

## Settings

| Input | Default | Purpose |
|---|---|---|
| Show closure line | true | Draw the horizontal line at swept extremes |
| Show equilibrium box | true | Draw the post-reversal eq box |
| Eq box extension (bars) | 10 | How many bars right the box stretches |
| Closure line | black | Color of the closure line |
| Closure line width | 2 | 1-4 |
| Eq box (bull rev) | translucent green | Box color when bullish reversal fires |
| Eq box (bear rev) | translucent red | Box color when bearish reversal fires |
| Eq box border | translucent gray | Border of the eq box |

## How the logic runs

The script keeps two independent rolling references:

- **Bearish ref** — tracks the active low. If a future candle's wick goes below this low (sweep) and:
  - **closes back above** the level → **bullish reversal** → draws closure line + equilibrium box on the next bar.
  - **closes below** the level → **bullish-side continuation** → draws closure line only, rolls the reference forward.
- **Bullish ref** — mirror image, tracking highs for bearish reversals/continuations.

Sweep is a strict comparison (`low < refLow` or `high > refHigh`). Equal levels are ignored.

## Why nothing showed before

Three causes were addressed in this revision:

1. **`var int x = na` initialization** could leave the references unevaluated in some bar configurations. The fix uses sentinel values (`-1` for ints, `0.0` for floats) and lazy-initialises on the first bar where the script runs.
2. **`barstate.isfirst` is only true on the absolute first bar of the dataset** and could be skipped if the series wasn't fully ready. Replaced with a sentinel-based check that always primes the state.
3. **Default box transparency was 80** — boxes were technically drawn but barely visible against the chart background. Lowered to 70 and added an opaque-ish gray border so the box edges read clearly.

## How to verify it's working

1. Apply the indicator to any liquid instrument with active price action (e.g. EURUSD 4H).
2. Look for short black horizontal lines bridging two adjacent candles at one of their wick levels — those are closure lines.
3. After a wick is taken and the candle closes back inside the prior range, you should see a green/red translucent rectangle appear on the next bar.

If you still see nothing:

- Open the indicator's status icon on the chart and check for compile errors.
- Confirm the chart has at least 50 bars of history loaded.
- Bump `Closure line width` to 3 or 4 and reduce `Eq box (bull/bear rev)` transparency for a stress-test.

## Tuning tips

- **Cleaner chart on trending pairs**: turn off `Show closure line` — closure lines are drawn on every sweep, so strong trends produce many of them. Equilibrium boxes alone show only the resolved reversal points.
- **Wider eq boxes for swing trading**: bump `Eq box extension` to 30-50.
- **Aligned LTF rendering**: this indicator runs on whatever TF the chart is on. To draw HTF eq boxes on a lower timeframe, wrap the calculation in `request.security(syminfo.tickerid, "HTF", ...)` with the desired HTF; that's a future enhancement, not required for the current behaviour.

## Limitations / known behaviour

- The reference rolls forward on **every** sweep (reversal or continuation). A long monotonic trend will produce many closure lines on the trend side. This is by design — each sweep is a discrete event.
- The equilibrium box uses the open of the bar **after** the reversal; on the very last (live) bar, the box for a reversal that just fired will only appear once the next bar opens.
- `max_lines_count` and `max_boxes_count` are capped at 500 each; older drawings are auto-removed by the engine if exceeded.
