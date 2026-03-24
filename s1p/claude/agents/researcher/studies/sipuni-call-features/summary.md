# Sipuni Call Features — What You're Not Seeing

**Date:** 2026-03-19
**Question:** What Sipuni capabilities exist that S1P isn't using? Where are the revenue opportunities?
**Account:** Professional plan (highest tier), valid until 2026-05-26, 5 licensed users

---

## TL;DR

We're using **~30% of Sipuni's API surface**. The remaining 70% contains features that directly map to CRM revenue: live call monitoring, operator status management, call quality ratings, auto-dialing campaigns, real-time WebSocket events, SMS routing, and speech analytics. Most of these require zero additional Sipuni cost — they're already included in our Professional plan.

---

## What We Currently Use

| Feature | API | Status |
|---|---|---|
| Click-to-call | `/api/callback/call_number` | Done |
| Call via IVR scheme | `/api/callback/call_tree` | Done |
| Bridge two externals | `/api/callback/call_external` | Done |
| Cancel callback | `/api/callback/cancel` | Done |
| Webhook events (event=2 only) | GET webhook | Done |
| Call recording URL | From webhook `call_record_link` | Done |
| One-click setup | Dashboard scraping | Done |

---

## What We're NOT Using — Ranked by CRM Revenue Impact

### TIER 1: HIGH VALUE, LOW EFFORT (build next)

**1. Real-time Call Events via WebSocket** `wss://wss.sipuni.com/api`
- Currently we only process event=2 (hang-up). We ignore event=1 (call start), event=3 (answer), event=4 (transfer hang-up)
- **What this unlocks:** Live call popups in CRM UI ("Incoming call from +7916... — Contact: Ivan Petrov"), call transfer tracking, live call duration counters
- **Revenue impact:** This is what makes Bitrix24 feel "alive". Without it, our CRM feels disconnected from the phone.
- **Effort:** Medium — need WebSocket client service + frontend real-time updates (SSE or polling)

**2. Operator Status Management** `GET /api/workplace/setUserStatus`
- Set operator status: Free (0), Break (1), Auto-dialing (2), Manual dialing (3), Away (4)
- Also: `GET /api/statistic/operators` returns current presence/status of all employees
- **What this unlocks:** Show operator availability in CRM, let managers set agent statuses, auto-set status during lunch/breaks
- **Revenue impact:** Call center managers NEED this. It's a basic expectation from any telephony-CRM integration.
- **Effort:** Low — two simple API calls + UI widget

**3. Call Hangup API** `POST /api/events/call/hangup`
- Terminate ANY active call by callId (not just callbacks we initiated)
- **What this unlocks:** "End call" button in CRM interface during live calls
- **Revenue impact:** Pairs with real-time events for full call control from CRM
- **Effort:** Very low — single endpoint

**4. Who Ended the Call (hangupinitor)** — Statistics Export parameter
- `hangupinitor` field tells you WHO terminated the call:
  - 2 = operator hung up
  - 3 = client hung up
  - 4 = busy
  - 5 = no answer
  - 6 = phone off/unreachable
  - 7 = carrier error
- **What this unlocks:** "Client hung up after 3 seconds" vs "Operator ended 45-min call" — completely different analytics stories
- **Revenue impact:** Phase 2 AI scoring needs this data. Start collecting NOW.
- **Effort:** Very low — add parameter to statistics export

**5. First-Time Caller Detection (firstTime)** — Statistics Export parameter
- Filter `firstTime=1` returns only calls from numbers never seen before
- **What this unlocks:** Auto-flag new leads, "New Customer" badge in Telegram notification, separate handling rules
- **Revenue impact:** Core CRM feature — knowing new vs returning callers
- **Effort:** Very low — flag in webhook processing

### TIER 2: HIGH VALUE, MEDIUM EFFORT

**6. Spy Mode / Call Whispering** `POST /api/events/call/spymode`
- `whisper=0` → silent listening (supervisor hears everything)
- `whisper=1` → coaching (supervisor speaks to agent only, client can't hear)
- **What this unlocks:** "Listen to live call" and "Coach agent" buttons in CRM
- **Revenue impact:** Premium call center feature. This is what Professional plan customers pay for. Our CRM can expose it.
- **Effort:** Medium — needs real-time call tracking (depends on #1) + UI

**7. Call Quality Ratings (Rating)** — Statistics Export parameter
- Filter by rating 0-9, supports ranges like "2-7"
- Quality rating is a post-call survey feature in Sipuni
- **What this unlocks:** Customer satisfaction scores per call, per operator, per team
- **Revenue impact:** Direct input to operator scoring (Phase 2). Managers love this dashboard.
- **Effort:** Low-medium — need to enable in Sipuni settings + add to call data model

**8. Schema/Routing List** `POST /api/schema/list`
- Get all routing schemes (incoming/outgoing) with their IDs
- **What this unlocks:** Show which IVR scheme handled the call, analytics per scheme, configure call routing from CRM
- **Revenue impact:** Operational visibility for managers
- **Effort:** Low

### TIER 3: MEDIUM VALUE, HIGHER EFFORT

**9. SMS Gateway Integration**
- Route all Sipuni SMS through our custom gateway
- Receives: number, sender, text via POST
- **What this unlocks:** SMS notifications to clients (appointment reminders, follow-ups), SMS conversations in CRM
- **Revenue impact:** Medium — SMS is dying in CIS (Telegram dominates), but still valuable for certain workflows
- **Effort:** Medium — need SMS gateway partner + integration

**10. Auto-Dialing Campaigns**
- Create auto-dial campaigns from CRM, import contact lists
- Three modes: Standard, Predictive, Informer (automated message)
- **What this unlocks:** Outbound campaign management from CRM, auto-dialing for sales teams
- **Revenue impact:** High for outbound sales teams, but complex to implement
- **Effort:** High — campaign management UI + webhook data processing

**11. Yandex SpeechSense Integration**
- Available in new cabinet as a connected service
- Speech recognition and analysis
- **What this unlocks:** Automated transcription, call analysis (Phase 2 accelerator)
- **Revenue impact:** Directly enables Phase 2 features WITHOUT building our own STT
- **Effort:** Medium — research their API, integrate with our pipeline

---

## Data We Should Be Collecting But Aren't

| Field | Source | Why It Matters |
|---|---|---|
| `hangupinitor` | Statistics API | Who ended the call — critical for AI scoring |
| `firstTime` | Statistics API | New vs returning callers — lead qualification |
| `Rating` | Statistics API | Post-call satisfaction scores |
| `numbersRinged` | Statistics API | Which agents were tried before someone answered |
| `numbersInvolved` | Statistics API | Who actually talked (for multi-party calls) |
| `outgoingLine` | Statistics API | Which phone line was used for outgoing calls |
| `dtmfUserAnswer` | Statistics API | Customer key presses during IVR |
| `treeName/treeNumber` | Webhook events | Which routing scheme handled the call |
| `transfer_from` | Webhook events | Who transferred the call |
| `last_called` | Webhook events | Last agent in the routing chain |
| `roistat` / `roistat_number` | Webhook events | Call tracking visitor data |
| Event 1 (call start) | Webhook | When the call begins — for live status |
| Event 3 (answer) | Webhook | When someone picks up — for ring time calculation |
| Event 4 (transfer hangup) | Webhook | Transfer tracking |
| `call_answer_timestamp` | Webhook event 2 | When answered (0 if unanswered) — for wait time |

---

## Statistics We Can Build But Don't

From the new cabinet (lk.sipuni.com) statistics page, Sipuni already shows:

**Per-Number Analytics:**
- Total calls, Answered, Active, New numbers, Different numbers
- Total duration, Talk duration, Avg call duration, Avg talk duration
- Avg answer time, Answer rate, Active rate

**Per-User Analytics:**
- Same metrics as above, plus activity history

**We should mirror these in our CRM dashboard** — users expect parity with what Sipuni already shows them.

---

## Recommended Build Order

1. **Collect missing webhook data** (events 1,3,4 + all fields) — 1 day, zero risk
2. **Add hangupinitor + firstTime to statistics** — 1 day
3. **Operator status widget** (get + set) — 2 days
4. **Call hangup button** — 1 day
5. **Real-time call events via WebSocket** — 3-5 days (game-changer)
6. **Schema list + per-scheme analytics** — 1 day
7. **Spy mode / whispering** — 2-3 days (premium feature)
8. **Call quality ratings** — 2 days
9. **Yandex SpeechSense research** — 1 day research, Phase 2 enabler

**Total: ~15-20 days to go from 30% to 90% Sipuni coverage.**

---

## Decision Needed from Founder

1. **Real-time events (WebSocket)** — This is the single biggest gap. Do we add WebSocket support now (Phase 1) or defer to Phase 2? It fundamentally changes the UX.
2. **Yandex SpeechSense** — This could fast-track Phase 2 AI. Worth investigating now?
3. **Auto-dialing** — Big feature, big effort. Is outbound dialing a priority for our target customers?
