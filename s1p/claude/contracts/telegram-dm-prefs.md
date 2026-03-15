# Contract: Telegram DM Notification Preferences

**Status:** locked
**Date:** 2026-03-15
**Affects:** both

---

## Endpoints

### GET /api/v1/company/telegram/config

The main config response includes `dm_notifications` (company-level toggle).
Individual user prefs are managed separately.

### PUT /api/v1/company/users/{user_id} (existing endpoint)

User's `telegram_dm_prefs` JSONB is updated through the normal user update flow.

#### DM Prefs Schema

```json
{
  "telegram_dm_prefs": {
    "my_calls": true,
    "my_leads": true,
    "assigned_to_me": true
  }
}
```

All keys default to `false` if missing. Empty object `{}` = all DMs off.

---

## UI Location

"Personal Notifications" section on the Telegram settings page.
Only visible when user has a linked telegram_user_id.

### Toggles

| Key | Label (en) | Label (ru) | Description |
|---|---|---|---|
| `my_calls` | My calls | Мои звонки | DM when I handle a call |
| `my_leads` | My leads | Мои лиды | DM when a lead is assigned to me |
| `assigned_to_me` | Assigned to me | Назначено мне | DM when something is assigned to me |

---

## Rules

- Company-level `dm_notifications` must be `true` for any DMs to send
- User must have `telegram_user_id` set (registered via Telegram)
- DMs are fire-and-forget (no retry, no error shown to user)
- One DM per event per user (no duplicates)
