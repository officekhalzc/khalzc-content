# khalzc-content

Content source for the **Khal Zidichov Chodorov** shul app. The app reads this repo
over `raw.githubusercontent.com` and renders whatever the manifest points at. This is
the **single source of truth** for member-facing content.

- **App fetch base:** `https://raw.githubusercontent.com/officekhalzc/khalzc-content/main/`
- Published from the Shul Workspace makers via `khalzc-publish.js` (GitHub Contents API).
- `raw.githubusercontent.com` CDN-caches ~5 min, so a publish reaches the app within a
  few minutes, not instantly.

## Layout

```
/bulletins/      bulletin files (pdf | png | jpg)
/calendars/      monthly calendar files (pdf | png | jpg)
/manifest.json   pointer the app reads first
/events.json     optional one-off events (flat array; empty is fine)
```

## manifest.json contract

Dual-key: a `bulletins[]` feed (newest first) **and** a legacy single `bulletin`
(= the latest), so both feed readers and old single-bulletin readers work. The
publish module maintains both — never hand-edit the array.

```json
{
  "bulletins": [
    { "file": "bulletins/<file>.(pdf|png|jpg)", "date": "YYYY-MM-DD", "title": "Parshas …" }
  ],
  "bulletin": { "file": "bulletins/<latest>.(pdf|png|jpg)", "date": "YYYY-MM-DD", "title": "Parshas …" },
  "calendar": { "file": "calendars/<file>.(pdf|png|jpg)", "month": "YYYY-MM" },
  "updated":  "<ISO-8601 timestamp>"
}
```

- The app renders bulletin/calendar by **file extension** (image vs PDF) — the extension
  must be accurate.
- Empty/absent feed → the app shows its last cached copy / an empty state.

## events.json contract

Flat array; missing or empty is fine.

```json
[
  { "date": "2026-06-19", "title": "Hachnasas Sefer Torah", "time": "7:30 PM", "type": "special" }
]
```

| Field   | Required | Meaning            |
|---------|----------|--------------------|
| `date`  | yes      | `YYYY-MM-DD`       |
| `title` | yes      | event name         |
| `time`  | no       | free text          |
| `type`  | no       | optional tag       |

## Publishing

Don't hand-edit `manifest.json`. Publish through `khalzc-publish.js`
(`publishToApp` / `publishEvent`), which commits the file and updates the manifest
atomically. Write access = a fine-grained PAT scoped to this repo only
(Contents: Read & write).
