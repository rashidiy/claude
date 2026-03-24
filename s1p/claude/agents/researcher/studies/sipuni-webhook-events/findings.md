# Sipuni Webhook Events — Live Testing Results

**Date:** 2026-03-24
**Tested by:** Founder + Claude (dev lead)
**Company:** Logistic (cabinet_id: 070196, webhook token: XcQr9r...)
**Operator:** SIP extension 201
**Destination:** +998948867473

---

## Event Types Reference

| Event | Name | When Fired | Sets State? |
|---|---|---|---|
| 1 | Call Start | Call initiated / ringing begins | No (we set RINGING) |
| 2 | Hangup | Call ended (final) | Yes — terminal state |
| 3 | Answer | Call answered | No (we don't update state) |
| 4 | Transfer Hangup | Transfer leg ended | No (not yet observed) |

---

## Test Results — All Scenarios

### Scenario 1: SIP /number — Failed Instantly (calls #83, #84)

**How:** Called from MicroSIP ext 201 → +998948867473, call dropped in 2-6 seconds.

**Events received:**
```
Event 1 (call start)   → timestamp=T
Event 1 (duplicate)    → timestamp=T+1  (identical except 1s offset)
Event 2 (hangup)       → status=NOANSWER, call_start=T, call_end=T+6, no call_answer
```

**Key fields on Event 2:**
- `status`: NOANSWER
- `call_answer_timestamp`: not present
- `last_called`: 201
- `call_record_link`: present (even for failed calls)

**DB result:**
- state=NOANSWER, billing_sec=NULL, call_answer_timestamp=NULL
- duration: 0 (correct — never answered)

---

### Scenario 2: SIP /number — Answered (calls #86, #88)

**How:** Called from MicroSIP ext 201 → +998948867473, callee picked up, talked briefly, hung up.

**Events received:**
```
Event 1 (call start)   → timestamp=T
Event 1 (duplicate)    → timestamp=T+1
Event 3 (answer)       → timestamp=T+10  (callee picked up)
Event 2 (hangup)       → status=ANSWER, call_start=T, call_answer=T+10, call_end=T+18
```

**Key fields on Event 3:**
- No `status` field
- `timestamp` = actual answer time (reliable)
- `last_called`: empty
- `channel`: changes from `SIP/070196100003-xxx` to `SIP/070196998783337125-xxx`

**Key fields on Event 2:**
- `status`: ANSWER
- `call_answer_timestamp`: matches Event 3 timestamp (reliable for SIP calls)
- `last_called`: 201
- `call_record_link`: present
- `billing_sec`: NOT sent by Sipuni (always absent)

**DB result:**
- state=ANSWER, billing_sec=NULL
- call_answer_timestamp set correctly
- duration computed from: call_end - call_answer = talk time

---

### Scenario 3: SIP /number — Callee Rejected (call #90)

**How:** Called from MicroSIP ext 201 → +998948867473, callee rejected the call after 19s of ringing.

**Events received:**
```
Event 1 (call start)   → timestamp=T
Event 1 (duplicate)    → timestamp=T+1
Event 2 (hangup)       → status=NOANSWER, call_start=T, call_end=T+19, no call_answer
```

**Identical to Scenario 1** (failed instantly) except longer ring time. Sipuni does NOT distinguish between "callee rejected" and "callee didn't answer".

---

### Scenario 4: SIP /number — Caller Hung Up in MicroSIP (call #91)

**How:** Called from MicroSIP ext 201 → +998948867473, caller ended the call after 8s before callee answered.

**Events received:**
```
Event 1 (call start)   → timestamp=T
Event 1 (duplicate)    → timestamp=T+1
Event 2 (hangup)       → status=NOANSWER, call_start=T, call_end=T+8, no call_answer
```

**Identical to Scenarios 1 and 3.** Sipuni does NOT distinguish who hung up. All unanswered = NOANSWER.

---

### Scenario 5: SIP /number — Event 2 Never Arrived (call #85)

**How:** Called from MicroSIP ext 201 → +998948867473, callee answered, but Sipuni never sent event 2.

**Events received:**
```
Event 1 (call start)   → timestamp=T
Event 1 (duplicate)    → timestamp=T+1
Event 3 (answer)       → timestamp=T+10
--- Event 2 NEVER arrived ---
```

**DB result:**
- state=RINGING (stuck forever)
- call_answer_timestamp set (proves it was answered)
- call_end_timestamp=NULL

**Bug:** Our code doesn't update state on event 3 (comment says "let event 2 set final state"). When event 2 never comes, call is stuck at RINGING permanently.

**Fix needed:** Event 3 handler should set state=ANSWER as safety net.

---

### Scenario 6: External /external — No Answer (call #94)

**How:** External call from +998946963233 → +998948867473 via operator 201. Callee didn't answer.

**Events received:**
```
Event 1 (call start)   → timestamp=T, channel=Local/998946963233@vatsout-xxx
Event 2 (hangup)       → status=NOANSWER, call_start=T, call_answer=T (!!), call_end=T+6
```

**Key differences from SIP /number:**
- **Single Event 1** (no duplicate)
- **`call_answer_timestamp` = `call_start_timestamp`** — WRONG. Sipuni sets answer=start even on NOANSWER for external calls. This is a Sipuni bug/quirk.
- **`channel` contains FROM number:** `Local/998946963233@vatsout-xxx` — the 3233 number only appears here
- **`pbxdstnum`** has no `+` prefix (vs SIP calls which include `+`)
- **`src_num`** still shows operator (070196201), not the FROM number

**DB result:**
- phone_1=+998946963233 (correctly extracted from channel)
- phone_2=+998948867473
- call_answer_timestamp=call_start_timestamp (wrong — fake answer time)
- **Bug:** Our duration calculation computes end-answer = 6s for a never-answered call

**Fix needed:** Only compute duration when `state == 'ANSWER'`, or validate `call_answer_timestamp > call_start_timestamp`.

---

### Scenario 7: External /external — Answered (call #96)

**How:** External call from +998946963233 → +998948867473 via operator 201. Callee answered, talked briefly.

**Events received:**
```
Event 1 (call start)   → timestamp=T, channel=Local/998946963233@vatsout-xxx
Event 3 (answer)       → timestamp=T+12
Event 2 (hangup)       → status=ANSWER, call_start=T, call_answer=T-1 (!!), call_end=T+19
```

**Key findings:**
- **`call_answer_timestamp` = T-1** — 1 second BEFORE call_start. Broken for external calls.
- **Event 3 timestamp (T+12) is the reliable answer time**
- Real talk duration: call_end - event3_timestamp = 19-12 = 7 seconds
- `call_answer_timestamp` from event 2 is useless for external calls

**DB result:**
- state=ANSWER
- call_answer_timestamp=1774376481 (before start 1774376482 — impossible)

---

### Scenario 8: External /external — Caller Hung Up First Leg (call #98)

**How:** External call from +998946963233 → +998948867473 via operator 201. First phone (3233) rang, user picked up 3233, but hung up 3233 before 7473 answered.

**Events received:**
```
--- NONE ---
```

**Zero webhook events.** Sipuni only fires webhooks when the call reaches the destination's PBX channel. If the first leg (caller) drops before the second leg connects, no events.

**DB result:**
- Row created by API call with phone_1, phone_2 but no state, no direction, no timestamps
- Orphan record that will never be updated

---

### Scenario 9: SIP /number via Mini App — MicroSIP Not Registered (calls #92, #93)

**How:** Used Mini App dialer numpad to call via /number endpoint. MicroSIP app was not actively connected.

**Events received:**
```
--- NONE ---
```

**Sipuni accepted the API request (HTTP 200) but couldn't reach the SIP extension.** No call was established, no webhooks fired.

**DB result:**
- Row created by API with phone_1=201, phone_2=+998948867473
- No state, direction, timestamps — orphan record

---

## Not Yet Tested

| Scenario | Description |
|---|---|
| Inbound — answered | External caller → Sipuni number → operator picks up |
| Inbound — missed | External caller → Sipuni number → no one picks up |
| Inbound — rejected by operator | External caller → operator rejects in MicroSIP |
| Transfer (event 4) | Answer call, then transfer to another extension |
| IVR /tree call | Call via tree scheme |
| External — callee hangs up first | Both legs connect, callee ends the call |

We have inbound NOANSWER data from real production calls (#82, #87, #95 — all Входящая/NOANSWER) but haven't captured a clean inbound answered event flow yet.

---

## Summary of Sipuni Webhook Behavior

### Event Delivery Pattern
- **SIP /number calls:** Always 2x Event 1 (duplicate, 1s apart), then Event 3 (if answered), then Event 2
- **External /external calls:** Single Event 1, then Event 3 (if answered), then Event 2
- **Event 2 is not guaranteed** — can be lost (Scenario 5), leaving call stuck at RINGING
- **No events at all** when first leg drops before second leg connects, or SIP extension unreachable

### Status Codes from Sipuni
- `ANSWER` — call connected and talked
- `NOANSWER` — covers ALL unanswered scenarios (no pickup, rejected, caller hung up, busy)
- `BUSY`, `CANCEL`, `CONGESTION`, `CHANUNAVAIL` — defined in our enum but never observed in testing
- Sipuni does NOT distinguish who hung up on unanswered calls

### Timestamp Reliability

| Field | SIP /number | External /external |
|---|---|---|
| `call_start_timestamp` | Reliable | Reliable |
| `call_end_timestamp` | Reliable | Reliable |
| `call_answer_timestamp` (evt 2) | Reliable | **BROKEN** (= start on NOANSWER, < start on ANSWER) |
| Event 3 `timestamp` | Reliable | Reliable |
| `billing_sec` | Never sent | Never sent |

### `billing_sec` Is Never Sent
Sipuni does not include `billing_sec` in any webhook event. Duration must always be computed from timestamps: `call_end_timestamp - call_answer_timestamp` (for SIP) or `call_end_timestamp - event3_timestamp` (for external).

### Phone Number Extraction
- **SIP calls:** `src_num` = cabinet_id + extension (e.g., `070196201` → strip prefix → `201`)
- **External calls:** `src_num` = cabinet_id + extension (same). FROM number is only in `channel` field: `Local/998946963233@vatsout-xxx`
- **`pbxdstnum`:** SIP calls include `+` prefix, external calls don't
- **Inbound calls:** `src_num` = external caller (no cabinet prefix), `dst_num` may have `_id` suffix

### Recording URLs
- Sipuni sends `call_record_link` on ALL calls — even NOANSWER and failed calls
- Recording content may be empty/silent for failed calls
- Sipuni throttles recording downloads from datacenter IPs (~1KB/s)
- Must use residential proxy for downloads (fix deployed 2026-03-24)

---

## Bugs Found / Fixes Needed

### 1. Event 3 should set state=ANSWER (P1)
**File:** `sipuni_provider.py:_handle_answer()`
**Current:** Comment says "Don't set state — let event 2 set the final state"
**Problem:** If event 2 never arrives (Scenario 5), call is stuck at RINGING forever
**Fix:** Add `data["state"] = "ANSWER"` in `_handle_answer()`. Event 2 overwrites if it arrives.

### 2. External call answer_timestamp is broken (P2)
**File:** `calls_enhanced.py` duration computation, `schemas/call/__init__.py`
**Problem:** For external calls, `call_answer_timestamp` equals `call_start_timestamp` on NOANSWER, or is BEFORE start on ANSWER
**Fix:** Only compute duration when `state == 'ANSWER'`. Validate `call_answer > call_start` before using. Prefer event 3 timestamp when available.

### 3. Orphan call records from failed initiations (P3)
**Problem:** /number and /external create DB rows before Sipuni confirms. If call never connects, rows stay with NULL state forever.
**Fix:** Background task to mark NULL-state calls older than 5 minutes as FAILED/EXPIRED.

### 4. Operator not resolved from last_called (P3)
**Problem:** `last_called=201` doesn't get matched to user with SIP extension 201
**File:** `webhooks.py` operator resolution logic

### 5. Outbound notification shows "Unknown" instead of destination (P2)
**Problem:** Telegram notification for outbound calls shows "Unknown" because phone_1=201 (SIP ext)
**Fix:** For outbound calls, display phone_2 (destination) in notifications, not phone_1
