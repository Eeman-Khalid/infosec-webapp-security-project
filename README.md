# Student Portal — Web Security Project (Blue Team Focus)

**Use case:** A university Student Portal — login, course search, course
reviews/comments, student profile, and a "change password" account
action. The same use case is implemented twice:

- `vulnerable/` — deliberately insecure build. Every attack below works
  end-to-end against it.
- `safe/` — the same app, hardened. Every attack below fails against it.

## Tech stack

- Node.js + Express (backend)
- EJS (server-rendered frontend — login, dashboard, search, course,
  profile, change-password pages)
- SQLite (`sqlite3` npm package) — a real, file-based, query-executing
  relational database. No separate DB server install needed.
- Vanilla JS for the DOM-based XSS demo (`public/client.js`)

> Note on SQL injection "advanced" stage: the assignment brief mentions
> `information_schema` as an example. This project uses SQLite, whose
> schema-introspection equivalent is `sqlite_master` (lists tables) and
> `pragma_table_info('table')` / the `sql` column of `sqlite_master`
> (lists columns) — functionally the same technique, just SQLite's name
> for it. This is explained again inline in `docs/PAYLOADS.md`.

## How to run both versions

Requires Node.js 18+ (check with `node -v`). Run the two apps in two
separate terminals — they use different ports so they can run at the
same time.

```bash
# Terminal 1 - vulnerable version (http://localhost:3000)
cd vulnerable
npm install
npm start

# Terminal 2 - safe version (http://localhost:4000)
cd safe
npm install
npm start
```

Each app recreates and reseeds its own `portal.db` SQLite file on every
restart, so the demo is always reproducible. Seeded accounts:

| username | password            | role    |
|----------|---------------------|---------|
| admin    | S3cr3tAdm1nP@ss      | admin   |
| alice    | alice123             | student |
| bob      | bobpass              | student |

## Implemented vulnerabilities / fixes

| # | Vulnerability | Where (vulnerable) | Fix (safe) |
|---|----------------|--------------------|------------|
| 1 | Reflected XSS | `/search?q=` echoed with `<%- %>` (unescaped) | Rendered with `<%= %>` (HTML-escaped) |
| 2 | Stored XSS | Course review `comment` rendered with `<%- %>` | Rendered with `<%= %>` |
| 3 | DOM-based XSS | `client.js` writes `location.hash` via `innerHTML` | `client.js` uses `textContent` |
| 4 | CSRF | `/do-change-password` accepts GET, no token check | POST-only, per-session CSRF token required |
| 5 | SQLi — Easy | Login query built by string concatenation → `' OR '1'='1' --` bypass | Parameterized query (`?` placeholders) |
| 6 | SQLi — Intermediate | `/api/check-user?cond=` injects a raw SQL fragment → blind boolean inference | Endpoint removed entirely (returns 404) |
| 7 | SQLi — Advanced | `/search?q=` built by concatenation → UNION chain (columns → tables → columns → data) | Parameterized query |

Full payload list and expected results: see `docs/PAYLOADS.md`.

## Project structure

```
secportal/
  vulnerable/        # insecure build, port 3000
  safe/              # hardened build, port 4000
  docs/
    PAYLOADS.md       # every payload + what to screenshot
  README.md
```
