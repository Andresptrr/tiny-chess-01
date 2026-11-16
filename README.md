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

Search stops after a completed depth reaches 25,000 nodes, or aborts at
50,001 nodes and retains the last completed depth. It reads no clock and
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

Trial-0 validation (local Python 3.14.4; Python 3.13 syntax checked):
0 wins, 2 draws, 4 losses; official scorer Elo 876. Draws against depth-2
and nodes-250k; all losses by checkmate. Across 309 moves, nodes averaged
41,752.89 (range 6--50,001), search time averaged 2.424 seconds and peaked
at 4.990 seconds. External instrumentation measured timing; no engine
clock. Every generated legal move list matched the real environment.
Perft depth 3 passed start position (8,902), Kiwipete (97,862), and the
standard rook/pawn endgame position (2,812). This version is below the
Sunfish baseline and the 1,154 practice qualification floor.

To inspect the packed source without running it:

```python
import ast, base64, pathlib, zlib
tree = ast.parse(pathlib.Path('main.py').read_text())
payload = tree.body[1].value.args[0].args[0].args[0].value
print(zlib.decompress(base64.b85decode(payload)).decode())
```
