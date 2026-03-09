# Backend — Lessons Learned

Read before every task. Write after every mistake, decision, or pattern discovery.

---

## Always Include company_id in Queries

**Source:** Audit (2026-03-08)
**What happened:** Found queries missing company_id filter — would have leaked tenant data.
**Lesson:** Every new query must include company_id. Treat missing filters as security breaches.

## Don't Forget Response Schemas

**Source:** Pre-prod audit (2026-03-08)
**What happened:** Model + router updated but Pydantic response schema forgotten. Data stored but invisible to API.
**Lesson:** When adding columns: update model, router, AND response schema. All three.

## Always Add Migration With Model Changes

**Source:** Pre-prod audit (2026-03-08)
**Lesson:** Every model change needs an Alembic migration. No exceptions.
