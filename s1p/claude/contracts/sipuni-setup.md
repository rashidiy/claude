# Contract: Sipuni One-Click Setup

**Status:** locked
**Date:** 2026-03-18
**Affects:** both

---

## Endpoints

### POST /api/v1/company/sipuni/setup

Start automated Sipuni setup. Logs into Sipuni dashboard, extracts credentials, enables services, adds webhook.

#### Auth
JWT + `settings.manage`

#### Request

```json
{
  "email": "string — Sipuni dashboard email",
  "password": "string — Sipuni dashboard password (used once, never stored)"
}
```

#### Response (200)

```json
{
  "setup_status": "setting_up",
  "setup_error": null,
  "setup_method": null,
  "is_connected": false,
  "cabinet_id": null,
  "webhook_url": null
}
```

#### Error Responses

| Code | When | Body |
|---|---|---|
| 400 | Company is not Sipuni provider | `{ "detail": "Company does not use Sipuni provider" }` |
| 409 | Setup already in progress | `{ "detail": "Setup already in progress" }` |

---

### GET /api/v1/company/sipuni/setup/status

Poll for setup progress.

#### Auth
JWT + `settings.read`

#### Response (200)

```json
{
  "setup_status": "not_started | setting_up | ready | failed",
  "setup_error": "string | null",
  "setup_method": "auto | manual | null",
  "is_connected": false,
  "cabinet_id": "string | null",
  "webhook_url": "string | null"
}
```

---

### POST /api/v1/company/sipuni/setup/manual

Manual setup — admin provides cabinet ID and security key directly.

#### Auth
JWT + `settings.manage`

#### Request

```json
{
  "cabinet_id": "string — Sipuni cabinet number",
  "security_key": "string — API integration key"
}
```

#### Response (200)

Same as setup/status response.

#### Error Responses

| Code | When | Body |
|---|---|---|
| 400 | Company is not Sipuni provider | `{ "detail": "Company does not use Sipuni provider" }` |

---

### GET /api/v1/company/sipuni/config

Get current Sipuni connection configuration.

#### Auth
JWT + `settings.read`

#### Response (200)

```json
{
  "is_connected": true,
  "cabinet_id": "018315",
  "security_key_masked": "0.w56***",
  "webhook_url": "https://api.example.com/api/v1/company/webhooks/abc123",
  "setup_status": "ready",
  "setup_method": "auto",
  "services_enabled": { "stream": true, "callback": true }
}
```

#### Response (when not connected)

```json
{
  "is_connected": false,
  "cabinet_id": null,
  "security_key_masked": null,
  "webhook_url": null,
  "setup_status": "not_started",
  "setup_method": null,
  "services_enabled": null
}
```

---

### POST /api/v1/company/sipuni/disconnect

Disconnect Sipuni. Clears provider_config, resets setup status.

#### Auth
JWT + `settings.manage`

#### Response (200)

```json
{
  "setup_status": "not_started",
  "setup_error": null,
  "setup_method": null,
  "is_connected": false,
  "cabinet_id": null,
  "webhook_url": null
}
```

---

## UI States

Frontend shows different views based on `setup_status`:

| Status | What to show |
|---|---|
| `not_started` | Two cards: "Automatic Setup" (email+password form) / "Manual Setup" (cabinet_id+security_key form) |
| `setting_up` | Spinner + "Connecting to Sipuni..." + poll every 3s |
| `ready` | Green status, cabinet ID, masked security key, webhook URL with copy, disconnect button |
| `failed` | Error message + retry button + manual fallback |

## Polling

When `setup_status === "setting_up"`:
- Poll `GET /setup/status` every 3 seconds
- Stop polling when status changes to `ready` or `failed`
- Max poll duration: 2 minutes, then show timeout message

## Rules

- Setup requires SETTINGS_MANAGE permission
- Only one setup can run at a time (409 if already setting_up)
- Password is passed in-memory to background task, never stored in DB/logs/files
- `provider_type` must be `sipuni` — reject for binotel companies
- Webhook URL format: `{API_BASE}/api/v1/company/webhooks/{company.webhook_token}`
- Does NOT remove webhooks from Sipuni on disconnect (would need password again)
- Manual setup skips Sipuni dashboard entirely — just stores credentials
