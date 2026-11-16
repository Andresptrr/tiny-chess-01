# Tiny chess

Self-contained Python 3.13 chess engine for make.fun tiny-chess-01.
`main.py` contains a zlib-compressed, base85-encoded Python source payload.
It uses only the standard library and talks to `MAKEFUN_ENV_URL` over HTTP.

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

To inspect the packed source without running it:

```python
import ast, base64, pathlib, zlib
tree = ast.parse(pathlib.Path('main.py').read_text())
payload = tree.body[1].value.args[0].args[0].args[0].value
print(zlib.decompress(base64.b85decode(payload)).decode())
```
