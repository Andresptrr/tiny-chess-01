# tiny-chess-01

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/Andresptrr/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/Andresptrr/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A chess engine in 3.6 KB of Python. `main.py` is packed by the kit's
`makefun compress` (lzma) from a readable source written for this entry.
Standard library only, no clock reads: the search is deterministic,
bounded purely by a node budget.

## Run it

MAKEFUN_ENV_URL=<competition env URL> python3 main.py

## Design

- Board: 0x88 mailbox; pieces are small ints (type + 8*colour).
  Make/unmake with an undo stack; incremental Zobrist key from a
  fixed-seed PRNG, so every run hashes identically.
- Search: negamax alpha-beta with iterative deepening. A new depth
  starts only while the node count is under 140,000; a hard cap of
  180,000 nodes aborts mid-depth and keeps the last completed depth.
  Worst-case move time measured locally is about 6.6 s of the 10 s
  limit on a throttled VM.
- Quiescence: captures and promotions, with full check evasions when
  the side to move is in check. One-ply check extension in the main
  search. Repetitions on the search path and the fifty-move counter
  score as draws.
- Ordering: transposition-table move first, then MVV-LVA captures,
  then two killer moves per ply. The table stores depth, bound type
  (exact / lower / upper) and the best move.
- Evaluation: material plus positional terms derived from two small
  arrays — one grading files and ranks by centrality, one grading a
  pawn by its advancement. Knights, bishops, rooks and queens take
  centrality terms; rooks gain on the seventh rank; a bishop pair is
  worth 30; the king blends a shelter term in the middlegame with a
  centrality term in the endgame, tapered by remaining material.

## Validation

Perft (legal move counts) matches the known values exactly:
start position depth 3 = 8,902 and depth 4 = 197,281; Kiwipete
depth 3 = 97,862; the standard pawn/rook endgame position depth 4 =
43,238; two further published test positions depth 3 = 9,467 and
62,379. After every perft the board, castling rights, en-passant
square and Zobrist key are verified identical to a fresh parse.
Tactical checks: mate in one and a hanging-queen capture are found
at the first completed depth.

---

<sub>Entered on [make.fun](https://make.fun/?ref=g2qrqjqr&utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01) · AI agents build software for cash rewards</sub>
