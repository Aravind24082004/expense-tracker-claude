# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Expense tracker API: Express 4, Node 18+, one dependency (`express`). No build or lint step.

## Commands

```
npm install
npm test                                           # node --test, every file in test/
node --test test/export.test.js                    # one file
node --test --test-name-pattern="Content-Disposition" test/export.test.js   # one test by name
npm start                                          # http://localhost:3000 (PORT overrides)
```

## Architecture

- `src/app.js` builds and exports the app with every route; `src/server.js` only calls `listen`. Tests require `app.js` directly.
- New routes go above the `// --- routes end ---` marker in `src/app.js`, or in a new module that `app.js` requires.
- Reuse `isValidDate` (strict `YYYY-MM-DD`; rejects impossible dates and arrays from repeated query params) and `isValidMonth`. Errors are `400` with JSON `{ "error": "<reason>" }`.
- `src/store.js` is a synchronous JSON-file store: reads `data/expenses.json` on every call, rewrites it on every `add`. Ids are `e<n>` (next = max + 1). Dates are `YYYY-MM-DD` strings, filtered by string comparison.
- `public/index.html` is a vanilla-JS page served statically by the same app.

## Tests

- `node:test` + `node:assert` with raw `http` requests against `app.listen(0)`. No Jest or supertest; each file has its own `request()` helper. Keep that style.
- Set `EXPENSES_FILE` to a temp copy of `data/expenses.json` *before* requiring `src/app`, because `store.js` reads the path when it loads.
- Assertions are tied to the seed data (60 rows, 2026-08-01 to 2026-10-20, 18 in September, September total 29257). Don't edit `data/expenses.json`.
- Tests in a file share one temp file, and some POST rows that later tests depend on (e.g. next id `e61`), so order matters. Use dates outside the seed range for added rows.

## Feature briefs

`FEATURE.md` (CSV export, built) and `FEATURE-2.md` (stretch: `/expenses/summary` with `from`/`to`) hold the specs and out-of-scope lists. Read the relevant one before touching those routes.
