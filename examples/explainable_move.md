# Explainable Move Example

One of The Solver's core goals is to make each deduction understandable rather than simply producing a completed puzzle.

The following example illustrates the intended shape of an explainable move.

---

## Line-State Notation

The Solver uses a compact text representation for Nonogram line states:

- `#` = occupied cell
- `x` = confirmed empty cell
- `.` = unknown / unresolved cell

This notation allows line states to be represented compactly in tests, transcripts, and explanations.

---

## Example

```text
Line: Row 8
Clues: 3

Previous state:
....#..........

Updated state:
...###.........