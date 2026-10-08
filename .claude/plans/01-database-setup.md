# Plan: Step 1 — Database Setup

## Context

`database/db.py` is only a comment stub, so the app has no data layer. Every later step (auth, profile, expenses) needs it. The spec is `.claude/spec/01-database-setup.md` (the folder is `spec/`, not `specs/`). It asks for SQLite via `sqlite3` with no ORM, parameterized SQL only, and no new dependencies. Only two files change: `database/db.py` and `app.py`. No routes change.

## Changes

### `expense-tracker/database/db.py` (replace the stub)

- Imports: `sqlite3`, `os`, `datetime`, `generate_password_hash` from `werkzeug.security`.
- `DB_PATH`: `expense_tracker.db` in the app directory (parent of `database/`), built with `os.path`. This name is already in `.gitignore`, and the spec allows it.
- `get_db()`: `sqlite3.connect(DB_PATH)`, set `row_factory = sqlite3.Row`, run `PRAGMA foreign_keys = ON` on every connection, return it.
- `init_db()`: `CREATE TABLE IF NOT EXISTS` for both tables, exactly per spec section 4.
  - `users`: `id` INTEGER PRIMARY KEY AUTOINCREMENT, `name` TEXT NOT NULL, `email` TEXT NOT NULL UNIQUE, `password_hash` TEXT NOT NULL, `created_at` TEXT DEFAULT (datetime('now')).
  - `expenses`: `id` PK AUTOINCREMENT, `user_id` INTEGER NOT NULL REFERENCES users(id), `amount` REAL NOT NULL, `category` TEXT NOT NULL, `date` TEXT NOT NULL, `description` TEXT, `created_at` TEXT DEFAULT (datetime('now')).
  - Commit, close the connection in a `try/finally`.
- `seed_db()`:
  - Return early if `SELECT COUNT(*) FROM users` is greater than 0.
  - Insert the demo user (Demo User / demo@spendly.com / `generate_password_hash("demo123")`).
  - Insert 8 expenses with `executemany` and `?` placeholders. They cover all 7 fixed categories (Food, Transport, Bills, Health, Entertainment, Shopping, Other), so one category appears twice. Dates fall on days 1–28 of the current month, built with `date.today().replace(day=N)` and formatted `YYYY-MM-DD`.
  - Get the demo user's id from `cursor.lastrowid`.

### `expense-tracker/app.py`

- Add `from database.db import get_db, init_db, seed_db`. `get_db` is imported per the spec, although unused here.
- After `app = Flask(__name__)`, add:

  ```python
  with app.app_context():
      init_db()
      seed_db()
  ```

  This is module-level, so the DB is ready before any route is served, with `python app.py` or any other import.
- The routes stay unchanged.

## Notes

- `database/__init__.py` already exists, so `from database.db import ...` works when run from the app directory (`python app.py`).
- The DB file is created in the app directory and is already gitignored.
- Errors: duplicate email and bad `user_id` raise `sqlite3.IntegrityError` naturally. Nothing swallows exceptions.

## Verification

Run from `D:\Claude\First_Porject\expense-tracker\expense-tracker`:

1. `python app.py` starts without errors, and `expense_tracker.db` appears.
2. A short script using `get_db()` checks:
   - Both tables exist with the right columns (`PRAGMA table_info`).
   - Exactly 1 user and 8 expenses, covering all 7 categories, with dates in the current month in `YYYY-MM-DD` form.
   - The stored password hash verifies with `check_password_hash(hash, "demo123")`.
3. Restart the app and re-run `init_db()` and `seed_db()`. Counts stay at 1 user and 8 expenses.
4. Check the constraints:
   - Inserting a duplicate email raises `IntegrityError`.
   - Inserting an expense with `user_id=999` raises `IntegrityError`.
5. Visit `/` and `/login` and confirm they still render.

## After approval

Copy this plan to `.claude/plans/01-database-setup.md` as requested. Plan mode only allowed me to write this file, so the copy is deferred. That folder is gitignored via `.claude/plans/` in `.gitignore`.
