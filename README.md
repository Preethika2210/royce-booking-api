# Royce Dental booking API (demo)

Your own copy of the Royce booking backend for the Zendesk AI agent demo.
Same endpoints, parameters, responses and error codes as the customer's POC
(`royce-booking-poc.onrender.com`), but clinics, dentists and bookings live
in **your own Postgres database** (Supabase), so bookings survive restarts.

Your Zendesk custom actions keep working: change only the **base URL** and
the **X-API-Key** value.

## What is inside

| Path | What it is |
| --- | --- |
| `sql/schema.sql` | Creates the 3 tables and loads 3 clinics and 8 dentists. Run it once in Supabase. |
| `src/server.js` | The API (Express). All endpoints, validation and error codes. |
| `src/slots.js` | Works out open slots from each dentist's weekly schedule. |
| `src/time.js` | Singapore date and time helpers and labels. |
| `public/index.html` | Demo patient website at `/` and `/demo`. Loads your Zendesk chat widget. |
| `public/admin.html` | Staff page at `/admin`: confirm, decline, cancel, clear all. |
| `render.yaml` | Render settings (free plan, Singapore region). |

## Endpoints

All `/api/*` calls need the header `X-API-Key: <API_KEY>`.

| # | Method and path | Purpose |
| --- | --- | --- |
| – | `GET /health` | Status check, no key |
| 1 | `GET /api/services` | consult, cleaning, filling |
| 2 | `GET /api/regions` | Central, East, North, North-East, West |
| 3 | `GET /api/clinics?q=&region=` | List or search clinics |
| 4 | `GET /api/clinics/{id}` | One clinic |
| 5 | `GET /api/clinics/{id}/dentists?service=` | Dentists with open slots |
| 6 | `GET /api/clinics/{id}/dates?service=&dentist=&from=&limit=` | Dates with openings |
| 7 | `GET /api/clinics/{id}/slots?date=&service=&dentist=&period=` | Times on one date |
| 8 | `GET /api/clinics/{id}/next?service=&dentist=&limit=` | Earliest open slots |
| 9 | `POST /api/bookings` | Create a booking request (201) |
| 10 | `GET /api/bookings?mobile=&all=` | A patient's bookings |
| 11 | `GET /api/bookings/{ref}?mobile=` | One booking |
| 12 | `POST /api/bookings/{ref}/reschedule` | Move to another slot, same clinic |
| 13 | `POST /api/bookings/{ref}/cancel` | Cancel |

Errors: `{ "error": "<code>", "message": "<text for the patient>" }` with
400 (invalid_service, invalid_date, invalid_slot, invalid_name, invalid_mobile,
invalid_email, invalid_json), 401 (unauthorized), 404 (clinic_not_found,
booking_not_found), 409 (slot_unavailable, not_changeable).

## Settings (environment variables)

| Name | Value |
| --- | --- |
| `DATABASE_URL` | Supabase **Session pooler** connection string (see the deploy guide) |
| `API_KEY` | The key Zendesk sends in `X-API-Key` |
| `ADMIN_PASSWORD` | Password for `/admin` (any username) |
| `ZENDESK_WIDGET_KEY` | Key of your Zendesk messaging Web Widget: Admin Center → Channels → Messaging → your widget → Installation, the part after `key=` in the snippet |

## Demo website

Open `https://<your-service>.onrender.com/` (or `/demo`). The page loads your
Zendesk Web Widget, so patients chat with your AI agent there. If
`ZENDESK_WIDGET_KEY` is empty, the page shows a red notice instead of the chat.
In Zendesk, add your Render address to the widget's allowed domains if you
have domain restrictions turned on.

## Run locally (optional)

```bash
npm install
DATABASE_URL=... API_KEY=... ADMIN_PASSWORD=... npm start
curl http://localhost:3000/health
```

## Change the data

- **Clinics and dentists:** edit them in Supabase Table Editor, or change
  `sql/schema.sql` and run it again (this also deletes all bookings).
- **Working hours:** the `schedule` column of `dentists`, e.g.
  `{"1":[["09:00","13:00"],["14:00","18:00"]]}` (1 = Monday … 7 = Sunday).
- **"Busy" slots:** about 35% of slots show as already taken so the diary
  looks real. Change `BUSY_SHARE` in `src/slots.js` (0 = all free).
- **Clear bookings before a demo:** `/admin` → Clear all bookings.

Dentist names in the seed data are fictional demo data.
