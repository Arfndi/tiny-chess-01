# tiny-chess-01 — a 4 KB chess engine

A Python 3.13 standard-library-only chess engine in at most 4,096 bytes
(`main.py`, produced by `makefun compress` from the readable `engine.py`).
It plays the house gauntlet through the env at `MAKEFUN_ENV_URL`.

## How it plays

- 0x88 board, alpha-beta search with a transposition table.
- Move ordering: transposition-table move first, then MVV-LVA captures,
  killer moves, then history.
- Quiescence search over captures and promotions, and over every reply
  while in check.
- Evaluation: material (pawn 100, knight 320, bishop 330, rook 500,
  queen 900) plus formula piece-square terms, a bishop-pair bonus, and
  a tapered king table (centralising in the endgame).
- Fixed node budget of 80,000 nodes per move, deepened one ply at a
  time; a depth that would pass the budget is dropped whole and the
  last whole depth's move stands. No clock in the search, no unseeded
  randomness — the same position always gives the same move.

## The driver

`main.py` loops on the env: `POST /game` for the position in play,
searches the FEN it is served, and answers with `POST /move` in UCI
(`e2e4`, `e7e8q`; castling is the king's move, `e1g1`). It never
resigns and never calls any other service. It exits when the env
answers `{"status": "done"}`.

## Run it

```
MAKEFUN_ENV_URL=http://127.0.0.1:8080 python3 main.py
```

## Build it

```
makefun compress engine.py --out main.py
```

`main.py` is 3,076 bytes as committed; the readable source is
`engine.py` in this folder.
