# Sweep Closure Sequencer MTF [SCS-MTF] — Guidelines

Multi-timeframe variant. Reversals are detected on a configurable HTF.
The equilibrium box is **fractal** — it represents an HTF setup but renders
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
| C2 closure line (depth 1) | yes | yes (toggleable) |
| C3 closure line (depth 2) | yes | yes (toggleable) |
| Equilibrium box | no | yes |
| HTF opening line | no | yes (full horizontal, extend right/both) |

No labels, no text. Lines and boxes only.

## Logic

When a new HTF candle (C4) opens, the indicator evaluates the just-closed sequence:

### Depth 1 — C2 closure (C3 reverses against C2)
- **Bullish**: `low(C3) < low(C2)` AND `close(C3) > low(C2)`
- **Bearish**: `high(C3) > high(C2)` AND `close(C3) < high(C2)`
- Closure line drawn at the swept extreme (`low(C2)` / `high(C2)`).

### Depth 2 — C3 closure (C3 reverses against C1, after C2 continued)
- C2 was a continuation of C1: `low(C2) < low(C1)` AND `close(C2) < low(C1)` (mirror for bearish).
- C3 then reversed back across C1's level: `low(C3) < low(C1)` AND `close(C3) > low(C1)`.
- Closure line drawn at C1's swept extreme (`low(C1)` / `high(C1)`).

### Equilibrium box
Drawn whenever either depth fires. Range = `open(C4) ↔ low(C3)` (bull) or `open(C4) ↔ high(C3)` (bear). Box spans from `open(C4)` to the midpoint of that range. Aligned LTF only.

## Bias filter

- **Auto**: both directions.
- **Bullish only**: only sweeps of lows that close back above.
- **Bearish only**: only sweeps of highs that close back below.

## Settings

| Group | Input | Default |
|---|---|---|
| Timeframe | HTF (reversal source) | 240 (4H) |
| Bias | Auto / Bullish only / Bearish only | Auto |
| Closure Line — C2 (depth 1) | Show / Color / Width / Style / Extension / Render on LTF | true / red `#ff1744` / 2 / Solid / 1 / true |
| Closure Line — C3 (depth 2) | Show / Color / Width / Style / Extension / Render on LTF | true / orange `#ff9100` / 2 / Solid / 1 / true |
| Equilibrium Box | Show / Bull/Bear fill / Bull/Bear border / Border width / Extension | true / blue 80% / blue 50% / 1 / 1 |
| HTF Opening Line | Show / Color / Width / Style / Extend / Keep historical opens | true / orange / 2 / Solid / Right / true |
| Display | Last N days | 7 |

The HTF opening line is fully configurable: pick **Right** to extend forward only, **Both** to extend infinitely in both directions across the chart, **None** to draw a fixed-length segment.

## Verification flow

1. HTF = 4H, chart TF = 15m. Apply the indicator.
2. Wait for a 4H bar to close that satisfies the reversal rule.
3. At the open of the next 4H bar you should see (on 15m):
   - **Red** closure line at C2's swept extreme (depth 1).
   - **Orange** closure line at C1's swept extreme (depth 2) when applicable.
   - Translucent **blue** equilibrium box from the new 4H open to the 50% level.
   - **Orange** horizontal HTF opening line extending right at the new 4H open price.
4. Switch the chart to 4H: equilibrium box and HTF opening line disappear (LTF-only). Closure lines remain.

## Tuning tips

- **Trade-day focus**: set `Last N days` to 1–2 to keep only fresh setups.
- **Historical HTF opens**: turn on `Keep historical opens` to leave one short horizontal segment per past HTF candle, like SMT Quarter Sequences does for daily opens.
- **Direction-locked sessions**: set bias to Bullish-only or Bearish-only when the higher-TF narrative is one-directional.
- **Disable depth 2** if you want a simpler chart: turn off `Closure Line — C3 (depth 2) → Show`.

## Notes

- Detection uses `request.security(... lookahead_on)` on closed-HTF series only (`high[1..3]`, `low[1..3]`, `close[1..2]`). This is safe: each `[n]` already references a fully-closed HTF candle, so there is no future leak.
- The new HTF bar's open is captured directly via the chart's `open` at the HTF boundary — no security call, no lookahead concerns.
- `barsPerHtf` is computed from `timeframe.in_seconds(htf) / timeframe.in_seconds(chart)` so closure-line widths look proportionally identical across HTF and aligned LTF charts.
- `max_lines_count` and `max_boxes_count` are 500 each; oldest drawings auto-prune.
