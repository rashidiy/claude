# Backend Reference — Project Structure, Models, Examples

Load this when you need project structure details, code examples, or model information.

---

## Tech Stack

- **Python 3.12** + **FastAPI** (async)
- **SQLAlchemy 2.0** (async, declarative models)
- **PostgreSQL 16** (primary database)
- **Redis 7** (caching, session store)
- **Alembic** (database migrations)
- **Pydantic v2** (request/response validation)
- **aiohttp** (async HTTP client for external APIs)
- **passlib + bcrypt** (password hashing)
- **PyJWT** (token management)
- **pytest + pytest-asyncio + httpx** (testing)
- **Docker Compose** (development and production)

---

## Environment Variables

Required: `POSTGRES_HOST`, `POSTGRES_PORT`, `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, `JWT_SIGNING_KEY`, `REDIS_URL`
Optional: `WEBHOOK_IP_WHITELIST_ENABLED`, `SIPUNI_ALLOWED_IPS`, `BINOTEL_ALLOWED_IPS`, `BASE_URL`, `JWT_ALGORITHM`, `RECORD_PROXY_SECRET`, `RECORD_PROXY_TOKEN_EXPIRY`
Template: `.env.copy`

---

## Swagger / API Docs

Swagger UI is split by audience at `/swagger`:
- `/swagger?type=owner` — Owner endpoints (`/api/v1/owner/*`)
- `/swagger?type=company` — Company user endpoints (`/api/v1/auth/*` + `/api/v1/company/*`)

## CORS

CORS regex in `main.py`: `allow_origin_regex=r"https?://[\w-]+\.(s1p\.com|localhost)(:\d+)?"`. Credentials, all methods, all headers allowed.

## CallEvent ID Pattern

`CallEvent.id` is Integer (not UUID), using company-scoped sequential numbering via `next_call_number()` helper. The `provider_call_id` field is prefixed with the provider name (e.g., `sipuni_<id>`).

---

## Project Structure

```
repos/s1p-backend/
  main.py                              → FastAPI app initialization, CORS, middleware, router mounts
  source/
    api/v1/
      routers/
        auth/
          router.py                    → Company user auth (login, refresh, set-password, forgot, reset)
        owner/
          router.py                    → Owner auth
          companies.py                 → Company CRUD (create, list, update, activate, deactivate, invite-admin)
          contracts.py                 → Contract CRUD (create, renew, cancel)
          analytics.py                 → Platform-wide analytics
          permissions.py               → Permission listing
        company/
          users.py                     → User management (invite, list, update, activate)
          contacts.py                  → Contact CRUD + bulk import + activity timeline
          leads.py                     → Lead CRUD + convert + assign
          deals.py                     → Deal CRUD + win/lose + pipeline summary
          tasks.py                     → Task CRUD + complete + my-today
          notes.py                     → Note CRUD + entity timeline
          calls.py                     → Call history, outcomes, linking, auto-link suggestions
          sipuni_calls.py              → Sipuni-specific call actions
          binotel_calls.py             → Binotel-specific call actions
          recordings.py                → Call recording proxy streaming
          webhooks.py                  → Inbound provider webhooks (Sipuni, Binotel)
          analytics.py                 → Operator/team analytics
          permission_groups.py         → Permission group CRUD
          contract.py                  → Company contract status
      schemas/
        auth.py                        → Auth request/response schemas
        owner.py                       → Owner schemas
        company.py                     → Company schemas (users, contacts, leads, deals, tasks, notes, calls, analytics)
    db/
      models/
        owner.py                       → Owner model
        company.py                     → Company model (provider_type, provider_config JSONB)
        user.py                        → User model (role enum, permissions JSONB, permission_group)
        call_event.py                  → CallEvent model (provider-agnostic, linked to contact/lead/deal)
        contact.py                     → Contact model (custom_fields JSONB)
        lead.py                        → Lead model (status enum, custom_fields JSONB)
        deal.py                        → Deal model (stage enum, custom_fields JSONB)
        task.py                        → Task model (status, priority, entity linking)
        note.py                        → Note model (polymorphic entity_type + entity_id)
        tag.py                         → Tag model
        contract.py                    → Contract model (billing, limits, status)
        permission_group.py            → PermissionGroup model
        audit_log.py                   → AuditLog model (action tracking)
      mixins/
        crud.py                        → Generic async CRUD operations
        auth.py                        → Auth helper mixins
    core/
      config.py                        → DatabaseConfig, JWTConfig, SMTPConfig, WebhookConfig
    utils/
      services/
        telephony/
          base.py                      → Abstract TelephonyProvider class
          sipuni.py                    → SipuniProvider (MD5 auth, cabinet-based API)
          binotel.py                   → BinotelProvider (API key + secret)
          factory.py                   → ProviderFactory.create(type, config)
          http_client.py               → HTTPClientPool (aiohttp, connection pooling, retries, DNS cache)
      services/
        analytics.py                   → AnalyticsService (SQL aggregation, Redis caching)
        cache.py                       → CacheService (Redis + in-memory fallback)
        email.py                       → EmailService (SMTP, background tasks)
      managers/
        password.py                    → PasswordManager (bcrypt)
        jwt.py                         → JWTManager (create, validate, refresh tokens)
      tasks/
        email_tasks.py                 → Background email sending
      validators/
        phone.py                       → Uzbek phone validation (998xx)
    alembic/                           → 6 migration files
    tests/                             → 14 test files, ~2000 LOC
```

---

## Existing Models (16 Total)

| Model | Key Fields | Multi-Tenant |
|---|---|---|
| Owner | id (UUID), email, password_hash, first_name, last_name, phone, is_active | No (platform-level) |
| Company | id (UUID), owner_id, name, subdomain, provider_type, provider_config (JSONB), webhook_token, is_active | Yes (owner's tenant) |
| User | id (UUID), company_id, email, role (4 levels), permissions (JSONB), permission_group_id, language | Yes |
| CallEvent | id (Integer, per-company), company_id, provider_type, phone_1, phone_2, operator_id, direction, state, record_url, contact_id, lead_id, deal_id, outcome | Yes |
| Contact | id (UUID), company_id, first_name, last_name, phone, email, custom_fields (JSONB), assigned_to | Yes |
| Lead | id (UUID), company_id, contact_id, title, status (5 states), custom_fields (JSONB), assigned_to | Yes |
| Deal | id (UUID), company_id, lead_id, contact_id, title, stage (6 states), amount, custom_fields (JSONB), assigned_to | Yes |
| Task | id (UUID), company_id, title, status, priority, due_date, assigned_to, entity_type, entity_id | Yes |
| Note | id (UUID), company_id, content, entity_type, entity_id, created_by | Yes |
| Tag | id (UUID), company_id, name, color | Yes |
| Contract | id (UUID), owner_id, company_id, max_admins/managers/operators, price, status, billing_period | Hybrid |
| PermissionGroup | id (UUID), company_id, name, permissions (JSONB), is_system | Yes |
| AuditLog | id (UUID), company_id, user_id, action, entity_type, entity_id, before/after (JSONB) | Yes |

---

## Code Examples

### Multi-Tenant Query
```python
query = select(Contact).where(
    Contact.company_id == current_user.company_id,
    Contact.deleted_at.is_(None)
)
```

### Permission Decorator
```python
@router.get("/")
@require_permissions("contacts.read")
async def list_contacts(
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    ...
```

### Pydantic Schema
```python
class ContactCreate(BaseModel):
    first_name: str
    last_name: str | None = None
    phone: str
    email: str | None = None
    company_name: str | None = None

class ContactResponse(BaseModel):
    id: UUID
    first_name: str
    last_name: str | None
    phone: str
    email: str | None
    company_name: str | None
    created_at: datetime

    model_config = ConfigDict(from_attributes=True)
```

### Async DB Operations
```python
async with db.begin():
    result = await db.execute(query)
```

### Error Handling
```python
raise HTTPException(status_code=404, detail="Contact not found")
raise HTTPException(status_code=403, detail="Not authorized to access this resource")
raise HTTPException(status_code=409, detail="Email already exists in this company")
```

### Background Tasks
```python
@router.post("/")
async def create_something(background_tasks: BackgroundTasks):
    # ... create the thing ...
    background_tasks.add_task(send_notification, data)
    return response
```

### Test Pattern
```python
class TestFeatureName:
    @pytest.mark.asyncio
    async def test_happy_path(self, client: AsyncClient, ...fixtures):
        response = await client.get("/api/v1/company/...", headers=auth_headers)
        assert response.status_code == 200
        data = response.json()
        assert data["field"] == expected_value

    @pytest.mark.asyncio
    async def test_unauthorized(self, client: AsyncClient):
        response = await client.get("/api/v1/company/...")
        assert response.status_code == 401

    @pytest.mark.asyncio
    async def test_tenant_isolation(self, client: AsyncClient, other_company_headers):
        response = await client.get("/api/v1/company/...", headers=company_b_headers)
        assert response.status_code in [403, 404]
```
