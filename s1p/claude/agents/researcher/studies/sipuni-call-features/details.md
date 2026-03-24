# Sipuni Call Features — Full Research Details

**Date:** 2026-03-19
**Researcher:** Claude (dev lead research session)
**Sources:** Sipuni dashboard exploration (Playwright), API docs (web), codebase audit
**Account:** Cabinet #018315, Professional plan, Contract СФ-26275/2023

---

## Table of Contents

1. [Complete API Surface](#1-complete-api-surface)
2. [What S1P Currently Implements](#2-what-s1p-currently-implements)
3. [Gap Analysis — APIs Not Used](#3-gap-analysis)
4. [Dashboard Exploration Findings](#4-dashboard-exploration)
5. [Data Fields We're Missing](#5-data-fields)
6. [Account & Infrastructure Details](#6-account-details)
7. [Integration Architecture Notes](#7-architecture-notes)
8. [Sources](#8-sources)

---

## 1. Complete API Surface

### 1.1 Authentication Systems

**Legacy/Main API** (sipuni.com/api/...):
- MD5 hash-based signatures
- Parameters concatenated with `+` delimiter in alphabetical order
- Secret key appended last, then MD5 hashed
- Example: `MD5(param1+param2+...+user+secret_key)`

**Calltracking API** (api.sipuni.com/v1/...):
- Token-based: `Authorization: Token {apiKey}`
- Separate from the main integration key

### 1.2 Statistics Endpoints

#### GET Call Records (CSV)
```
POST https://sipuni.com/api/statistic/export
```
Parameters (ALL available — from dashboard):
| Param | Description | Values |
|---|---|---|
| user | Account number | 018315 |
| from | Date from | DD.MM.YYYY |
| to | Date to | DD.MM.YYYY |
| timeFrom | Time from | HH:MM |
| timeTo | Time to | HH:MM |
| type | Call type | 0=All, 1=Incoming, 2=Outgoing, 3=Internal |
| state | Call status | 0=All, 1=Missed, 2=Answered |
| tree | Scheme number | 000-913898 (Входящая), 000-913901 (Исходящая), 000-3062543, 000-3062548 |
| showTreeId | Show scheme ID | 0/1 |
| fromNumber | Source number filter | Phone or extension |
| toNumber | Destination number filter | Phone or extension |
| numbersRinged | Show numbers that rang | 0/1 |
| numbersInvolved | Show all participants | 0/1 |
| names | Show employee names | 0/1 |
| outgoingLine | Show outgoing line | 0/1 |
| toAnswer | Who answered filter | Phone or extension |
| anonymous | Anonymous calls | 0=show all, 1=hide anonymous |
| crmLinks | Show CRM entity links | 0/1 |
| firstTime | New customers only | 0/1 |
| dtmfUserAnswer | Show DTMF input | 0/1 |
| Rating | Filter by rating | 0-9, comma-separated (0,2,5), range (2-7) |
| hangupinitor | Call termination initiator | 1=show, 2=operator, 3=client, 4=busy, 5=no answer, 6=offline, 7=carrier error |
| ignoreSpecChar | Ignore semicolons | 0/1 |
| hash | MD5 signature | See hash order below |

**Hash parameter order:** anonymous, crmLinks, dtmfUserAnswer, firstTime, from, fromNumber, hangupinitor, ignoreSpecChar, names, numbersInvolved, numbersRinged, outgoingLine, rating, showTreeId, state, timeFrom, timeTo, to, toAnswer, toNumber, tree, type, user, secret

#### GET All Records (Paginated)
```
POST https://sipuni.com/api/statistic/export/all
```
Parameters: user, limit (1-200000), order (asc/desc), page, hash

#### Download Recording
```
POST https://sipuni.com/api/statistic/record
```
Parameters: user, id (recording ID from CSV), hash
Returns: MP3 audio file

#### Get Operator Statuses
```
POST https://sipuni.com/api/statistic/operators
```
Parameters: user, hash
Returns: CSV with employee list and online/offline/DND status

### 1.3 Call Creation Endpoints

#### Call Number (Click-to-Call)
```
POST https://sipuni.com/api/callback/call_number
```
Parameters: user, phone, sipnumber, reverse (0/1), antiaon (0/1), hash
Hash order: antiaon, phone, reverse, sipnumber, user, secret

#### Call via IVR Scheme
```
POST https://sipuni.com/api/callback/call_tree
```
Parameters: user, phone, sipnumber, tree, reverse, callAttemptTime (min 30s), hash
Hash order: callAttemptTime, phone, reverse, sipnumber, tree, user, secret

Available schemes (from dashboard):
- 000-913898 — Входящая (Incoming)
- 000-913901 — Исходящая (Outgoing)
- 000-3062543 — Новая схема
- 000-3062548 — Новая схема

#### Bridge Two External Numbers
```
POST https://sipuni.com/api/callback/call_external
```
Parameters: user, phoneFrom, phoneTo, sipnumber, sipnumber2, hash
Hash order: phoneFrom, phoneTo, sipnumber, sipnumber2, user, secret

#### Cancel Callback
```
POST https://sipuni.com/api/callback/cancel
```
Parameters: user, callbackId, hash
Hash order: callbackId, user, secret

### 1.4 PBX Event APIs

#### Terminate Active Call (NEW - we don't use this)
```
POST https://sipuni.com/api/events/call/hangup
```
Parameters: user, callId, hash
Hash order: callId, user, secret

#### Spy Mode / Whispering (NEW - we don't use this)
```
POST https://sipuni.com/api/events/call/spymode
```
Parameters:
| Param | Description | Values |
|---|---|---|
| user | Account number | 018315 |
| sipnumber | Operator to connect to | 201, 202, 203, 204, 205 |
| callId | Active call ID | From event webhook |
| whisper | Mode | 0=Silent listening, 1=Coaching (agent hears you) |
| hash | Signature | |

Hash order: callId, sipnumber, user, whisper, secret

### 1.5 SMS Gateway (NEW - we don't use this)

Route all Sipuni SMS through a custom gateway. Configure URL in settings, then Sipuni sends POST requests:

| Param | Description |
|---|---|
| number | Phone number (79161234567) |
| sender | Sender name (default: Sipuni.com) |
| text | SMS message text |

Gateway must respond HTTP 200 on success.

### 1.6 Helper/Auxiliary APIs (NEW - we don't use these)

#### Get Schema List
```
POST https://sipuni.com/api/schema/list
```
Parameters: user, type (''=all, 0=incoming, 1=outgoing), hash
Hash order: type, user, secret

#### Set User Status
```
GET https://sipuni.com/api/workplace/setUserStatus
```
Parameters: sipnumber, status, user, hash
Statuses:
- 0 = Free (Свободен)
- 1 = Break (Перерыв)
- 2 = Auto-dialing (Автообзвон)
- 3 = Manual dialing (Ручной обзвон)
- 4 = Away (Не на месте)

Hash order: sipnumber, status, user, secret

### 1.7 WebSocket Events (NEW - we don't use this)

Connection: `wss://wss.sipuni.com/api`
Authentication code available in dashboard settings.

Real-time PBX events over persistent WebSocket connection. Same event format as HTTP webhooks but live.

### 1.8 Webhook Events (Full Detail)

#### Universal Fields (ALL events):
| Field | Type | Description |
|---|---|---|
| event | int | 1=Call, 2=Hang-up, 3=Answer, 4=Transfer hang-up |
| call_id | string | Unique call ID (preserved across transfers) |
| src_num | string | Caller number/address |
| src_type | int | 1=external, 2=internal |
| dst_num | string | Destination number |
| dst_type | int | 1=external, 2=internal |
| timestamp | int | Unix timestamp (UTC) |
| short_src_num | string | Short internal number |
| short_dst_num | string | Short internal number |
| user_id | string | Account number |
| transfer_from | string | Employee executing transfer |
| last_called | string | Last employee in routing chain |
| treeName | string | Routing scheme name |
| treeNumber | string | Routing scheme number |

#### Event 1 (Call Start) — additional:
- `is_inner_call` — 1 if internal call
- `roistat` — Roistat visitor ID
- `roistat_number` — tracking number called
- `roistat_market` — source/campaign label

#### Event 2 (Hang-up) — additional:
- `status` — ANSWER, BUSY, NOANSWER, CANCEL, CONGESTION, CHANUNAVAIL
- `call_start_timestamp` — when call began
- `call_answer_timestamp` — when answered (0 if not)
- `call_record_link` — recording URL

#### Event 3 (Answer):
- No additional fields beyond universal

#### Event 4 (Transfer Hang-up):
- Same as event 2 (status, timestamps, recording)

**We currently only process event=2. Events 1, 3, 4 are ignored.**

---

## 2. What S1P Currently Implements

### Backend Code Audit

| Component | File | What It Does |
|---|---|---|
| SipuniProvider | `source/utils/services/telephony/sipuni_provider.py` | MD5 auth, webhook handling, call_number/tree/external/cancel |
| SipuniAsyncClient | `source/utils/services/sipuni_client.py` | Dashboard scraping for one-click setup |
| Sipuni Setup | `source/api/v1/routers/company/sipuni_setup.py` | 5 setup endpoints |
| Call Endpoints | `source/api/v1/routers/company/calls/sipuni.py` | 4 call-making endpoints |
| Webhooks | `source/api/v1/routers/company/webhooks.py` | Webhook ingestion |
| Statistics | `source/api/v1/services/sipuni/statistics.py` | Call stats aggregation (LEGACY model) |

### Webhook Processing (current):
1. Only event=2 (hang-up) is processed
2. Extracts: call_id, src_num, dst_num, status, call_start_timestamp, timestamp, call_record_link
3. Creates/updates CallEvent
4. Fires outbound webhook for call.completed or call.missed
5. Sends Telegram notification

### What We Store per Call:
- provider_call_id
- phone_1, phone_2
- state (ANSWER, NOANSWER, etc.)
- call_start_timestamp, call_end_timestamp
- record_url
- direction (inbound/outbound/internal)

---

## 3. Gap Analysis

### APIs Not Implemented

| API | Endpoint | Priority | Why |
|---|---|---|---|
| Call Hangup | `/api/events/call/hangup` | P1 | End calls from CRM UI |
| Spy/Whisper | `/api/events/call/spymode` | P1 | Premium call center feature |
| Set User Status | `/api/workplace/setUserStatus` | P1 | Operator availability management |
| Get Operators | `/api/statistic/operators` | P1 | Show who's online/available |
| Schema List | `/api/schema/list` | P2 | Analytics per routing scheme |
| WebSocket Events | `wss://wss.sipuni.com/api` | P1 | Real-time call tracking |
| SMS Gateway | Custom URL | P3 | SMS from CRM |
| Recording Download | `/api/statistic/record` | P2 | Download recordings by ID |
| Export All (paginated) | `/api/statistic/export/all` | P2 | Bulk call history sync |

### Webhook Events Not Processed

| Event | What | Priority |
|---|---|---|
| Event 1 (Call Start) | Call initiation — enables live call tracking | P1 |
| Event 3 (Answer) | Call answered — enables ring time calculation | P1 |
| Event 4 (Transfer Hangup) | Transfer completed — enables transfer tracking | P2 |

### Statistics Parameters Not Used

| Parameter | What | Priority |
|---|---|---|
| hangupinitor | Who ended the call | P1 |
| firstTime | First-time caller flag | P1 |
| Rating | Post-call quality score | P2 |
| numbersRinged | Which extensions rang | P2 |
| numbersInvolved | Who participated | P2 |
| outgoingLine | Which line was used | P3 |
| dtmfUserAnswer | IVR key presses | P3 |
| crmLinks | CRM entity links (amoCRM) | P3 |

### Webhook Fields Not Extracted

| Field | What | Priority |
|---|---|---|
| treeName / treeNumber | Routing scheme info | P2 |
| transfer_from | Who transferred | P2 |
| last_called | Last agent in chain | P2 |
| call_answer_timestamp | When answered | P1 |
| short_src_num / short_dst_num | Clean extension numbers | P1 |
| roistat / roistat_number | Call tracking data | P3 |

---

## 4. Dashboard Exploration Findings

### Old Cabinet (sipuni.com) — Settings Sidebar

Full settings menu discovered:
1. Основные настройки (General settings)
2. Оформление документов (Document formatting)
3. Контактные данные (Contact details)
4. Мои номера (My numbers)
5. Делегирование (Delegation — share account access)
6. Интеграции (Integrations)
7. API
8. Запись звонков (Call recording settings)
9. Статистика для сотрудников (Employee statistics access)
10. Оценка качества обслуживания (Service quality rating)
11. Пропущенный звонок (Missed call handling)
12. Оповещения о пропущенных звонках (Missed call notifications)
13. Голосовой звонок (Voice call / TTS)
14. SIM-карты (SIM cards)
15. Оповещения о балансе (Balance alerts)
16. Логины и пароли (Logins and passwords)
17. Дополнительные настройки (Additional settings)
18. Автообзвон (Auto-dialing)
19. Метка (Labels — for call categorization)
20. Тег (Tags)
21. Настройка очередей (Queue configuration)
22. Выбор линии для сотрудника (Line selection per employee)
23. Настройка статусов оператора (Operator status configuration)

### Old Cabinet — API Integration Tabs

The integration page has these API category tabs:
- **Статистика** — call history export, recording download, operator status
- **Звонок на номер** — click-to-call, call via scheme, bridge calls, cancel
- **События на АТС** — PBX event webhooks (HTTP + WebSocket), call hangup
- **Голосовой звонок** — TTS voice calls (call_voice API — not visible in new cabinet)
- **SMS** — SMS gateway routing
- **Суфлирование** — Spy mode / call whispering
- **Анализ речи** — Speech analytics (partner service)
- **Вспомогательные** — Schema list, set user status
- **Автообзвон** — Auto-dialing campaign management

### New Cabinet (lk.sipuni.com) — Modern UI

Sections:
- **Номера** — Phone number management (SIP connect)
- **Конструктор** — Visual call routing builder
- **Пользователи** — Employee management with device credentials
- **Оплата и документы** — Billing, licenses
- **Статистика** — Three tabs: By Calls, By Users, By Numbers
- **Интеграции** — CRM (amoCRM connected, Bitrix24, yclients, Юздеск) + Analytics (Roistat, webhooks, Calltouch, TQM, **Yandex SpeechSense**)
- **Автообзвон** — Auto-dialing campaigns
- **Мои данные** — Owner profile + accountant email
- **Настройки** — Call recording, balance alerts, timezone, IP restrictions, 2FA
- **Настройки кол-центра** — Active call threshold, SL time, ACW time, extended statuses

### Constructor Elements (Routing Components)

Available building blocks:
- Входящие/Исходящие схемы (Incoming/Outgoing schemes)
- Сотрудники (Employees)
- Входящие/Исходящие линии (Incoming/Outgoing lines)
- Отделы (Departments)
- Врем. диапазоны (Time ranges / business hours)
- Аудиофайлы (Audio files for IVR/hold music)
- Очереди звонков (Call queues)
- Голосовые меню (Voice menus / IVR)
- Шаблоны (Templates)
- Голосовая почта (Voicemail)
- Режимы работы (Operating modes)
- Списки линий (Line lists)
- Hotdesk

### Statistics Page (New Cabinet) — Column Details

**Stats by Calls** table columns:
- Тип (Type) — incoming/outgoing/internal
- Время (Time)
- Схема (Scheme)
- Откуда (From)
- Статус (Status)
- Кто ответил (Who answered)
- Длительность (Duration)
- Длительность разговора (Talk duration)
- Время ответа (Answer time)
- Комментарий (Comment)
- Тeги (Tags)

**Stats by Numbers** columns:
- Номер (Number)
- Всего звонков (Total calls)
- Отвеченных звонков (Answered calls)
- Активных звонков (Active calls)
- Общая продолжительность звонков (Total call duration)
- Общая продолжительность разговора (Total talk duration)
- Средняя продолжительность звонков (Avg call duration)
- Средняя продолжительность разговора (Avg talk duration)
- Среднее время ответа (Avg answer time)
- Процент отвеченных звонков (Answer rate)
- Процент активных (Active rate)

**Stats by Users:**
- Per-user performance metrics
- Activity history timeline

---

## 5. Data Fields We're Missing in CallEvent Model

### Currently stored:
```
provider_call_id, phone_1, phone_2, state,
call_start_timestamp, call_end_timestamp,
record_url, direction
```

### Should add:
```
call_answer_timestamp    — when answered (0 if not) → enables wait time calculation
hangup_initiator         — who ended: operator/client/busy/noanswer/offline/error
is_first_time            — new caller flag
rating                   — post-call quality score (0-9)
scheme_name              — which routing scheme handled the call
scheme_number            — scheme ID
transfer_from            — who transferred
numbers_ringed           — which extensions rang before answer
numbers_involved         — all participants
outgoing_line            — which line was used for outgoing
dtmf_input               — customer IVR key presses
short_src_num            — clean extension number for source
short_dst_num            — clean extension number for destination
```

---

## 6. Account Details

| Property | Value |
|---|---|
| Cabinet Number | 018315 |
| Contract | СФ-26275/2023 |
| Plan | Профессиональная (Professional — highest tier) |
| Valid Until | 2026-05-26 |
| Licensed Users | 5 |
| SIP Server | voip.uz.sipuni.com |

### Employees

| Ext | Name | SIP Login | SIP Password | App Auth Key |
|---|---|---|---|---|
| 201 | Алекс | 018315100002 | QOpLRMXA | e27e554bc057b19b195d4bfe6e03753c |
| 202 | Адхам | 018315100005 | n0k5DU3T | 1dd790b9a5a470c91819e34bfc8e0a0e |
| 203 | Zamira | 018315100008 | 5FGJBnHJ | f453684e4361a7a36866ddd71ae79925 |
| 204 | Fathullo | 018315100011 | 9hAkhbkb | 245d67bad1561ed3302aeaeed9ff4241 |
| 205 | Жафар | 018315100014 | QA5yrfE9 | 022b52306b3523559942ee198190f57a |

### Routing Schemes

| ID | Name | Type |
|---|---|---|
| 000-913898 | Входящая | Incoming |
| 000-913901 | Исходящая | Outgoing |
| 000-3062543 | Новая схема | — |
| 000-3062548 | Новая схема | — |

### Available Integrations

**Connected:**
- amoCRM (CRM) — already connected in Sipuni dashboard

**Available to connect:**
- Bitrix24 (ERP & CRM)
- yclients (CRM)
- Юздеск (Customer support automation)
- Roistat (Business analytics)
- webhooks (Variable passing to CRM)
- Calltouch
- TQM
- **Yandex SpeechSense** (Speech recognition)

---

## 7. Architecture Notes

### Two Dashboards
Sipuni is migrating from old cabinet (sipuni.com) to new cabinet (lk.sipuni.com). Many settings pages in the old cabinet now return 404. The API documentation lives in the new cabinet's "Интеграция по API" tab. Some features (like Labels/Метки) only exist in the old cabinet.

### Number Format
Internal numbers come as `<account_number><short_number>`:
- `018315100002` = account 018315, extension 100002
- `short_src_num` / `short_dst_num` contain just `201`, `202`, etc.

### Call ID Persistence
`call_id` is preserved across all events for a single call, including transfers. This is the correlation key for reconstructing call journeys.

### Recording Retention
Professional plan = unlimited retention in WAV format.

### WebSocket vs HTTP Webhooks
Both deliver the same events. WebSocket is real-time persistent connection. HTTP webhooks are POST/GET per event. Can use both simultaneously.

### Rate Limits
Not officially documented. Sipuni API docs say "API is still in testing and expanding."

### Response Format
- Statistics endpoints return CSV (not JSON)
- Call creation endpoints return JSON
- Recording download returns MP3 binary
- Operator status returns CSV

---

## 8. Sources

1. **Sipuni Dashboard** — Live exploration via Playwright (2026-03-19)
   - Old cabinet: sipuni.com/ru_RU/settings/integration
   - New cabinet: lk.sipuni.com/integrations (API tab)
2. **S1P Backend Codebase** — `repos/s1p-backend/source/` (audit 2026-03-19)
3. **Sipuni API Knowledge Base** — https://help.sipuni.com/articles/134-182--api/
4. **Sipuni HTTP Events API** — https://help.sipuni.com/articles/134-182-110/
5. **Official HTTP API PDF** — sipuni.com/idealats/pdf/SIPUNI описание HTTP API.pdf
6. **Sipuni PHP API (GitHub)** — github.com/sipuni/sipuni-api-php
7. **Sipuni Python Wrapper** — github.com/bzdvdn/sipuni-api-wrapper
8. **Sipuni Go Client** — github.com/bzdvdn/sipuni-go
9. **Sipuni PBX Functions** — sipuni.com/ru_RU/virtual-pbx/functions
10. **Sipuni Pricing** — sipuni.com/price/
