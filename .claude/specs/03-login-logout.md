# Spec: Login and Logout

## Overview

Make the existing `/login` page functional and replace the `/logout` stub. Registration (step 2) creates users but nobody can sign in yet. This step adds POST handling on `/login` that verifies the email and password hash and stores the user in a Flask session, plus a `/logout` route that clears the session. Both the shared navbar (`base.html`) and the standalone landing page reflect auth state, so the signed-in state is visible. `/profile` is still a stub until step 4, so login redirects to the landing page; logout redirects to the login page. This covers the roadmap's "register and login" (login half) and step 3 (logout), and unblocks every user-scoped feature that follows.

## Depends on

- Step 1 — Database setup (`users` table, `get_db()`, `init_db()`)
- Step 2 — Registration (`create_user`, `get_user_by_email`, users with werkzeug password hashes)

## Routes

- `GET /login` — render the sign-in form (already exists); redirect to `/` if already logged in — public
- `POST /login` — validate the form, verify credentials, set the session and redirect to `/`; re-render the form with an error on failure — public
- `GET /logout` — clear the session and redirect to `/login` — public (harmless if not logged in)

## Database changes

No database changes. `get_user_by_email(email)` from step 2 already returns the row needed (`id`, `name`, `password_hash`).

## Templates

- Create: none
- Modify:
  - `expense-tracker/templates/login.html` — change `action="/login"` to `action="{{ url_for('login') }}"`; repopulate the `email` input after a failed submit (never the password).
  - `expense-tracker/templates/base.html` — in `.nav-links`, when `session.user_id` is set show the user's name (link to `profile`) and a "Sign out" link (`url_for('logout')`); otherwise keep "Sign in" and "Get started".
  - `expense-tracker/templates/landing.html` — this page does not extend `base.html`, so in the top-right, when `session.user_id` is set show an avatar with the user's initial, the user's name and a labelled "Sign out" button (`url_for('logout')`); otherwise show the "S" avatar and a labelled "Sign in" button. The state must be visibly different, because identical Sign in / Sign out icons made a successful logout look like it had not happened.

## Files to change

- `expense-tracker/app.py` — set `app.secret_key`; import `os`, `session`, `check_password_hash`; accept `GET` and `POST` on `/login`; implement `/logout` (remove it from the placeholder section).
- `expense-tracker/templates/login.html` — as above.
- `expense-tracker/templates/base.html` — as above.
- `expense-tracker/templates/landing.html` — as above.
- `expense-tracker/static/css/landing.css` — styles for the labelled button (`.icon-btn.labelled`) and the user name (`.auth-name`), using existing CSS variables.

## Files to create

- `.claude/specs/03-login-logout.md` — this spec (the only new file).

## New dependencies

No new dependencies.

## Rules for implementation

- No SQLAlchemy or ORMs
- Parameterised queries only
- Passwords verified with werkzeug (`check_password_hash`); never compare plaintext
- Use CSS variables — never hardcode hex values
- All templates extend `base.html` (except `landing.html`, which is already standalone)
- Secret key read from the `SECRET_KEY` environment variable, with a clearly-labelled dev-only fallback; never commit a real secret
- Email is stripped and lowercased before lookup
- Unknown email and wrong password must return the same generic error ("Invalid email or password.") with HTTP 401, so accounts cannot be enumerated
- Empty email or password re-renders the form with an error and HTTP 400
- On success call `session.clear()` first (avoids session fixation), then store only `session["user_id"]` and `session["user_name"]` (never the password hash); redirect (302) to `url_for('landing')`
- Logout uses `session.clear()` and redirects to `url_for('landing')`
- Do not change the `users` schema or any other route; `/profile` stays a stub until step 4
- When testing locally, run only one dev server on port 5001. On Windows several copies can bind the same port silently and requests are split between them, which looks like flaky login/logout.

## Definition of done

- [ ] `GET /login` renders the form with no errors when logged out
- [ ] Logging in as `demo@spendly.com` / `demo123` redirects to `/`
- [ ] A user created via `/register` can log in with their password
- [ ] Email matching is case-insensitive and ignores surrounding whitespace
- [ ] A wrong password and an unknown email both show "Invalid email or password." with status 401
- [ ] Empty email or password is rejected with an error
- [ ] After a failed login the email field keeps its value and the password field is empty
- [ ] The form posts to the URL produced by `url_for('login')`
- [ ] After login the landing page shows the user's initial, name and a "Sign out" button, and pages using `base.html` (e.g. `/terms`) show the user's name and "Sign out" instead of "Sign in" / "Get started"
- [ ] Visiting `/login` while logged in redirects to `/`
- [ ] A single click on Sign out clears the session and lands on `/login`; going back to `/` shows the "S" avatar and a "Sign in" button
- [ ] After logout, `session` no longer contains `user_id`
- [ ] The app starts without errors and no existing page is broken
