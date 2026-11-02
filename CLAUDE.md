# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Property search app over MLS (RETS) listing data: an Express + MySQL API in `backend/` and a Create React App frontend (React 19, React Router 7, Tailwind 3) in `frontend/`. The two are separate npm packages; there is no root `package.json`.

## Commands

Backend (`cd backend`):
- `npm run dev` — start with nodemon (`src/index.js`)
- `npm start` — start without reload
- No tests are configured for the backend.

Frontend (`cd frontend`):
- `npm start` — CRA dev server (port 3000)
- `npm run build` — production build
- `npm test` — Jest in watch mode via react-scripts
- Run once / single file: `CI=true npm test -- src/test/Pagination.test.js` (or `-t "test name"` to filter by name)

Linting is CRA's built-in ESLint (`react-app` config), surfaced through `npm start`/`npm run build`.

## Environment

- `backend/.env` (gitignored, not committed): `PORT`, `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`. The server reads `PORT` with no default, so it must be set — use `5000` to match the frontend proxy.
- `frontend/.env` (see `.env.example`): `REACT_APP_GOOGLE_MAPS_API_KEY` for the Maps Embed iframe in `PropertyMap`.
- Database: MySQL/MariaDB with tables `rets_property` and `rets_openhouse`. Dumps `rets_property.sql` (~600MB) and `rets_openhouse.sql` live at the repo root and are gitignored (`*.sql`); don't read them whole — grep the `CREATE TABLE` section for column meanings.

## Architecture

**Request flow:** the frontend calls relative `/api/...` URLs (`API_BASE = ''` in `frontend/src/api/client.js`); CRA's `"proxy": "http://localhost:5000"` in `frontend/package.json` forwards them to Express. All API calls go through `client.js`.

**Backend API** (`backend/src/routes/properties.js`, mounted at `/api/properties`; plus `GET /api/health`):
- `GET /` — filters `city`, `zipcode`, `minPrice`, `maxPrice`, `beds`, `baths`; pagination via `limit` (1–100, default 20) and `offset`. Returns `{ total, limit, offset, results }`. Runs a `COUNT(*)` query and a data query sharing the same parameterized `WHERE` clause.
- `GET /:id` — single property by `L_ListingID`.
- `GET /:id/openhouses` — returns `{ propertyId, count, openhouses }`.
- `/:id/openhouses` is declared before `/:id`; keep that ordering when adding routes.
- DB access is a shared `mysql2/promise` pool (`backend/src/mysql.js`); always use `?` placeholders.

**RETS column names are cryptic** — rows are returned raw (`SELECT *`) and the frontend reads these fields directly:
| Column | Meaning |
|---|---|
| `L_ListingID` | listing ID used in URLs, favorites, and API lookups |
| `L_SystemPrice` | price |
| `L_Keyword2` | bedrooms |
| `LM_Dec_3` | bathrooms |
| `LM_Int2_3` | approx. finished sqft |
| `L_Keyword5` | garage capacity |
| `LM_char10_70` | subdivision |
| `LMD_MP_Latitude` / `LMD_MP_Longitude` | coordinates |
| `L_Photos` | JSON string of photos (strings or objects) — parse with `frontend/src/utils/parsePhotos.js` |

**Frontend structure:**
- Routes in `App.js`: `/` (ListingsPage), `/property/:id` (PropertyDetailPage), `/favorites` (FavoritesPage).
- `hooks/useProperties` fetches listings; it re-runs when `JSON.stringify(params)` changes and guards against stale responses with a `cancelled` flag.
- `hooks/useFavorites` stores listing IDs in `localStorage` (`favoriteProperties`) and keeps every hook instance in sync via a custom `favoritePropertiesChanged` window event plus the `storage` event. FavoritesPage fetches each favorite individually with `fetchPropertyDetail`.
- Styling is Tailwind utility classes.

## Tests

Frontend tests live in `frontend/src/test/` (Jest + React Testing Library; `fetch` is mocked by assigning `global.fetch = jest.fn()`). The project's `writetest` skill (`.claude/skills/writetest`) says: use Jest, keep tests as simple as possible, and put them in the test directory.
