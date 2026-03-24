# Contract: SIP Extension Integration + Call Fixes

**Status:** locked
**Date:** 2026-03-19
**Affects:** both

---

## 1. Sipuni Operators List

```
GET /api/v1/company/sipuni/operators
```

**Auth:** JWT cookie, requires CALLS_MAKE permission
**Provider:** Must be Sipuni

### Response (200)

```json
[
  {
    "extension": "201",
    "name": "Алекс",
    "status": "online"
  }
]
```

### Error Responses

| Code | When | Body |
|---|---|---|
| 400 | Company provider is not Sipuni | `{ "detail": "This endpoint is only available for sipuni companies" }` |
| 502 | Sipuni API unreachable | `{ "detail": "Sipuni service error: ..." }` |

---

## 2. User Schema Changes

### UserResponse (GET /users/me, GET /users, GET /users/{id})

Added field:
```json
{
  "sip_extension": "string | null"
}
```

### UserUpdateRequest (PUT /users/{id})

Added field:
```json
{
  "sip_extension": "string | null"
}
```

### UserDetailResponse

Inherits `sip_extension` from UserResponse.

---

## 3. Invite Token Changes

### InviteTokenCreateRequest (POST /users/invite-telegram)

Added field:
```json
{
  "sip_extension": "string | null"
}
```

`sip_extension` is propagated: InviteToken → User on registration.

---

## 4. Call Endpoints Behavior Change

### All 3 call endpoints:
- `POST /calls/sipuni/external`
- `POST /calls/sipuni/number`
- `POST /calls/sipuni/tree`

**New behavior for operator_id resolution:**

1. If caller is `COMPANY_OPERATOR` role:
   - Use `user.sip_extension` as the SIP extension
   - Ignore `request.operator_id` from request
   - Return 400 if no `sip_extension` assigned
2. If caller is admin/manager:
   - Use `request.operator_id` as before (required)
   - Return 400 if not provided

**Frontend impact:** Call popup `operator_id` field becomes a `<Select>` dropdown populated from endpoint #1. For operators, it's disabled and auto-filled.

---

## 5. Webhook Phone Overwrite Fix

**No API change.** Backend-only fix.

When updating an existing CallEvent via webhook, if `phone_1` or `phone_2` already has a value, skip overwriting from webhook data.

---

## Rules

- `sip_extension` is a String(20), nullable, no unique constraint (multiple users can share one)
- Operators see their own extension pre-filled and disabled in the call popup
- Admins/managers see a dropdown of all Sipuni operators
- The Sipuni operators API calls `POST https://sipuni.com/api/statistic/operators` with `user=cabinet_id, hash=MD5(cabinet_id+secret)`
- Response is CSV, must be parsed into JSON

## Frontend Types

```typescript
interface SipuniOperator {
  extension: string;
  name: string;
  status: string;
}

// Add to AuthUser interface:
sip_extension?: string | null;

// Add to InviteTokenCreateRequest:
sip_extension?: string | null;

// Add to UserResponse/UserUpdateRequest:
sip_extension?: string | null;
```

## i18n Keys (add to calls section)

```json
{
  "selectOperator": {
    "ru": "Выберите оператора",
    "en": "Select operator",
    "uz": "Operatorni tanlang"
  },
  "noSipExtension": {
    "ru": "SIP номер не назначен",
    "en": "No SIP extension assigned",
    "uz": "SIP raqam tayinlanmagan"
  },
  "sipExtension": {
    "ru": "SIP номер",
    "en": "SIP Extension",
    "uz": "SIP raqam"
  },
  "loadingOperators": {
    "ru": "Загрузка операторов...",
    "en": "Loading operators...",
    "uz": "Operatorlar yuklanmoqda..."
  }
}
```

Add to users section:
```json
{
  "sipExtension": {
    "ru": "SIP номер",
    "en": "SIP Extension",
    "uz": "SIP raqam"
  }
}
```
