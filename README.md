# Polgár 5334 — Chess Puzzle Trainer

A web app that loads all 4,462 mate puzzles from László Polgár's
*Chess: 5334 Problems, Combinations and Games* (the mate-in-1, mate-in-2,
and mate-in-3 sections — the 600-game miniatures section is full games,
not standalone puzzles).

## Files

```
index.html         The app — open this in a browser
puzzles_data.js    All 4,462 puzzles bundled as a JS variable
polgar_puzzles.json  Same data as JSON (for reuse in other tools)
```

You can also share just the JSON; it's clean data with FEN, side-to-move,
the book's first move, and the full main-line variation for every puzzle.

## How to run it

The app needs to be served over HTTP — opening `index.html` directly with
`file://` will mostly work, but Stockfish loading from the CDN will fail
because of browser security rules. Two easy options:

**Option 1 — Run a local server (recommended):**

```bash
cd path/to/polgar_chess/app
python3 -m http.server 8000
```

Then open http://localhost:8000 in your browser.

**Option 2 — Open directly:**

Open `index.html`. Everything works except the Stockfish engine; the app
falls back to a pure-JavaScript mate solver, which is slower (1-3 seconds
on hard mate-in-3 positions) but still correct.

## How it works

- The puzzles were extracted directly from the LaTeX `skak` chess font
  in the original PDF — no OCR, near-perfect accuracy. 4,461 of 4,462
  puzzles validated with python-chess; the one outlier is a printing
  error in the source book itself.
- The app uses **chessboard.js** for the board, **chess.js** for move
  validation, and **Stockfish.js** (loaded via the Blob-Worker trick)
  for engine verification.
- When the user makes a move, the app first checks if it delivers
  checkmate (chess.js). If not and the puzzle is mate-in-2 or mate-in-3,
  Stockfish verifies whether the move starts a forced mate of the
  required length. The system accepts **any** move that forces mate,
  not just the book's specified first move — important for mate-in-2
  puzzles that often have multiple correct first moves.

## For your daughter

Keyboard shortcuts:
- `←` / `→` — previous / next puzzle
- `R` — random puzzle
- `H` — hint (highlights the key piece)

Progress (which puzzles she's solved) is saved in the browser, so her
progress will persist across sessions on the same browser.

## First-attempt rule (the strict mode)

To prevent guessing, the app uses a **first-attempt-counts** rule:

- If she gets the **first move right** on her first try → puzzle counts as **solved**, badge appears next to puzzle number.
- If she gets the **first move wrong** on her first try → puzzle is added to her **Wrong Book** permanently (until she explicitly removes it).
- Clicking **"Show solution"** before solving also adds the puzzle to the Wrong Book (no free skipping).
- Even if she gets it right on a later attempt, the puzzle stays in the Wrong Book — she can still practice and complete it, but the "first try" record is locked.

## The Wrong Book (错题本)

Pick **"📕 Wrong Book — review"** from the Section dropdown to see only the puzzles she's missed. This is her review list.

When she's confident she's mastered a puzzle in the wrong book, she can click the **"Remove"** button next to the "in wrong book" badge to take it out of review rotation.

The progress bar shows two numbers: how many she's solved on the first try, and how many are currently in her Wrong Book.

## Reusing the data

`polgar_puzzles.json` has 4,462 entries like:

```json
{
  "id": 1,
  "section": "Mate in 1",
  "mate_in": 1,
  "fen": "3q1rk1/5pbp/5Qp1/8/8/2B5/5PPP/6K1 w - - 0 1",
  "side": "w",
  "first_move": "Qxg7#",
  "main_line": ["Qxg7#"],
  "solution": "1.QXg7m",
  "page": 16
}
```

This is clean, validated data you can load into any chess UI or training
system. The compressed runtime version `puzzles_data.js` uses shorter
field names (`i`, `m`, `f`, `s`, `fm`, `ml`) to save bytes.
