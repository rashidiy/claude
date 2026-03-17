# Contract: User Avatar

## Overview
Add persistent avatar storage. Download Telegram photos on registration/login, allow custom uploads, show avatars in teams page with initials fallback.

## Backend

### DB Migration
Add to `users` table:
- `avatar` VARCHAR(255), nullable — stored filename (e.g., `{user_id}.jpg`)
- `avatar_is_custom` BOOLEAN, NOT NULL, DEFAULT false — true if user uploaded manually

### User Model Changes
Add two fields: `avatar` and `avatar_is_custom` to `User` model.

### Schema Changes
**UserResponse** — add:
- `avatar_url: Optional[str]` — computed property, returns `{BASE_URL}/media/avatars/{avatar}` if avatar is set, else null

### Static File Serving
- Create directory: `source/static/media/avatars/`
- Mount in `main.py`: `app.mount("/media", StaticFiles(directory="source/static/media"), name="media")`

### Avatar Utility
Create `source/utils/services/avatar_service.py`:
- `async def download_telegram_avatar(user_id: str, telegram_user_id: int) -> str | None`
  - Uses aiogram Bot to get profile photos
  - Downloads file via httpx from Telegram API
  - Saves to `source/static/media/avatars/{user_id}.jpg`
  - Returns filename or None on failure
- `def save_uploaded_avatar(user_id: str, file_bytes: bytes, content_type: str) -> str`
  - Saves to `source/static/media/avatars/{user_id}.jpg`
  - Returns filename
- `def delete_avatar(user_id: str) -> None`
  - Removes file from disk if exists

### Modified Endpoints

**POST /api/v1/auth/telegram/register** (telegram_auth.py)
- Add `skip_avatar: bool = False` to `TelegramRegisterRequest`
- After user creation: if `telegram_avatar_file_id` exists and NOT `skip_avatar`:
  - Call `download_telegram_avatar(user.id, telegram_user_id)`
  - Set `user.avatar = filename`

**POST /api/v1/webhooks/telegram/{secret}** — login handler (company/telegram_webhook.py)
- In `_handle_login_start`, after OTP generation:
  - If `user.avatar_is_custom is False`:
    - Fetch Telegram profile photo
    - Download and save (overwrite existing)
    - Update `user.avatar` and `user.telegram_avatar_file_id`

### New Endpoints (in company/users.py)

**POST /api/v1/company/users/me/avatar**
- Content-Type: multipart/form-data
- Accepts: `file` (UploadFile, max 5MB, JPEG/PNG/WebP)
- Saves file, sets `user.avatar = filename`, `user.avatar_is_custom = True`
- Returns: `UserResponse` (200)

**DELETE /api/v1/company/users/me/avatar**
- Removes file from disk
- Sets `user.avatar = None`, `user.avatar_is_custom = False`
- Returns: `UserResponse` (200)

---

## Frontend

### Types (src/types/api.ts)
Add to UserResponse interface extension (since UserResponse comes from generated types, add a manual extension):
- `avatar_url?: string | null`

### API Client (src/lib/api.ts)
Add methods:
- `uploadAvatar(file: File): Promise<UserResponse>` — POST multipart to `/users/me/avatar`
- `deleteAvatar(): Promise<UserResponse>` — DELETE `/users/me/avatar`

### Registration Page (src/app/register/page.tsx)
Step 3 changes:
- Replace the green status bar with an interactive avatar area:
  - Show Telegram avatar (from proxy URL) or empty state
  - **Trash icon**: small overlay button on avatar circle to remove photo (sets state to "none")
  - **Click on avatar**: opens hidden file input for custom upload
  - Track state: `avatarState: 'telegram' | 'custom' | 'none'`
  - Track custom file: `customAvatarFile: File | null`
- On submit:
  - Call `telegramRegister({ ..., skip_avatar: avatarState === 'none' || avatarState === 'custom' })`
  - If `avatarState === 'custom'` and `customAvatarFile`: call `apiClient.uploadAvatar(customAvatarFile)` after registration

### Users Page (src/app/(company)/users/page.tsx)
- Replace the `<TeamOutlined>` icon in user cards with:
```tsx
{user.avatar_url ? (
  <img src={user.avatar_url} className="h-10 w-10 rounded-full object-cover" />
) : (
  <div className="h-10 w-10 rounded-full bg-crm-indigo-100 flex items-center justify-center text-crm-indigo-600 font-semibold text-sm">
    {user.first_name?.[0]?.toUpperCase()}{user.last_name?.[0]?.toUpperCase() || ''}
  </div>
)}
```
