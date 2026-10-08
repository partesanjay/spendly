# Spec: Registration

## Overview

Make the existing `/register` page functional. Today `/register` only renders the form; this step adds POST handling that validates input, hashes the password with werkzeug and inserts a new row into the `users` table. It is the first half of the roadmap's step 2 (register and login) and unblocks login, logout and every user-scoped feature that follows. Login and sessions are out of scope here.

## Depends on

- Step 1 — Database setup (`users` table, `get_db()`, `init_db()`)

## Routes

- `GET /register` — render the registration form (already exists, unchanged) — public
- `POST /register` — validate the form, create the user, redirect to `/login` on success; re-render the form with an error on failure — public

## Database changes

No database changes. The `users` table in `database/db.py` already has `name`, `email` (UNIQUE), `password_hash` and `created_at`.

Add two helper functions to `database/db.py` (no schema impact):

- `create_user(name, email, password_hash)` — inserts a row with a parameterised query and returns the new user id. Lets the `sqlite3.IntegrityError` from the UNIQUE email constraint propagate.
- `get_user_by_email(email)` — returns the matching `sqlite3.Row` or `None`.

## Templates

- Create: none
- Modify:
  - `expense-tracker/templates/register.html` — change `action="/register"` to `action="{{ url_for('register') }}"`, repopulate `name` and `email` inputs after a failed submit (never the password), and add `minlength="8"` to the password input.

## Files to change

- `expense-tracker/app.py` — accept `GET` and `POST` on `/register`; import `request`, `redirect`, `url_for`, `generate_password_hash`, and the new db helpers.
- `expense-tracker/database/db.py` — add `create_user` and `get_user_by_email`.
- `expense-tracker/templates/register.html` — as above.

## Files to create

- `.claude/specs/02-registration.md` — this spec (the only new file).

## New dependencies

No new dependencies.

## Rules for implementation

- No SQLAlchemy or ORMs
- Parameterised queries only
- Passwords hashed with werkzeug (`generate_password_hash`)
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- Validation, all server-side, in this order; the first failure re-renders the form with a single `error` message and HTTP 400:
  - `name` and `email` are non-empty after `strip()`
  - `email` is lowercased and stripped before checking and storing, and contains `@`
  - `password` is at least 8 characters
  - email is not already registered (check with `get_user_by_email`, and also catch `sqlite3.IntegrityError` to cover a race)
- Never store or echo the plaintext password
- On success redirect to `url_for('login')` (302). Do not create a session — login is the next step.
- Do not change the `users` schema or any other route

## Definition of done

- [ ] `GET /register` still renders the form with no errors
- [ ] Submitting a valid name, new email and 8+ character password redirects to `/login`
- [ ] The new user exists in `expense_tracker.db` and `password_hash` is a werkzeug hash, not the plaintext password
- [ ] Registering the same email again (also with different casing) re-renders the form with an "already registered" error and creates no second row
- [ ] Registering `demo@spendly.com` is rejected as already registered
- [ ] A password shorter than 8 characters is rejected with an error and no row is created
- [ ] Empty or whitespace-only name or email is rejected with an error
- [ ] After a failed submit the name and email fields keep their values and the password field is empty
- [ ] The form posts to the URL produced by `url_for('register')`
- [ ] The app starts without errors and no existing page is broken
