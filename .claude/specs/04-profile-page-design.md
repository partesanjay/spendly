# Spec: Profile Page Design

## Overview

Replace the `/profile` placeholder (currently a plain string, "coming in Step 4") with a designed profile page for the signed-in user. The page shows who the user is (avatar with initial, name, email, member-since date) and a small spending snapshot drawn from their own expenses (total spent, number of expenses, top category) plus their most recent expenses. Login (step 3) already stores `user_id` and `user_name` in the session and the navbar already links the user's name to `profile`, so this step makes that link land on a real page. It also introduces the first login-required guard, which later user-scoped steps (expense list, add/edit/delete) will reuse.

## Depends on

- Step 1 — Database setup (`users` and `expenses` tables, `get_db()`)
- Step 2 — Registration (users exist with hashed passwords)
- Step 3 — Login and Logout (`session["user_id"]`, `session["user_name"]`, navbar links to `profile`)

## Routes

- `GET /profile` — render the signed-in user's profile page; redirect to `/login` if there is no session — logged-in

No other new routes. The `/profile` placeholder in `app.py` is replaced, not duplicated.

## Database changes

No database changes. The existing `users` (`id`, `name`, `email`, `password_hash`, `created_at`) and `expenses` (`id`, `user_id`, `amount`, `category`, `date`, `description`, `created_at`) tables in `database/db.py` already hold everything the page needs. Two new read-only query functions are added (see Files to change); no schema edits.

## Templates

- Create:
  - `expense-tracker/templates/profile.html` — extends `base.html`; profile header card (avatar initial, name, email, "Member since"), stats row (total spent in ₹, expense count, top category), and a "Recent expenses" list/table with an empty state when the user has none.
- Modify:
  - `expense-tracker/templates/base.html` — only if needed to mark the profile link as active; otherwise unchanged.

## Files to change

- `expense-tracker/app.py` — replace the `/profile` stub with a real view: redirect to `url_for('login')` when `session.get("user_id")` is missing, load the user and summary data, `render_template("profile.html", ...)`. If the session user no longer exists in the database, clear the session and redirect to `/login`.
- `expense-tracker/database/db.py` — add `get_user_by_id(user_id)` and `get_expense_summary(user_id)` (returns total, count, top category) and `get_recent_expenses(user_id, limit=5)`; all parameterised.
- `expense-tracker/static/css/style.css` — profile page styles (card, avatar, stats grid, recent list, empty state, responsive stacking on narrow screens).
- `expense-tracker/ROUTES.md` and `expense-tracker/CLAUDE.md` — move `/profile` from placeholders to implemented; note the login guard.
- `expense-tracker/ROADMAP.md` — mark step 4 done.

## Files to create

- `expense-tracker/templates/profile.html`
- `.claude/specs/04-profile-page-design.md` — this spec

## New dependencies

No new dependencies.

## Rules for implementation

- No SQLAlchemy or ORMs
- Parameterised queries only
- Passwords hashed with werkzeug; `password_hash` must never be passed to the template or shown
- Use CSS variables — never hardcode hex values (reuse `--accent`, `--paper-card`, `--border`, `--ink-muted`, `--radius-*` etc. from `style.css`)
- All templates extend `base.html`
- Every query is scoped by `user_id` from the session; never accept a user id from the URL or query string
- Amounts are displayed in rupees (₹) with two decimals
- Jinja autoescaping must stay on; do not use `|safe` on user-supplied fields (name, email, description)
- Layout must work on mobile widths (single column below ~600px)
- Do not change the schema, the login/logout/register behaviour, or any other placeholder route
- Run only one dev server on port 5001 when testing (see CLAUDE.md)

## Definition of done

- [ ] Visiting `/profile` while logged out redirects to `/login`
- [ ] Logging in as `demo@spendly.com` / `demo123` and clicking the name in the navbar opens a styled profile page, not a plain string
- [ ] The page shows the user's avatar initial, name, email and member-since date
- [ ] The stats show the correct total spent (₹), expense count and top category for the demo user's seeded expenses
- [ ] "Recent expenses" lists at most 5 expenses, newest first, with date, category, description and amount
- [ ] A newly registered user with no expenses sees ₹0.00, a count of 0 and an empty-state message instead of an error
- [ ] One user never sees another user's expenses
- [ ] The password hash does not appear anywhere in the page source
- [ ] The page matches the existing palette and fonts and is usable at phone width (no horizontal scroll)
- [ ] If the logged-in user's row is deleted, visiting `/profile` clears the session and redirects to `/login`
- [ ] Other pages (`/`, `/login`, `/register`, `/terms`, `/privacy`) still work and the app starts without errors
