# Property Search App

A web app for browsing MLS property listings. You can filter listings by city, zip code, price, bedrooms and bathrooms, open a listing to see its details, photos, map location and open houses, and save favorites.

- **Backend:** Node.js, Express 5, MySQL (`mysql2`)
- **Frontend:** React 19 (Create React App), React Router 7, Tailwind CSS 3

## Prerequisites

- Node.js and npm
- MySQL or MariaDB
- A Google Maps Embed API key (for the map on the property detail page)

## Setup

### 1. Database

Create a database and import the two SQL dumps from the repo root:

```bash
mysql -u <user> -p -e "CREATE DATABASE <db_name>"
mysql -u <user> -p <db_name> < rets_property.sql
mysql -u <user> -p <db_name> < rets_openhouse.sql
```

This creates the `rets_property` and `rets_openhouse` tables. The dumps are large (`rets_property.sql` is about 600MB) and are not committed to git, so you need to get them separately.

### 2. Backend

```bash
cd backend
npm install
```

Create `backend/.env`:

```
PORT=5000
DB_HOST=localhost
DB_PORT=3306
DB_USER=<user>
DB_PASSWORD=<password>
DB_NAME=<db_name>
```

`PORT` must be set, and it should be `5000`, because the frontend's dev proxy sends API requests there.

### 3. Frontend

```bash
cd frontend
npm install
cp .env.example .env
```

Then set `REACT_APP_GOOGLE_MAPS_API_KEY` in `frontend/.env`.

## Running

Start the backend and the frontend in separate terminals:

```bash
# Terminal 1
cd backend
npm run dev      # auto-reloads with nodemon; use `npm start` for no reload

# Terminal 2
cd frontend
npm start        # opens http://localhost:3000
```

To check that the backend can reach the database, open http://localhost:5000/api/health.

## Pages

| Path | Page |
|---|---|
| `/` | Listings with filters and pagination |
| `/property/:id` | Property details, photos, map and open houses |
| `/favorites` | Saved properties |

Favorites are saved in the browser's `localStorage`, so they stay on that browser only.

## API

All endpoints are under `http://localhost:5000/api`.

| Method | Endpoint | Description |
|---|---|---|
| GET | `/health` | Database connection status |
| GET | `/properties` | List properties |
| GET | `/properties/:id` | One property by listing ID |
| GET | `/properties/:id/openhouses` | Open houses for a property |

`GET /properties` accepts these query parameters, all optional:

| Parameter | Description |
|---|---|
| `city` | City name (case-insensitive) |
| `zipcode` | Zip code |
| `minPrice`, `maxPrice` | Price range |
| `beds`, `baths` | Minimum bedrooms / bathrooms |
| `limit` | Results per page, 1–100 (default 20) |
| `offset` | Number of results to skip (default 0) |

Example:

```
GET /api/properties?city=Ann%20Arbor&beds=3&limit=20&offset=0
```

Response:

```json
{ "total": 123, "limit": 20, "offset": 0, "results": [ ... ] }
```

Results are raw rows from the `rets_property` table, so field names follow the MLS/RETS format. A few common ones:

| Field | Meaning |
|---|---|
| `L_ListingID` | Listing ID |
| `L_SystemPrice` | Price |
| `L_Keyword2` | Bedrooms |
| `LM_Dec_3` | Bathrooms |
| `LM_Int2_3` | Approx. finished square feet |
| `LMD_MP_Latitude`, `LMD_MP_Longitude` | Coordinates |
| `L_Photos` | Photos (JSON string) |

## Tests

Frontend tests use Jest and React Testing Library and live in `frontend/src/test/`.

```bash
cd frontend
npm test                                      # watch mode
CI=true npm test                              # run once
CI=true npm test -- src/test/Pagination.test.js   # one file
```

The backend has no tests yet.

## Project structure

```
backend/
  src/
    index.js            Express server and health check
    mysql.js            MySQL connection pool
    routes/properties.js  Property and open house endpoints
frontend/
  src/
    api/client.js       API calls
    pages/              Listings, detail and favorites pages
    components/         Cards, filters, pagination, carousel, map
    hooks/              useProperties, useFavorites
    test/               Tests
```
