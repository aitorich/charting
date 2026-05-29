# Sweep Closure Sequencer — change log

Snapshot of decisions for this conversation, preserved for future sessions.

## Logic

- **C2 reversal** (depth 1): on the new HTF bar, last-closed HTF bar (C3) sweeps the previous HTF bar (C2) and closes back beyond C2's swept extreme.
- **C2 continuation** (depth 1): same sweep but close stays past C2's swept extreme. Drawn with a different color but no equilibrium box.
- **C3 reversal** (depth 2): C2 was a continuation of C1, then C3 reverses against C1's swept extreme. Drawn at C1's level.
- **Equilibrium box**: drawn whenever a reversal fires. Range = `open(C4) ↔ low(C3)` (bull) or `open(C4) ↔ high(C3)` (bear). Box spans from `open(C4)` to the midpoint.
- **Bias filter**: Auto / Bullish only / Bearish only.

## Aligned timeframe rendering

| HTF | Primary LTF | Middle TF |
|---|---|---|
| 1D  | 1H  | 4H |
| 4H  | 15m | 1H |
| 1H  | 5m  | 15m |
| 30m | 3m  | 15m |
| 15m | 1m  | 5m |

- **Closure lines** render on HTF, optionally on Primary LTF and Middle TF.
- **Equilibrium box** renders on Primary LTF AND Middle TF only (never HTF).
- **HTF opening line** (vertical) renders on Primary LTF only.

## Visual rules

- No labels, no text. Lines and boxes only.
- Closure line at swept extreme — visually identifies which depth fired (C2 red, C2 cont gray, C3 orange).
- Equilibrium box translucent blue (configurable per direction).
- HTF opening line vertical, dotted yellow by default, configurable extend (Both/Up/Down/None) and history.

## Files

- `SweepClosureSequencer.pine` — standalone MTF indicator (this branch / `sweep-closure-sequencer`).
- `smt_quarter_sequences.pine` — original SMT Quarter Sequences (4HR) augmented with the SCS module on `feat/scs-into-quarter-sequences`. HTF opening line omitted in that integration to avoid duplicating Daily Open Line.
- `logic-canvas.html` — visual reference for the four scenarios (reversal/continuation/rolling/equilibrium) on `logic-canvas`.

## Multi-pair Pine reference

`smt_triad.pine` on branch `feat/smt-triad-indicator` is the canonical
multi-symbol pattern in this repo (`request.security` for each symbol).
`smt_quarter_sequences.pine` uses the same pattern with NQ/YM hard-wired.
