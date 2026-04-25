# Wordle Windmill Solver

A C++ program that, given any valid Wordle target word, finds five guess words that together form a **windmill** — a specific visual pattern where the color results across all five rows create a pinwheel shape on the board.

## What is a Windmill?

A Wordle windmill is a challenge where you share a board in which all five rows follow a strict color pattern:

```
Row 1:  Y  x  Y  Y  Y
Row 2:  Y  x  Y  x  x
Row 3:  Y  Y  G  Y  Y   ← center green (the "hub")
Row 4:  x  x  Y  x  Y
Row 5:  Y  Y  Y  x  Y
```

`G` = green (correct letter, correct position)  
`Y` = yellow (correct letter, wrong position)  
`x` = black (letter not in the word)

The pattern forms four "blades" radiating from the single green tile at the center of the 5×5 board.

### Hard vs Soft Windmill

| Type | Description |
|------|-------------|
| **Hard Windmill** | All five rows match the exact pattern above — yellows and greens land exactly as shown |
| **Soft Windmill** | The black positions are fixed exactly, but non-black cells can be either yellow **or** green |

Both types are valid windmills. Hard windmills are rarer (~130 of 2315 possible targets). A soft windmill is accepted when no hard solution exists for the target.

> **Game integrity rule:** No guess in rows 1–4 may score all-green (`GGGGG`), because that would end the game before row 5 is reached. This constraint is enforced automatically.

---

## Word Lists

Word lists are sourced directly from the original NYT Wordle JavaScript bundle:

| List | Count | Source |
|------|-------|--------|
| Answer list | 2,315 words | [cfreshman/a03ef2cba789d8cf00c08f767e0fad7b](https://gist.github.com/cfreshman/a03ef2cba789d8cf00c08f767e0fad7b) |
| Valid guesses | 10,657 words | [cfreshman/cdcdf777450c5b5301e439061d29694c](https://gist.github.com/cfreshman/cdcdf777450c5b5301e439061d29694c) |

Both lists are hardcoded in `wordle_solver.cpp`. The solver searches the combined pool of ~12,972 unique words.

---

## Build

Requires a C++11 (or later) compiler.

```bash
g++ -O2 -o solver wordle_solver.cpp
```

---

## Usage

Pass the target word on standard input:

```bash
echo "acrid" | ./solver
echo "crane" | ./solver
echo "abbey" | ./solver
```

Or run interactively and type the word:

```bash
./solver
crane
```

---

## Output

### Hard Windmill

```
$ echo "acrid" | ./solver
HARD WINDMILL
cedar  YxYYY
cease  YxYxx
cardi  YYGYY
elder  xxYxY
cider  YYYxY
```

### Soft Windmill

```
$ echo "crane" | ./solver
SOFT WINDMILL
clean  GxYYY
adept  YxYxx
caner  GYYYY
abase  xxGxG
canoe  GYYxG
```

### Impossible

```
$ echo "abbey" | ./solver
IMPOSSIBLE
```

---

## Results Across All 2,315 Targets

| Result | Count |
|--------|-------|
| Hard Windmill | 130 |
| Soft Windmill | 1,050 |
| Impossible | 1,135 |

---

## How the Solver Works

### 1. Color function — NYT duplicate-letter rules

The `color(guess, target)` function replicates the exact NYT coloring algorithm:

**Pass 1 — Greens:** For each position `i`, if `guess[i] == target[i]`, mark green and consume that target letter.

**Pass 2 — Yellows/Blacks (left-to-right):** For each non-green position, scan the remaining unconsumed target letters. First match → yellow (consumed). No match → black.

This handles duplicate letters correctly. For example:
- Guess `"speed"`, target `"abide"` → `x x x G x` (only one `e` in target, consumed by the green at position 3)
- Guess `"llama"`, target `"hello"` → `x Y x x x` (only one `l` available after the green `l` at position 3 is consumed)

### 2. Candidate generation

For each of the 5 rows, the solver filters the full word list to words whose color result matches the row's requirement:

- **Hard search:** `color(word, target) == exact_pattern[row]`
- **Soft search:** black positions must score `0`; non-black positions must score `1` or `2`

Words scoring `GGGGG` are excluded from rows 1–4.

### 3. Distinct word selection

A depth-first backtracking search assigns one word per row, ensuring all five chosen words are distinct. The search exits immediately on the first valid assignment.

---

## License

Word lists are property of the New York Times. This solver is an independent tool for puzzle enthusiasts and is not affiliated with or endorsed by the NYT.
