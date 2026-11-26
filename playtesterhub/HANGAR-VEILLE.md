# Hangar Veille reports through the PlayTesterHub API and MCP

[Hangar Veille](https://veille.hangarmobile.com) opens your live Android app every day on real phones (a different phone each day), takes screenshots, tries a few safe gestures and writes a report. Your Hangar account **is** your PlayTesterHub account, so the same API key or OAuth connection reads your Hangar reports.

Everything here is **read-only**. No endpoint or tool starts a check or drives the phones.

- **REST base:** `https://playtesterhub.com/api/v1/hangar`
- **MCP address:** `https://playtesterhub.com/api/mcp` (same server as PlayTesterHub, three extra tools)
- **Auth:** `Authorization: Bearer pth_...` (API key from your dashboard › "Your AI agents") or OAuth. You only ever see your own account.

No Hangar subscription yet? You get an empty list and a `message` explaining how to start, never an error.

## REST endpoints

### `GET /api/v1/hangar/watches`

Apps and sites watched by your account.

```bash
curl -s https://playtesterhub.com/api/v1/hangar/watches \
  -H "Authorization: Bearer pth_YOUR_KEY"
```

```json
{
  "account": { "exists": true, "plan": { "id": "garde", "name": "Guard", "price_eur_month": 29 }, "subscription_active": true },
  "watches": [
    {
      "id": "wch_0f3a...",
      "name": "My App",
      "package": "com.example.app",
      "source": "play",
      "play_store_url": "https://play.google.com/store/apps/details?id=com.example.app",
      "server_url": "https://api.example.com/health",
      "plan": { "id": "garde", "name": "Guard" },
      "paused": false,
      "state": "ok",
      "state_since": "2026-11-24T03:12:40.000Z",
      "server_state": "up",
      "created_at": "2026-11-20T10:00:00.000Z",
      "last_check": { "id": "chk_...", "at": "2026-11-26T03:14:02.000Z", "outcome": "ok", "phone": { "brand": "Samsung", "model": "Galaxy A04s", "android": "13" } },
      "latest_report": { "id": "rpt_...", "period": "daily", "date": "2026-11-25", "status": "ok" }
    }
  ]
}
```

- `state`: `ok`, `slow`, `warning`, `broken` or `unknown` (first check still to come).
- `server_state`: `up`, `down`, `unknown`, or `null` when your plan has no server check.

### `GET /api/v1/hangar/reports`

Your reports, newest first: one per day, plus weekly (Guard, Pro) and monthly (Pro).

| Parameter | Meaning |
|---|---|
| `watch` | Only this watch (`wch_...` from `/watches`) |
| `since` | ISO 8601 date: reports whose period starts on or after it |
| `limit` | 1 to 50, default 20 |
| `offset` | For paging, default 0 |

```bash
curl -s "https://playtesterhub.com/api/v1/hangar/reports?since=2026-11-20&limit=7" \
  -H "Authorization: Bearer pth_YOUR_KEY"
```

```json
{
  "account": { "exists": true, "plan": { "id": "garde", "name": "Guard", "price_eur_month": 29 }, "subscription_active": true },
  "total": 6, "limit": 7, "offset": 0,
  "reports": [
    {
      "id": "rpt_9c1e...",
      "watch": { "id": "wch_0f3a...", "name": "My App", "package": "com.example.app" },
      "period": "daily",
      "date": "2026-11-25",
      "period_start": "2026-11-24T23:00:00.000Z",
      "period_end": "2026-11-25T23:00:00.000Z",
      "status": "ok",
      "title": "Rapport quotidien du 25 novembre 2026",
      "body": "Rapport quotidien du 25 novembre 2026 · My App\nTout va bien. 8 ouvertures de l’appli : 8 normales.\n...",
      "title_en": "Daily report, 25 November 2026",
      "body_en": "Daily report, 25 November 2026 · My App\nAll good. 8 app launches: 8 normal.\n...",
      "phones": [ { "brand": "Samsung", "model": "Galaxy A04s", "android": ["13"], "checks": 4 } ],
      "checks": [
        {
          "id": "chk_...", "at": "2026-11-25T03:14:02.000Z", "kind": "scheduled", "outcome": "ok", "reason": null,
          "phone": { "brand": "Samsung", "model": "Galaxy A04s", "android": "13" },
          "startup_ms": 2100,
          "screenshot_url": "https://veille.hangarmobile.com/api/v1/shots/chk_...?exp=...&sig=...",
          "journey": { "steps": 4 }
        }
      ],
      "checks_truncated": false,
      "data": { "verdict": "ok", "checks": { "total": 8, "ok": 8, "slow": 0, "warning": 0, "broken": 0, "infra": 0 }, "load": { "avg_ms": 2100, "p95_ms": 2600, "max_ms": 2600 }, "...": "..." }
    }
  ]
}
```

- `status`: `ok`, `warning` (slow or something to keep an eye on), `problem` (a failure confirmed on a second phone, or the server was down) or `no_data`.
- `body` / `body_en`: the readable summary in French (same text as in your Hangar space and e-mails) and in English. `title` / `title_en` likewise.
- `data`: every number of the report as JSON (checks, startup times, phones covered, gestures, server uptime, incidents).
- `outcome` of a check: `ok`, `slow`, `warning`, `broken`, `not_installed` or `infra` (not run because of our platform: never counted against your app).
- `screenshot_url`: temporary signed link (valid 7 days; screenshots are kept 60 days). Call the API again for fresh links.

### `GET /api/v1/hangar/reports/{id}`

One report, with every check of the period and, for each one, the gestures of the short walk in the app with a screenshot per step.

```bash
curl -s https://playtesterhub.com/api/v1/hangar/reports/rpt_9c1e... \
  -H "Authorization: Bearer pth_YOUR_KEY"
```

```json
{
  "report": {
    "id": "rpt_9c1e...",
    "status": "ok",
    "checks": [
      {
        "id": "chk_...",
        "outcome": "ok",
        "phone": { "brand": "Samsung", "model": "Galaxy A04s", "android": "13" },
        "screenshot_url": "https://veille.hangarmobile.com/api/v1/shots/chk_...?exp=...&sig=...",
        "journey": [
          { "step": 1, "gesture": "touche « Commencer »", "result": "ok", "message": null, "screenshot_url": "https://veille.hangarmobile.com/api/v1/shots/chk_...-1?exp=...&sig=..." }
        ]
      }
    ],
    "...": "same fields as in the list"
  }
}
```

A report of another account answers `404`, like an unknown id.

### Errors

| Code | When |
|---|---|
| `400 invalid_query` / `invalid_id` | Bad filter (the message says which one) or malformed id |
| `401 unauthorized` | Missing, unknown or revoked key |
| `404 not_found` | Unknown report, or not yours |
| `503 hangar_unavailable` | Hangar is briefly unreachable: try again in a few minutes |

## MCP tools

| Tool | What it does |
|---|---|
| `hangar_list_watches` | Apps and sites watched, plan, state, last check, latest report |
| `hangar_list_reports` | Reports, newest first. Optional `watch_id`, `since`, `limit` (max 50), `offset` |
| `hangar_get_report` | One report (`report_id`) with the screenshots as images (JPEG). `include_images=false` for text only, `max_images` 1 to 24 (default 8) |

All three are read-only. Add the server once (see the [main README](../README.md)) and they show up next to the PlayTesterHub tools.

## Questions to ask your agent

- "Did my app have a problem this week on Hangar? If so, on which phone, and show me the screenshot."
- "Summarize my last 7 Hangar reports and tell me which phones were the slowest to start the app."
- "Open yesterday's Hangar report, look at the screenshots of each step and tell me if anything looks broken."
