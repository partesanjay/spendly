# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository layout

The git root is `D:\Claude\First_Porject\expense-tracker`, but the app lives in the nested `expense-tracker/` directory (run everything from there). The sibling `venv/` (Windows virtualenv, `venv\Scripts\Activate.ps1`) and `__MACOSX/` are not part of the app.

## Commands

```
pip install -r requirements.txt
python app.py            # dev server, debug mode, http://localhost:5001
pytest                   # pytest + pytest-flask are in requirements, but no tests exist yet
pytest path/to/test_file.py::test_name   # single test
```

No linter or build step is configured.

On Windows, several `python app.py` copies can bind port 5001 at once without any error, and requests are split between them (stale code, flaky login/logout). Stop the old server before starting a new one.

Git: the default branch is `master` (not `main`), and `origin` is `https://github.com/partesanjay/spendly.git`. Work is done on feature branches (e.g. `feature/database-setup`).

See `ROUTES.md` for the route map and `ROADMAP.md` for the planned tutorial steps.

## Architecture

"Spendly" is a Flask expense tracker built as a step-by-step teaching project, so much of it is intentionally unimplemented:

- `app.py` — single-file Flask app with all routes. Working routes: `/`, `/register`, `/login`, `/logout`, `/terms`, `/privacy`. Routes under "Placeholder routes" (`/profile`, `/expenses/add`, `/expenses/<id>/edit`, `/expenses/<id>/delete`) return stub strings labelled with the tutorial step that should implement them. `/register` handles POST (validates, hashes with werkzeug, inserts via `create_user`). `/login` handles POST (`check_password_hash`, then `session["user_id"]`/`session["user_name"]`) and redirects to `/`; `/logout` clears the session and redirects to `/`. Sessions use `app.secret_key` from the `SECRET_KEY` env var (insecure dev fallback). There is no login-required guard yet.
- `database/db.py` — contains only comments specifying the intended API: `get_db()` (SQLite connection with `row_factory` and foreign keys enabled), `init_db()` (`CREATE TABLE IF NOT EXISTS`), `seed_db()` (sample data). The SQLite file `expense_tracker.db` is gitignored.
- `templates/` — `base.html` provides navbar, footer and `{% block title/head/content/scripts %}`; `login`, `register`, `terms` and `privacy` extend it. `landing.html` is a standalone page (own sidebar layout, loads `landing.css`) and does not extend `base.html`, so auth state (Sign in / Sign out) is rendered separately in both `base.html` and `landing.html`.
- `static/css/style.css` — shared styles driven by CSS variables in `:root` (ink/paper/accent palette); `landing.css` is landing-only. `static/js/main.js` is an empty placeholder.

Use `url_for(...)` with the route function names (`landing`, `login`, `register`, `terms`, `privacy`) when linking between pages. The currency theme is rupees.

