# Route Map

All routes are defined in `app.py`. Dev server: `python app.py` → http://localhost:5001

## Implemented

| Method | Path        | Function   | Template        | Notes                                   |
|--------|-------------|------------|-----------------|-----------------------------------------|
| GET    | `/`         | `landing`  | `landing.html`  | Standalone page (does not extend base)  |
| GET    | `/register` | `register` | `register.html` | Form display only, no submit handling   |
| GET/POST | `/login`  | `login`    | `login.html`    | Verifies password, sets session, to `/` |
| GET    | `/logout`   | `logout`   | none            | Clears session, redirects to `/login`  |
| GET    | `/terms`    | `terms`    | `terms.html`    | Terms and Conditions                    |
| GET    | `/privacy`  | `privacy`  | `privacy.html`  | Privacy Policy                          |

## Placeholders (return a plain string)

| Method | Path                       | Function       | Planned in |
|--------|----------------------------|----------------|------------|
| GET    | `/profile`                 | `profile`      | Step 4     |
| GET    | `/expenses/add`            | `add_expense`  | Step 7     |
| GET    | `/expenses/<int:id>/edit`  | `edit_expense` | Step 8     |
| GET    | `/expenses/<int:id>/delete`| `delete_expense` | Step 9   |

## Navigation

```
/ (landing) ──┬── /login ───── /register
              ├── /register
              └── footer (via base.html): /terms, /privacy
```

`login`, `register`, `terms` and `privacy` extend `base.html`, which links to `landing`, `login`, `register`, `terms` and `privacy` via `url_for`.
