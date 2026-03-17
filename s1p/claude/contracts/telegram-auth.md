# Contract: Telegram-Based Authentication

**Status:** implemented
**Date:** 2026-03-12
**Affects:** both

---

## Overview

Replace email+password login for company users (Admin, Manager, Operator) with Telegram-based authentication. Owner keeps email+password (unchanged). Two flows: (1) registration via invite token + Telegram bot, (2) login via Telegram OTP.

**Bot username:** `@s1pcrm_bot`
**Bot deep link base:** `https://t.me/s1pcrm_bot`

---

## DB Model Changes

### User model (`source/db/models/user.py`)

Add columns:
- `telegram_user_id`: `BigInteger`, nullable, unique globally, indexed
- Make `email` nullable (was `nullable=False`) — still unique per company when present
- Make `password_hash` nullable (was `nullable=False`) — only Owner needs it
- Make `phone` required (`nullable=False`) and unique per company (add `UniqueConstraint('company_id', 'phone', name='uq_company_phone')`)

Update constraints:
- Keep `uq_company_email` but it now allows NULLs (Postgres unique constraints ignore NULLs)
- Add `Index('idx_users_telegram_user_id', 'telegram_user_id')`

### New model: `InviteToken` (`source/db/models/invite_token.py`)

```
id              UUID, PK, default=uuid4
company_id      UUID, FK -> companies.id (CASCADE), NOT NULL, indexed
token_hash      String(64), NOT NULL, indexed  -- SHA-256 hex of XXXX-XXXX token
role            Enum(RoleEnum), NOT NULL
first_name      String(225), NOT NULL
last_name       String(225), nullable
phone           String(50), NOT NULL
permissions     JSONB, default=[]
permission_group_id  UUID, FK -> permission_groups.id (SET NULL), nullable
created_by      UUID, FK -> users.id (SET NULL), nullable, indexed
expires_at      DateTime(tz=True), NOT NULL  -- created_at + 48h
used_at         DateTime(tz=True), nullable
used_by         UUID, FK -> users.id (SET NULL), nullable
created_at      DateTime(tz=True), server_default=now()
```

### New model: `TelegramAuthChallenge` (`source/db/models/telegram_auth_challenge.py`)

```
id              String(32), PK  -- nanoid or hex, NOT a UUID (used in deep links, must be URL-safe and short)
company_id      UUID, FK -> companies.id (CASCADE), NOT NULL, indexed
telegram_user_id  BigInteger, nullable  -- set when bot handles /start
user_id         UUID, FK -> users.id (SET NULL), nullable  -- resolved CRM user
otp_hash        String(64), nullable  -- SHA-256 hex of 6-digit OTP
purpose         String(10), NOT NULL  -- "login" or "register"
expires_at      DateTime(tz=True), NOT NULL
attempts        Integer, default=0  -- max 3
used            Boolean, default=False
created_at      DateTime(tz=True), server_default=now()
```

### New enum value in `enums.py`

```python
class ChallengePurposeEnum(str, enum.Enum):
    LOGIN = "login"
    REGISTER = "register"
```

---

## Endpoints

### 1. Create Invite Token (Admin creates user)

Replaces the current `POST /api/v1/company/users/invite` flow. Instead of creating a user + sending email with temp password, this generates an invite token. The user record is NOT created until registration completes.

```
POST /api/v1/company/users/invite-telegram
```

**Auth:** Required (company_admin or company_manager with `users.create` permission)
**Company:** From JWT `company_id`

#### Request

```json
{
  "first_name": "string, required, max 225",
  "last_name": "string | null, max 225",
  "phone": "string, required, max 50",
  "role": "string, default 'company_operator' -- company_admin | company_manager | company_operator",
  "permissions": "string[] | null -- custom permissions, validated against VALID_PERMISSIONS",
  "permission_group_id": "UUID | null"
}
```

#### Response (201)

```json
{
  "invite_token": "A3B7-K9M2",
  "expires_at": "2026-03-14T12:00:00Z",
  "role": "company_operator",
  "first_name": "Aziz",
  "phone": "+998901234567"
}
```

The `invite_token` plaintext is returned ONCE. Backend stores only SHA-256 hash.

#### Error Responses

| Code | When | Body |
|---|---|---|
| 400 | Invalid role (owner) | `{ "detail": "Cannot create owner-level users" }` |
| 400 | Invalid permissions | `{ "detail": "Invalid permissions: [...]" }` |
| 400 | Permission group not found or wrong company | `{ "detail": "Permission group not found" }` |
| 400 | Phone already in use by active user in company | `{ "detail": "User with this phone already exists in your company" }` |
| 403 | Insufficient permissions | `{ "detail": "Permission denied" }` |
| 409 | Active (unused) invite token already exists for this phone+company | `{ "detail": "An active invite already exists for this phone number" }` |

---

### 2. List Invite Tokens (Admin views pending invites)

```
GET /api/v1/company/users/invite-tokens
```

**Auth:** Required (company_admin or `users.read` permission)

#### Query Parameters

| Param | Type | Default | Description |
|---|---|---|---|
| `page` | int | 1 | Page number |
| `page_size` | int | 20 | Items per page (max 100) |
| `status` | string | null | Filter: `pending` (unused + not expired), `used`, `expired` |

#### Response (200)

```json
{
  "items": [
    {
      "id": "uuid",
      "role": "company_operator",
      "first_name": "Aziz",
      "last_name": null,
      "phone": "+998901234567",
      "created_by_name": "Admin User",
      "expires_at": "2026-03-14T12:00:00Z",
      "used_at": null,
      "created_at": "2026-03-12T12:00:00Z",
      "status": "pending"
    }
  ],
  "total": 5,
  "page": 1,
  "page_size": 20
}
```

`status` is computed: if `used_at` is set -> `"used"`, if `expires_at < now` -> `"expired"`, else `"pending"`.

---

### 3. Revoke Invite Token

```
DELETE /api/v1/company/users/invite-tokens/{token_id}
```

**Auth:** Required (company_admin or `users.create` permission)

#### Response (204)

No body. Hard-deletes the row.

#### Error Responses

| Code | When | Body |
|---|---|---|
| 404 | Token not found or wrong company | `{ "detail": "Invite token not found" }` |
| 400 | Token already used | `{ "detail": "Cannot revoke a used invite token" }` |

---

### 4. Create Login Challenge

Called by the frontend login page when user clicks "Get code via Telegram".

```
POST /api/v1/auth/telegram/login-challenge
```

**Auth:** None (public endpoint)
**Company:** From Origin header (subdomain), same as existing login

#### Request

No body. Company is resolved from Origin header.

#### Response (200)

```json
{
  "challenge_id": "aB3kM9x2pQ7nR4wZ",
  "deep_link": "https://t.me/s1pcrm_bot?start=login_aB3kM9x2pQ7nR4wZ",
  "expires_at": "2026-03-12T12:05:00Z"
}
```

**Rate limit:** 10/minute per IP

#### Error Responses

| Code | When | Body |
|---|---|---|
| 400 | Missing or invalid Origin header | `{ "detail": "Origin header is required" }` |
| 404 | Company not found | `{ "detail": "Company not found" }` |
| 403 | Company is inactive | `{ "detail": "Company is inactive" }` |

---

### 5. Poll Login Challenge Status

Frontend polls this to know when the bot has processed the /start command and the OTP has been sent. This tells the UI to show the OTP input.

```
GET /api/v1/auth/telegram/login-challenge/{challenge_id}/status
```

**Auth:** None (public endpoint)

#### Response (200)

```json
{
  "status": "pending | otp_sent | expired | used",
  "expires_at": "2026-03-12T12:05:00Z"
}
```

- `pending` — challenge created, user hasn't opened the bot yet
- `otp_sent` — bot sent OTP, user can now enter code
- `expired` — challenge expired (5 min)
- `used` — OTP already verified successfully

**Rate limit:** 30/minute per IP

#### Error Responses

| Code | When | Body |
|---|---|---|
| 404 | Challenge not found | `{ "detail": "Challenge not found" }` |

---

### 6. Verify Login OTP

User enters the 6-digit code from Telegram and submits.

```
POST /api/v1/auth/telegram/verify-otp
```

**Auth:** None (public endpoint)
**Company:** From Origin header (subdomain)

#### Request

```json
{
  "challenge_id": "string, required -- the challenge ID from step 4",
  "otp": "string, required -- 6-digit code, e.g. '472918'"
}
```

#### Response (200)

Same shape as existing `AuthSchema.AuthorizedResponse`:

```json
{
  "id": "uuid",
  "first_name": "Aziz",
  "last_name": "Karimov",
  "email": null,
  "phone": "+998901234567",
  "is_active": true,
  "must_change_password": false,
  "credentials": {
    "type": "Bearer",
    "access": "eyJ...",
    "refresh": "eyJ..."
  }
}
```

Also sets `access_token` and `refresh_token` as httpOnly cookies (same as existing `_set_auth_cookies`).

**Token durations:**
- Access token: 24 hours (`timedelta(hours=24)`)
- Refresh token: 30 days (`timedelta(days=30)`)

**Rate limit:** 5/minute per IP

#### Error Responses

| Code | When | Body |
|---|---|---|
| 400 | Challenge not found or wrong company | `{ "detail": "Invalid or expired challenge" }` |
| 400 | Challenge expired | `{ "detail": "Invalid or expired challenge" }` |
| 400 | Challenge already used | `{ "detail": "Invalid or expired challenge" }` |
| 400 | OTP not yet sent (no telegram_user_id on challenge) | `{ "detail": "OTP not yet sent. Open the Telegram link first." }` |
| 401 | OTP mismatch (increment attempts) | `{ "detail": "Invalid code. N attempts remaining." }` |
| 423 | Max attempts reached (3) | `{ "detail": "Too many attempts. Request a new code." }` |
| 423 | User locked out (5 failed OTPs in 15 min -> 30 min cooldown) | `{ "detail": "Account temporarily locked. Try again in N minutes." }` |
| 403 | User is suspended or inactive | `{ "detail": "Account is suspended" }` |

---

### 7. Create Registration Challenge (Bot-initiated)

This is NOT an HTTP endpoint. This happens inside the Telegram bot handler when user sends `/register`.

**Bot handler: `/register` command**

1. Bot receives `/register` from telegram user
2. Bot creates a `TelegramAuthChallenge` with:
   - `purpose = "register"`
   - `telegram_user_id = message.from_user.id`
   - `expires_at = now + 10 minutes`
   - `company_id` = NULL (not known yet — resolved at token validation)
3. Bot replies with a message containing deep links for each known company (or a generic link):

```
To complete registration, open this link and enter your invite code:

https://s1p.uz/auth/register?s=REG_SESSION_ID
```

**Note:** The registration page URL is a single domain (not company-specific subdomain) because the user doesn't know their company yet. The backend resolves the company from the invite token.

---

### 8. Validate Registration (Complete invite)

User opens the registration link, enters the XXXX-XXXX invite token.

```
POST /api/v1/auth/telegram/register
```

**Auth:** None (public endpoint)

#### Request

```json
{
  "session_id": "string, required -- REG_SESSION_ID from bot deep link",
  "invite_token": "string, required -- XXXX-XXXX format"
}
```

#### Response (201)

```json
{
  "id": "uuid",
  "first_name": "Aziz",
  "last_name": "Karimov",
  "email": null,
  "phone": "+998901234567",
  "is_active": true,
  "must_change_password": false,
  "credentials": {
    "type": "Bearer",
    "access": "eyJ...",
    "refresh": "eyJ..."
  }
}
```

Also sets httpOnly cookies.

**Backend logic:**
1. Look up `TelegramAuthChallenge` by `session_id` where `purpose = "register"` and not expired and not used
2. Get `telegram_user_id` from the challenge
3. Hash the `invite_token` with SHA-256, look up `InviteToken` by `token_hash`
4. Validate: token not expired, not used
5. Check no existing user with this `telegram_user_id`
6. Check no existing user with this `phone` + `company_id`
7. Create `User` record with data from `InviteToken` + `telegram_user_id` from challenge:
   - `first_name`, `last_name`, `phone`, `role`, `permissions`, `permission_group_id` from InviteToken
   - `telegram_user_id` from challenge
   - `company_id` from InviteToken
   - `email` = NULL
   - `password_hash` = NULL
   - `is_active` = True
   - `email_verified` = True (no email to verify)
8. Mark `InviteToken` as used (`used_at = now`, `used_by = new_user.id`)
9. Mark `TelegramAuthChallenge` as used
10. Generate JWT credentials (access 24h, refresh 30d)
11. Return user + credentials

**Rate limit:** 5/minute per IP

#### Error Responses

| Code | When | Body |
|---|---|---|
| 400 | Session not found or expired | `{ "detail": "Invalid or expired registration session" }` |
| 400 | Session already used | `{ "detail": "Invalid or expired registration session" }` |
| 400 | Invite token invalid or expired | `{ "detail": "Invalid or expired invite token" }` |
| 400 | Invite token already used | `{ "detail": "This invite token has already been used" }` |
| 409 | Telegram account already linked to a user | `{ "detail": "This Telegram account is already registered" }` |
| 409 | Phone already exists in company | `{ "detail": "A user with this phone number already exists" }` |

---

### 9. Bot Handler: `/start login_{CHALLENGE_ID}`

This is NOT an HTTP endpoint. This is a Telegram bot handler for the deep link.

**When bot receives `/start login_{CHALLENGE_ID}`:**

1. Parse `CHALLENGE_ID` from the payload
2. Look up `TelegramAuthChallenge` by `id = CHALLENGE_ID` where `purpose = "login"` and not expired and not used
3. If not found or expired, reply: "This login link has expired. Please request a new one from the login page."
4. Look up `User` where `telegram_user_id = message.from_user.id` AND `company_id = challenge.company_id` AND `deleted_at IS NULL`
5. If no user found, reply: "No account found for this Telegram account in this company."
6. If user is suspended or inactive, reply: "Your account is suspended. Contact your administrator."
7. Check rate limit: no OTP sent for this `user_id` in last 60 seconds. If violated, reply: "Please wait before requesting another code."
8. Check lockout: if user has 5+ failed OTP verifications in last 15 minutes, reply: "Account temporarily locked. Try again later."
9. Generate 6-digit OTP (numeric, leading zeros allowed, e.g. `042918`)
10. Hash OTP with SHA-256, store in `challenge.otp_hash`
11. Set `challenge.telegram_user_id = message.from_user.id`
12. Set `challenge.user_id = user.id`
13. Set `challenge.expires_at = now + 5 minutes` (tighten from creation time)
14. Reply to user:

```
Your login code: 472918

This code expires in 5 minutes. Enter it on the login page.
```

---

### 10. Refresh Token

**No changes to the existing `/api/v1/auth/refresh` endpoint.** It already works with JWT tokens. The only change is the token duration:

- For Telegram-auth users, access tokens are 24h (vs current 30min for password-auth)
- Refresh tokens remain 30 days

This is handled at token creation time, not at refresh time. The refresh endpoint just issues a new access token with the same duration as the original.

**Important:** The `JWTManager.generate_credentials` call must pass `access_duration=timedelta(hours=24)` for Telegram-auth flows.

---

### 11. Logout

**No changes to the existing `/api/v1/auth/logout` endpoint.** Cookie clearing works the same way.

---

## Bot Command Summary

| Command/Trigger | Handler | What It Does |
|---|---|---|
| `/register` | New handler | Creates register challenge, sends deep link to website |
| `/start login_{ID}` | Modify existing `/start` handler | Validates challenge, sends OTP via DM |
| `/start` (no payload) | Keep existing | Shows Chat ID for notification setup |

---

## Business Rules

### Invite Token Format
- Charset: `A-Z` + `2-9` (33 chars total, excludes `0`, `O`, `1`, `I`, `L`)
- Format: `XXXX-XXXX` (8 chars + dash separator, dash is cosmetic and stripped before hashing)
- Entropy: 33^8 = ~1.4 trillion combinations
- Generation: `secrets.choice()` from charset, NOT predictable
- Storage: SHA-256 hex hash of the 8 alphanumeric chars (strip dash before hashing, uppercase)
- Expiry: 48 hours from creation
- Single-use: once used, `used_at` is set and token cannot be reused

### OTP Rules
- 6 digits, `000000`-`999999` (leading zeros valid)
- Generated with `secrets.randbelow(1_000_000)`, zero-padded to 6 digits
- Stored as SHA-256 hex hash
- Expires: 5 minutes from generation
- Max attempts: 3 per challenge (after 3 wrong codes, challenge is burned)
- Rate limit: 1 OTP per 60 seconds per user (enforced in bot handler)
- Lockout: 5 failed OTP verifications in 15 minutes -> 30 minute cooldown per user
- Single-use: burned after successful verification

### Auth Hierarchy
- **Owner** (`RoleEnum.OWNER`): email + password only. No Telegram auth. Existing `/api/v1/owner/auth/*` endpoints unchanged.
- **Company users** (Admin, Manager, Operator): Telegram OTP only. The old `/api/v1/auth/login` (email+password) endpoint remains for backward compatibility during migration but will be deprecated.

### JWT Token Durations (Telegram auth)
- Access token: 24 hours
- Refresh token: 30 days
- httpOnly cookies (same `_set_auth_cookies` helper)

### Multi-Tenant Isolation
- Every `InviteToken` and `TelegramAuthChallenge` is scoped to `company_id`
- User lookup for login always filters by `telegram_user_id + company_id`
- A single Telegram account CAN be linked to users in different companies (one user per company)
- A single Telegram account CANNOT be linked to multiple users in the same company

### User Creation via Invite
- User record is NOT created at invite time. Only `InviteToken` is created.
- User record is created at registration completion (endpoint 8).
- This means the admin "invite" flow changes: admin no longer sees the user in the user list until registration completes.
- The invite tokens list (endpoint 2) shows pending invites.

---

## Migration Notes

### Alembic Migration Required

1. **Alter `users` table:**
   - Add `telegram_user_id` column (BigInteger, nullable, unique, indexed)
   - Make `email` nullable (ALTER COLUMN SET NULL)
   - Make `password_hash` nullable (ALTER COLUMN SET NULL)
   - Make `phone` NOT NULL (need to backfill NULLs first -- set to empty string or prompt admin)
   - Add `uq_company_phone` unique constraint on (`company_id`, `phone`)

2. **Create `invite_tokens` table**

3. **Create `telegram_auth_challenges` table**

**Migration ordering matters.** The phone backfill must happen before the NOT NULL constraint.

### Backward Compatibility
- Existing users with email+password continue to work via `/api/v1/auth/login`
- The old invite flow (`POST /api/v1/company/users/invite`) remains functional during transition
- New Telegram invite flow is a separate endpoint (`/api/v1/company/users/invite-telegram`)
- Frontend should detect which auth method to show based on presence of `telegram_user_id` on the user

---

## Frontend Implementation Notes

### New Pages

1. **`/auth/register`** — Public page. Shows form with single input for XXXX-XXXX token. Reads `?s=REG_SESSION_ID` from URL query param. On submit, calls endpoint 8.

2. **`/login` (modified)** — Existing login page gets a new mode for Telegram auth:
   - Show "Log in via Telegram" button + 6-digit OTP input below it
   - Button click -> call endpoint 4 -> open deep link in new tab/window -> start polling endpoint 5
   - When status becomes `otp_sent`, focus the OTP input
   - User enters 6 digits -> call endpoint 6
   - Keep existing email+password form as fallback (hidden behind "Use email instead" link during transition)

### API Client Additions (`src/lib/api.ts`)

```typescript
// Telegram Auth
async createLoginChallenge() {
  const response = await this.client.post('/api/v1/auth/telegram/login-challenge');
  return response.data;
}

async pollChallengeStatus(challengeId: string) {
  const response = await this.client.get(`/api/v1/auth/telegram/login-challenge/${challengeId}/status`);
  return response.data;
}

async verifyOtp(data: { challenge_id: string; otp: string }) {
  const response = await this.client.post('/api/v1/auth/telegram/verify-otp', data);
  // save tokens same as existing login
  if (response.data.credentials) {
    this.saveTokens(response.data.credentials);
  }
  this.saveUserType('company_user');
  return response.data;
}

async telegramRegister(data: { session_id: string; invite_token: string }) {
  const response = await this.client.post('/api/v1/auth/telegram/register', data);
  if (response.data.credentials) {
    this.saveTokens(response.data.credentials);
  }
  this.saveUserType('company_user');
  return response.data;
}

// Invite Tokens (admin)
async createInviteToken(data: InviteTokenCreateRequest) {
  const response = await this.client.post('/api/v1/company/users/invite-telegram', data);
  return response.data;
}

async getInviteTokens(params?: { page?: number; page_size?: number; status?: string }) {
  const response = await this.client.get('/api/v1/company/users/invite-tokens', { params });
  return response.data;
}

async revokeInviteToken(tokenId: string) {
  await this.client.delete(`/api/v1/company/users/invite-tokens/${tokenId}`);
}
```

### New Types (`src/types/api.ts`)

```typescript
export interface InviteTokenCreateRequest {
  first_name: string;
  last_name?: string | null;
  phone: string;
  role?: string;
  permissions?: string[];
  permission_group_id?: string | null;
}

export interface InviteTokenResponse {
  invite_token: string;
  expires_at: string;
  role: string;
  first_name: string;
  phone: string;
}

export interface InviteTokenListItem {
  id: string;
  role: string;
  first_name: string;
  last_name: string | null;
  phone: string;
  created_by_name: string;
  expires_at: string;
  used_at: string | null;
  created_at: string;
  status: 'pending' | 'used' | 'expired';
}

export interface LoginChallengeResponse {
  challenge_id: string;
  deep_link: string;
  expires_at: string;
}

export interface ChallengeStatusResponse {
  status: 'pending' | 'otp_sent' | 'expired' | 'used';
  expires_at: string;
}

export interface VerifyOtpRequest {
  challenge_id: string;
  otp: string;
}

export interface TelegramRegisterRequest {
  session_id: string;
  invite_token: string;
}
```

### Polling Strategy for Challenge Status
- Poll every 2 seconds
- Max poll duration: 5 minutes (matches challenge expiry)
- Stop polling when status is `otp_sent`, `expired`, or `used`
- On `expired`, show "Link expired" message with button to retry

### AuthorizedResponse Schema Change
The existing `AuthorizedResponse` has `email: str`. This must become `email: str | null` since Telegram-auth users may not have an email. Also add `phone: str | null` to the response.

---

## Backend Implementation Notes

### File Structure

New files:
- `source/db/models/invite_token.py`
- `source/db/models/telegram_auth_challenge.py`
- `source/api/v1/routers/auth/telegram_auth.py` — new router for endpoints 4, 5, 6, 8
- `source/api/v1/schemas/telegram_auth.py` — request/response schemas for Telegram auth
- `source/utils/services/invite_token_service.py` — token generation, hashing, validation
- `alembic/versions/xxx_add_telegram_auth.py` — migration

Modified files:
- `source/db/models/user.py` — add `telegram_user_id`, make `email`/`password_hash` nullable
- `source/db/models/enums.py` — add `ChallengePurposeEnum`
- `source/db/models/__init__.py` — register new models
- `source/api/v1/routers/auth/__init__.py` — mount telegram_auth router
- `source/api/v1/routers/company/users.py` — add invite-telegram endpoint + invite-tokens list/revoke
- `source/api/v1/routers/telegram_webhook.py` — add `/start login_{ID}` and `/register` handlers
- `source/api/v1/schemas/auth.py` — make email nullable in AuthorizedResponse, add phone
- `source/api/v1/schemas/user.py` — make email optional in UserInviteRequest (for telegram flow), add phone to required fields
- `source/core/config.py` — add `TELEGRAM_BOT_USERNAME` env var

### Challenge ID Generation
Use `secrets.token_urlsafe(16)` (22 chars, URL-safe). Not a UUID — needs to be short for deep links.

### SHA-256 Hashing
```python
import hashlib

def hash_token(token: str) -> str:
    """Hash a token (invite code or OTP) with SHA-256."""
    return hashlib.sha256(token.encode()).hexdigest()
```

For invite tokens, strip the dash and uppercase before hashing: `hash_token(raw.replace("-", "").upper())`

### Rate Limiting for OTP
Use existing Redis-based rate limiting (same as `is_locked`/`record_failed_login` pattern). Keys:
- `otp_rate:{user_id}` — TTL 60s, for 1-per-minute OTP generation limit
- `otp_lockout:{user_id}` — counter with 15min window, lockout at 5 failures for 30min

### Router Mounting
The telegram auth endpoints should be mounted under the existing auth router prefix:
- `POST /api/v1/auth/telegram/login-challenge`
- `GET /api/v1/auth/telegram/login-challenge/{challenge_id}/status`
- `POST /api/v1/auth/telegram/verify-otp`
- `POST /api/v1/auth/telegram/register`

The invite management endpoints stay under company:
- `POST /api/v1/company/users/invite-telegram`
- `GET /api/v1/company/users/invite-tokens`
- `DELETE /api/v1/company/users/invite-tokens/{token_id}`

### Cleanup Job
Add a periodic task (or DB trigger) to delete expired, unused `TelegramAuthChallenge` rows older than 1 hour. Invite tokens should be kept for audit trail (soft-expire, don't delete).

---

## Security Considerations

1. **Invite tokens** are high-entropy (33^8) and hashed. Even if DB is compromised, attacker cannot reconstruct valid tokens.
2. **OTPs** are hashed with SHA-256. Even if DB is compromised mid-session, attacker cannot read the OTP.
3. **Challenge IDs** are `secrets.token_urlsafe(16)` — 128 bits of entropy. Not guessable.
4. **Rate limits** on all public endpoints prevent brute force.
5. **Lockout** after 5 failed OTPs protects against targeted attacks.
6. **Single-use** tokens and OTPs prevent replay attacks.
7. **Origin header** validation for company resolution prevents cross-tenant attacks.
8. **Telegram user_id** is verified by the bot receiving the message — it cannot be spoofed in the Telegram API.
