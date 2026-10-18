# OpenTrail

Community-driven trail information resource for thru-hikers.



## Getting Started

### Preview the design without a database

```sh
npm ci
npm run dev:demo
```

Open http://localhost:5174/app?trail=test (the command opens it automatically).
This development-only mode includes four fictional waypoints and sample comments.
You can test the map, navigation, filters, detail sheets, settings and add-waypoint
form. Saving, uploading and moderation return an explicit demo-mode error.
Other trails display their maps without sample waypoints. Internet access is
required for the trail geometry and map tiles. No database credentials are needed.

For mobile layouts, use your browser's device toolbar (F12, then Ctrl+Shift+M in
Chrome/Edge). To open it on a phone on the same Wi-Fi, run
`npm run dev:demo -- --host 0.0.0.0` and use the Network URL printed in the terminal.
Phone geolocation, offline storage and PWA installation require HTTPS; use
localhost on your computer for those features. Stop the server with Ctrl+C.

### Develop with the backend

Create a local `.env` with `DATABASE_URL` pointing to a development PostgreSQL
database matching `prisma/schema.prisma`, then run:

```
npm install
npx prisma generate
npm run dev
```

Marker elevation lookup and photo uploads additionally need the Google Maps and
S3 credentials referenced in the API routes. Use a test database and test storage
for submission testing.

## API

### GET /api/getData

Returns trail markers as a GeoJSON FeatureCollection.

**Query parameters:**

- `trail` (required) — Trail name: `PCT`, `AT`, or `CDT`

**Example:**

```
GET /api/getData?trail=PCT
```

**Response:** `200` — GeoJSON `FeatureCollection` of `Point` features. Each feature's `properties` includes `title`, `mile`, `elev`, `desc`, `icons`, `images`, `comments`, `commentCount`, and `dbid`.

Supports `ETag`/`If-None-Match` (returns `304` if unchanged). Errors return `400`.
