# Roadmap

Step numbers come from the placeholder routes in `app.py` and the comment in `database/db.py`. Steps 2, 5 and 6 are not named in the code, so their contents below are proposals.

## Done

- Landing page, register/login/terms/privacy pages (templates only)
- Shared base layout and styling (`base.html`, `style.css`, `landing.css`)

## Planned (from the code)

| Step | Goal | Details |
|------|------|---------|
| 1 | Database setup | `get_db()`, `init_db()`, `seed_db()` in `database/db.py` (SQLite, foreign keys on) |
| 2 *(proposed)* | Register and login | POST handling, password hashing with Werkzeug, session cookie |
| 3 | Logout | Clear the session, redirect to landing |
| 4 | Profile page | Show the signed-in user's details |
| 5 *(proposed)* | Expense list / dashboard | List the user's expenses, totals |
| 6 *(proposed)* | Categories and filters | Category per expense, filter by date and category |
| 7 | Add expense | Replace the `/expenses/add` stub with a form |
| 8 | Edit expense | Replace the `/expenses/<id>/edit` stub |
| 9 | Delete expense | Replace the `/expenses/<id>/delete` stub |

## Ideas beyond the steps

- Change delete (and add/edit submit) to POST, since a GET delete link can be triggered by a prefetch or a crawler
- Login-required guard on profile and expense routes; ownership check on edit/delete
- Form validation and flash messages
- Monthly summary and charts
- CSV export
- Tests with `pytest-flask` (already in `requirements.txt`, no tests yet)
- Move the secret key and config out of code (`.env` is already gitignored)
