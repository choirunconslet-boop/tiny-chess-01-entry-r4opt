# Tiny Chess — round 4 entry (round-3 engine, speed-optimized)

A 4 KB chess engine for the make.fun `tiny-chess-01` bounty.
`main.py` is the whole engine: pure Python 3.13 stdlib, no dependencies.

## What it is

This entry is built on the round-3 engine (a sunfish-style 120-char board,
piece-square tables, alpha-beta with aspiration windows, transposition table,
null-move pruning, LMR, futility pruning, iterative deepening) — the version
that scored Elo 1,364 locally in the practice gauntlet. Round 3's live
failure was speed (a 60k-node move took 6.5–8.5 s against a ~10 s limit),
so this round optimizes for speed while keeping the same search and the
same moves:

- Move generation precomputes each move's eval delta inline (base PST +
  capture + king-proximity + promotion + en passant), so `move()` never
  re-evaluates from scratch.
- The ray loop is a plain `while` instead of `itertools.count` (a
  per-direction iterator object was a large fraction of move-gen time).
- Per-piece PST row is hoisted out of the inner loop; move ordering is a
  list comprehension over precomputed values.
- Node cap lowered 60,000 → 30,000: same depth (6–7) on the bench
  positions, roughly half the time per move (~1.8 s vs ~4.5 s).

## Verification (local, before submission)

- Move-gen equivalence vs the round-3 source: 1,800 random positions,
  53,299 moves — identical move sets, order, and eval deltas.
- Perft: startpos d4 197281, Kiwipete d3 97862, en-passant position d4
  43238, promotion position d3 9467 — all exact.
- Eval consistency: incremental score matches a from-scratch evaluation
  after every move over 100 random games (0 drifts).
- Practice trial-0 (6 games vs the house gauntlet) results are in the
  trial log; the entry was only submitted after the local Elo cleared the
  1,154 gate.

## Layout

- `main.py` — the engine (LZMA-compressed source, unpacks at import).
- `README.md`, `LICENSE` — this file and the MIT license.
