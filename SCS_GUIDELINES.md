# Sweep Closure Sequencer MTF [SCS-MTF] — Guidelines

Multi-timeframe variant. Reversals are detected on a configurable HTF.
The equilibrium box is **fractal**: it represents an HTF setup but renders
ONLY on the aligned LTF chart, so it functions as a precise LTF entry zone.

## Aligned timeframe pairs

| HTF (detection) | Aligned LTF (rendering) |
|---|---|
| 15m | 1m |
| 30m | 3m |
| 1H  | 5m |
| 4H  | 15m |
| 1D  | 1H |
| 1W  | 4H |

If the chart TF matches neither the chosen HTF nor its aligned LTF, nothing draws.

## What it draws

| Element | HTF chart | Aligned LTF chart |
|---|---|---|
| Closure line at swept extreme | yes | yes (toggleable) |
| Equilibrium box (open(C4) ↔ midpoint) | no | yes |
| HTF opening line | no | yes |

No labels, no text. Lines and boxes only.

## Logic

When a new HTF candle (call it C4) opens, the previous HTF candle (C3) is
evaluated against the one before it (C2):

- **Bullish reversal**: `low(C3) < low(C2)` AND `close(C3) > low(C2)`.
- **Bearish reversal**: `high(C3) > high(C2)` AND `close(C3) < high(C2)`.

On a reversal:

- Draw the closure line at the swept extreme (C2 low for bull, C2 high for bear).
- On aligned LTF only, draw the equilibrium box from `open(C4)` to the midpoint of `[open(C4), swept extreme]`.
- The HTF opening line at `open(C4)` updates on every new HTF candle (independent of reversals).

## Bias filter

- **Auto**: draw both bullish and bearish reversals.
- **Bullish only**: only bullish reversals (sweeps of lows that close back above).
- **Bearish only**: only bearish reversals (sweeps of highs that close back below).

## Settings

| Group | Input | Default |
|---|---|---|
| Timeframe | HTF (reversal source) | 240 (4H) |
| Bias | Bias | Auto |
| Closure Line | Show / Color / Width / Style / Extension (HTF bars) / Render on LTF | true / red / 2 / Solid / 1 / true |
| Equilibrium Box | Show / Bull/Bear fill / Bull/Bear border / Border width / Extension (HTF bars) | true / blue 80% / blue 50% / 1 / 1 |
| HTF Opening Line | Show / Color / Width / Style / Keep historical | true / orange / 1 / Solid / false |
| Display | Last N days | 7 |

## Verification flow

1. Pick HTF = 4H, chart TF = 15m. Apply the indicator.
2. Wait until a 4H candle closes that satisfies the reversal rule. At the open of the next 4H candle, you should see:
   - A red closure line on the 15m chart at the swept C2 high/low.
   - A translucent blue equilibrium box from the new 4H candle's open to its 50% level.
   - An orange line at the 4H open extending right through the 15m bars.
3. Switch the chart to 4H: the equilibrium box disappears (LTF-only), the closure line stays, the HTF opening line disappears (LTF-only).

## Tuning tips

- **Trade-day focus**: set `Last N days` to 1–2 to keep only fresh setups.
- **Tight entries**: leave `Extension (HTF bars)` at 1 so the box ends at the next HTF candle.
- **Trail the level**: turn on `Keep historical opens` to record every HTF open as a static line for reference.
- **Direction-locked sessions**: set bias to Bullish-only or Bearish-only when the higher narrative is one-directional.

## Notes

- Detection uses `request.security(... lookahead_on)` on closed-HTF series (`high[1]`, `low[1]`, `close[1]`). This is safe because we only act on `isNewHtfBar`, i.e. the bar after C3 has fully closed.
- `barsPerHtf` is computed from `timeframe.in_seconds(htf) / timeframe.in_seconds(chart)` so line/box widths look proportionally identical between HTF and aligned LTF charts.
- `max_lines_count` and `max_boxes_count` are 500 each; oldest drawings auto-prune.
