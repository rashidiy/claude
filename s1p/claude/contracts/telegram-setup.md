# Contract: Telegram V2 Setup Flow

**Status:** locked
**Date:** 2026-03-15
**Affects:** both

---

## Endpoints

### GET /api/v1/company/telegram/config

Returns full config including V2 fields.

#### Response (200)

```json
{
  "id": "uuid",
  "company_id": "uuid",
  "chat_id": "string | null",
  "bot_enabled": true,
  "notify_completed_calls": true,
  "notify_missed_calls": true,
  "notify_new_leads": true,
  "notify_deal_stage_change": true,
  "group_chat_id": 123456789,
  "setup_status": "not_started | creating | ready | failed | manual",
  "setup_error": "string | null",
  "invite_link": "https://t.me/+abc123 | null",
  "group_name": "S1P — Company Name | null",
  "language": "ru | en | uz",
  "send_recordings": true,
  "daily_digest": true,
  "dm_notifications": false
}
```

### POST /api/v1/company/telegram/setup

Start automated group setup.

#### Request

```json
{
  "company_name": "string — company display name for the group title",
  "language": "ru | en | uz — default: ru"
}
```

#### Response (200)

```json
{
  "setup_status": "creating",
  "setup_error": null,
  "invite_link": null,
  "group_name": "S1P — Company Name",
  "group_chat_id": null
}
```

#### Error Responses

| Code | When | Body |
|---|---|---|
| 409 | Setup already in progress | `{ "detail": "Setup already in progress" }` |
| 503 | Pyrogram not available | `{ "detail": "Automated setup not available. Use manual setup." }` |

### GET /api/v1/company/telegram/setup/status

Poll for setup progress.

#### Response (200)

```json
{
  "setup_status": "not_started | creating | ready | failed | manual",
  "setup_error": "string | null",
  "invite_link": "string | null",
  "group_name": "string | null",
  "group_chat_id": 123456789
}
```

### POST /api/v1/company/telegram/setup/manual

Manual setup fallback.

#### Request

```json
{
  "chat_id": "string — the Telegram chat/group ID"
}
```

#### Response (200)

Same as setup/status response.

### PUT /api/v1/company/telegram/config

Update config including V2 fields.

#### Request (partial update)

```json
{
  "bot_enabled": true,
  "notify_completed_calls": true,
  "notify_missed_calls": true,
  "notify_new_leads": true,
  "notify_deal_stage_change": true,
  "language": "ru | en | uz",
  "send_recordings": true,
  "daily_digest": true,
  "dm_notifications": false
}
```

---

## UI States

Frontend should show different views based on `setup_status`:

| Status | What to show |
|---|---|
| `not_started` | Setup wizard: "Connect Telegram" button + manual fallback |
| `creating` | Spinner + "Setting up..." + poll every 3s |
| `ready` | Success state: invite link, notification toggles, language selector |
| `failed` | Error message + retry button + manual fallback |
| `manual` | Partial success: connected without topics, notification toggles |

## Polling

When `setup_status === "creating"`:
- Poll `GET /setup/status` every 3 seconds
- Stop polling when status changes to `ready` or `failed`
- Max poll duration: 2 minutes, then show timeout message

## Rules

- Setup requires SETTINGS_MANAGE permission
- Only one setup can run at a time (409 if already creating)
- Automated setup may fail (Pyrogram not available) — manual fallback always works
- `invite_link` is only set when automated setup succeeds
- Language affects all future notifications (not retroactive)
