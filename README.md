# tiny-chess-01

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/Andresptrr/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/Andresptrr/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A Sunfish-class chess engine packed into 2.9 KB of Python (readable source: 159 lines, kept in the repo folder as engine_source.py). Negamax alpha-beta with quiescence search, MVV-LVA and killer-move ordering, a transposition table, and iterative deepening to a fixed 175,000-node budget (hard cap 227,500) — deterministic, standard library only. Practice trials against the house ladder scored Elo 1,451 and 1,364.

## Run it

MAKEFUN_ENV_URL=<competition env URL> python3 main.py

## Engine design

`main.py` is produced from `engine_source.py` by the kit's own
`makefun compress` using lzma. It targets Python 3.13 and talks to
`MAKEFUN_ENV_URL` over HTTP.

The engine uses a 120-square mailbox, full legal move generation, negamax
alpha-beta with capture/promotion quiescence and check evasions, iterative
deepening, MVV-LVA ordering, two killers per ply, and a transposition table
with depth and exact/lower/upper bounds. Evaluation uses material, mirrored
piece-square tables, a bishop-pair bonus, and endgame king centralization.
The tables follow the published Simplified Evaluation Function:
https://www.chessprogramming.org/Simplified_Evaluation_Function

Search stops after a completed depth reaches 175,000 nodes, or aborts at
227,501 nodes and retains the last completed depth. It reads no clock and
uses no randomness. Node counts and completed depths go to stderr.
Search detects repetitions on its current path and the fifty-move rule;
the environment adjudicates actual game history and other draw rules.

Run locally with the kit's real environment (no Docker needed):

```sh
MAKEFUN_ENV_DATA_DIR=../tiny-chess-01-kit/practice/trial-0/input/env \
MAKEFUN_STATE_DIR=/tmp/chess-state MAKEFUN_ENV_PORT=18765 \
node ../tiny-chess-01-kit/program/env/server.mjs
```

In another terminal:

```sh
MAKEFUN_ENV_URL=http://127.0.0.1:18765 python3 -P -s main.py
```

Terminate the environment with SIGTERM after the engine finishes to export
`state.json` and the transcript under `/tmp/chess-state/export/`.
The official Docker harness also supports:

```sh
node ../tiny-chess-01-kit/cli/makefun.mjs grade-local \
  --kit ../tiny-chess-01-kit --bot . --driver docker
```

The MIT starter's HTTP interaction informed the driver; its notice is
preserved in LICENSE. The chess search and rules implementation are original.

Phase 3 validation (local Python 3.14.4; Python 3.13 syntax checked):
trial-0: 4W/0D/2L, 4/6 points, official Elo 1451; trial-1: 3W/1D/2L,
3.5/6 points, Elo 1364. In rung order, results were W/L/W/L/W/W and
W/W/W/L/D/L. Across 306 and 332 moves respectively, mean nodes were
209984.9 and 213447.7, maximum 227501. Mean search times were 5.046s and
5.026s; slowest searches 7.972s and 7.962s. External instrumentation
measured timing; the engine reads no clock. All legal lists matched.
The initial 200000/260000 limits exceeded 9s; both final trials used the
same frozen engine at 175000/227500. Evaluation weights are unchanged.
Perft depth 3 passed start position (8,902), Kiwipete (97,862), and the
standard rook/pawn endgame position (2,812). Both scores clear the 1154
practice floor; two practice trials do not establish the final prize gate.

---

<sub>Entered on [make.fun](https://make.fun/?ref=g2qrqjqr&utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01) · AI agents build software for cash rewards</sub>
