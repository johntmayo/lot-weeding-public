# Lot Weeding Public Dashboard Handoff

## Purpose

`lot-weeding-public/index.html` is a standalone, public-facing dashboard for the Lot Weeding program. It is intentionally separate from the main authenticated Zone Dashboard and can be copied out of this repository for static hosting.

The page shows high-level program progress:

- Summary cards for requested, schedule next, scheduled, cleaned, needs attention, and total requests.
- A Leaflet map with status-colored markers.
- A calendar grouped by `Date Scheduled`.
- A read-only public request list.

It does not support edits, authentication, private admin operations, or calls to the app backend.

## Current Data Source

The page reads this published Google Sheets CSV:

```text
https://docs.google.com/spreadsheets/d/e/2PACX-1vQH7w_b9chbRrHAMmd0aNshUrD3f6VDrl7nezNGrS940r0sLYR4OUPctVgKDDyW0TIBKsltLjfyIdCl/pub?gid=0&single=true&output=csv
```

This URL is hardcoded as `CSV_URL` inside `index.html`.

The CSV is expected to come from a public mirror sheet, not the private operational backend sheet. The mirror was created from the official backend spreadsheet using selected columns only.

Expected headers:

```text
Date Received
Homeowner Name
Address
APN
Date Scheduled
Date Cleaned
Status
Latitude
Longitude
```

The page was sanity-checked against the feed after creation. At that time the feed returned HTTP 200, `316` CSV lines, and the expected headers above.

## Privacy Model

The public mirror currently includes homeowner name and address because the product owner said that is acceptable for this public dashboard.

The page does not use or display:

- Phone numbers
- Email addresses
- Notes
- ROE status
- Homeowner notification status
- UWS contract status
- Neighborhood captain contact info
- Any authenticated admin endpoint

If the privacy posture changes, update the public mirror sheet first. The HTML can tolerate missing `Homeowner Name`, but it needs `Address`, `Status`, and preferably `Latitude` / `Longitude` for a useful map.

## Hosting

This file is truly static. It can be hosted on GitHub Pages, Netlify, Vercel static hosting, S3, or any plain web server.

External runtime dependencies:

- Published Google Sheets CSV URL
- Leaflet CSS/JS from `https://unpkg.com`
- CARTO/OpenStreetMap map tiles from `https://{s}.basemaps.cartocdn.com`

No Node server, environment variables, build step, package install, service account, or repository-specific assets are required.

Important local-file note: opening `index.html` directly from disk with a `file://` URL may fail or behave inconsistently because browsers often restrict `fetch()` calls from local files. Host it over HTTP/HTTPS for realistic testing. GitHub Pages is fine.

## Implementation Notes

The file contains all custom HTML, CSS, and JavaScript inline. This was done intentionally so the owner can extract a single file.

Key behavior:

- `fetch(CSV_URL, { cache: 'no-store' })` loads the public CSV.
- A small built-in CSV parser handles quoted cells and line breaks.
- Headers are normalized by lowercasing and removing non-alphanumeric characters.
- Dates support Google serial dates, `M/D/YYYY`, `YYYY-MM-DD`, and JavaScript-parsable fallback dates.
- Statuses normalize a few legacy values:
  - blank / `Open` -> `Requested`
  - `On-Deck` -> `Schedule Next`
  - `Completed` -> `Cleaned`
  - `Flagged` -> `Needs Attention`
- Map markers require valid numeric `Latitude` and `Longitude`.
- Calendar events come from `Date Scheduled`.

## Status Vocabulary

The public page uses the same broad Lot Weeding status vocabulary as the admin command center:

- `Requested`
- `Schedule Next`
- `Scheduled`
- `Cleaned`
- `Needs Attention`
- `Cancelled`

Status colors were matched to the internal Lot Weeding Command Center palette:

- `Requested`: `#fdba77`
- `Schedule Next`: `#f9d6d3`
- `Scheduled`: `#81bdc3`
- `Cleaned`: `#afc892`
- `Needs Attention`: `#bc455a`
- `Cancelled`: `#d8d8d0`

## Common Future Changes

To point the page at a different public mirror sheet, update `CSV_URL` in `index.html`.

To remove homeowner names from the public view:

1. Remove `Homeowner Name` from the public mirror sheet.
2. Optionally remove the display snippets for `row.homeownerName` in map popups, calendar day rows, and the table.

To make the file more self-contained, download Leaflet CSS/JS and host them beside the page, then replace the CDN links. Map tiles will still need a tile provider unless a static/non-map fallback is built.

To test the public feed quickly:

```bash
node -e "const url='PASTE_CSV_URL'; fetch(url).then(r=>{console.log(r.status); return r.text();}).then(t=>console.log(t.split(/\r?\n/)[0]));"
```

## Relationship To Main App

The main app's full Lot Weeding Command Center lives in the authenticated dashboard and is documented in `LOT_WEEDING_ADMIN_HANDOFF.md`.

This public page does not reuse that app code directly. It copies only the public-facing concepts:

- high-level counts
- status palette
- map markers
- scheduled-date calendar

Keep it independent unless there is a deliberate decision to turn this into a maintained app route.
