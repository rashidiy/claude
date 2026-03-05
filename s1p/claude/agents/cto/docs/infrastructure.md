# Infrastructure & Security

**Source:** CTO Brain v100
**Last updated:** 2026-03-03

---

## Phase 1 Monthly Infrastructure Cost (Target)

| Service | Cost | Notes |
|---|---|---|
| VPS (app + DB) | $20-40/month | Hetzner or Timeweb (popular in CIS, accepts UZS) |
| Domain + SSL | $15/year | Let's Encrypt for SSL |
| Telegram Bot API | Free | No cost |
| Email (SMTP) | $0-10/month | Brevo free tier (300 emails/day) or Mailgun |
| Monitoring | Free | Uptime Kuma (self-hosted) or free tier of BetterUptime |
| **Total** | **~$40-60/month** | |

**Gross margin at 100 clients:** Revenue $2,900/month - Infra $60/month = **98% gross margin.** This is SaaS economics at their best. Protect this margin.

---

## Scaling Triggers

| Clients | Action needed |
|---|---|
| 1-100 | Single VPS handles everything |
| 100-500 | Separate DB server, add Redis instance |
| 500-1000 | Load balancer, 2 app servers, managed PostgreSQL |
| 1000+ | Kubernetes or managed containers, read replicas, CDN |

Don't over-provision. Start small. Scale when it hurts, not before.

---

## Security Non-Negotiables

| Requirement | Why | Implementation |
|---|---|---|
| Tenant isolation in every query | One company must NEVER see another's data | company_id filter on every DB query, automated tests verify this |
| Call recordings are sensitive | Legal liability if leaked | Stream from provider, never store locally in Phase 1. HTTPS only. |
| API keys must be hashed | If DB is breached, keys are useless | Hash with bcrypt, show full key only once at creation |
| Rate limiting on public API | Prevent abuse, protect infrastructure | Token bucket per API key, 100 req/min default |
| Webhook signatures | Prevent spoofing | HMAC-SHA256 with per-company secret |
| Uzbek data protection | UZ has data localization discussions ongoing | Monitor regulatory landscape. Be ready to host in-country if required. |
| No PII in logs | Compliance basic | Sanitize phone numbers, emails from application logs |

---

## Technical Debt Budget

**Phase 1 is allowed to accumulate debt in these areas:**
- Frontend polish (CSS, animations, responsive edge cases) — fix in Phase 1.5
- Test coverage below 80% — acceptable for launch, improve weekly
- No automated deployment pipeline — manual deploy is fine for < 100 clients
- Email templates are plain text, not beautiful HTML — fix in week 2
- Error messages are in English only — localize in week 2
- No rate limiting on internal API — add before 100+ clients

**Phase 1 must NOT accumulate debt in these areas:**
- Multi-tenant isolation — ZERO tolerance for tenant data leaks
- Auth/JWT security — no shortcuts
- Data model design — custom fields JSONB structure must be right from day one
- Public API contract — clients will integrate, breaking changes are expensive
- Telegram bot reliability — missed notifications = lost trust
