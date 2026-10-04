# Flappy Clone

A browser Flappy Bird clone with a global leaderboard.

- **Frontend:** HTML5 canvas and plain JavaScript, with no build step.
- **Backend:** FastAPI with SQLite. It serves the game and stores scores.

## Run

```bash
cd backend
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
uvicorn main:app --reload
```

Open http://localhost:8000. Interactive API docs are at http://localhost:8000/docs.

## Controls

| Action | Input |
|---|---|
| Start / flap / restart | Space, mouse click, or tap |

When a game ends with a score above 0, a name box appears so you can submit the
score. Your personal best is kept in the browser's localStorage.

## API

| Method | Path | Body / query | Response |
|---|---|---|---|
| GET | `/api/health` | none | `{"status": "ok"}` |
| GET | `/api/scores` | `?limit=1..100` (default 10) | Top scores, highest first; ties go to the earlier entry |
| POST | `/api/scores` | `{"name": "ann", "score": 12}` | `201` with the stored row |

Validation rules:

- `name`: 1–20 characters from letters, digits, space, `_`, `.` and `-`. Surrounding spaces are trimmed.
- `score`: an integer from 0 to 10000.

Invalid input returns `422`.

Example:

```bash
curl -X POST localhost:8000/api/scores -H 'Content-Type: application/json' \
     -d '{"name":"ann","score":12}'
curl 'localhost:8000/api/scores?limit=5'
```

## Configuration

| Variable | Default | Meaning |
|---|---|---|
| `DB_PATH` | `scores.db` | SQLite file location |

## Tests

```bash
cd backend
pytest
```

## How the game works

- **Time-based physics.** Every frame updates velocity and position using the
  real time elapsed (`dt`), so the game runs at the same speed at 60 Hz and 144 Hz.
  `dt` is capped at 1/30 s, so returning to a background tab doesn't jump the bird forward.
- **Pipes** spawn every 1.5 s with a random gap position and scroll left at a fixed speed.
- **Collision** treats the bird as a circle and each pipe as a rectangle.
- **Scoring** adds a point when a pipe's right edge passes the bird.

All tuning constants (`GRAVITY`, `FLAP_VELOCITY`, `PIPE_SPEED`, `PIPE_GAP`, …) are
at the top of `frontend/game.js`.

## Limitations

- **Scores can be faked.** The browser reports the score and the server accepts
  any value in range. Preventing that would need server-side game replay or
  input verification, which isn't implemented.
- **No rate limiting or accounts.** Put a reverse proxy with rate limits in front
  of the server before exposing it publicly.
- **SQLite** is fine for a single server, not for many concurrent writers.
