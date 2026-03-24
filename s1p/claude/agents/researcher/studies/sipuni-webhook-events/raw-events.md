# Sipuni Raw Webhook Events — Captured 2026-03-24

All events from Logistic company (cabinet_id: 070196).
Operator: SIP extension 201. Destination: +998948867473.

---

## #83 — SIP /number, NOANSWER (failed instantly, 6s)

```
EVENT 1: call_id=1774371532.533505 event=1 src_type=2 src_num=018315201 short_src_num=201
  dst_type=1 dst_num=998948867473 pbxdstnum=+998948867473 timestamp=1774371532
  treeName=Исходящая treeNumber=000-913901 channel=SIP/018315100002-000471a4

EVENT 2: call_id=1774371532.533505 event=2 status=NOANSWER
  call_start_timestamp=1774371532 timestamp=1774371538
  last_called=201 call_record_link=https://sipuni.com/api/crm/record?id=1774371532.533505&hash=f5d4eeb5...
  channel=SIP/018315100002-000471a4
```

Note: This call used Samyy Tech company (018315), not Logistic.

---

## #85 — SIP /number, STUCK AT RINGING (event 2 never arrived)

```
EVENT 1a: call_id=1774372146.533584 event=1 src_type=2 src_num=070196201 short_src_num=201
  dst_type=1 dst_num=998948867473 pbxdstnum=+998948867473 timestamp=1774372146
  treeName=Исходящая treeNumber=000-1734445 channel=SIP/070196100003-000471cc user_id=070196

EVENT 1b: call_id=1774372146.533584 event=1 timestamp=1774372147
  (identical to 1a except timestamp +1s)

EVENT 3: call_id=1774372146.533584 event=3 timestamp=1774372157
  channel=SIP/070196998783337125-000471cd
  (no status, no last_called, no call_record_link)

--- EVENT 2 NEVER ARRIVED ---
```

DB: state=RINGING, call_answer_timestamp=1774372157, call_end_timestamp=NULL

---

## #86 — SIP /number, ANSWER (8s talk)

```
EVENT 1a: call_id=1774372584.533596 event=1 src_type=2 src_num=070196201 short_src_num=201
  dst_type=1 dst_num=998948867473 pbxdstnum=+998948867473 timestamp=1774372584
  treeName=Исходящая treeNumber=000-1734445 channel=SIP/070196100003-000471d3 user_id=070196

EVENT 1b: call_id=1774372584.533596 event=1 timestamp=1774372585
  (identical except timestamp +1s)

EVENT 3: call_id=1774372584.533596 event=3 timestamp=1774372594
  channel=SIP/070196998783337125-000471d4

EVENT 2: call_id=1774372584.533596 event=2 status=ANSWER
  call_start_timestamp=1774372584 call_answer_timestamp=1774372594 timestamp=1774372602
  last_called=201 call_record_link=https://sipuni.com/api/crm/record?id=1774372584.533596&hash=93247a3b...
  channel=SIP/070196998783337125-000471d4
```

DB: state=ANSWER, answer=594, end=602, duration=8s

---

## #90 — SIP /number, NOANSWER (callee rejected after 19s)

```
EVENT 1a: call_id=1774375311.534654 event=1 src_type=2 src_num=070196201 short_src_num=201
  dst_type=1 dst_num=998948867473 pbxdstnum=+998948867473 timestamp=1774375312
  treeName=Исходящая treeNumber=000-1734445 channel=SIP/070196100003-000473e4 user_id=070196

EVENT 1b: call_id=1774375311.534654 event=1 timestamp=1774375313
  (identical except timestamp +1s)

EVENT 2: call_id=1774375311.534654 event=2 status=NOANSWER
  call_start_timestamp=1774375312 timestamp=1774375331
  last_called=201 call_record_link=https://sipuni.com/api/crm/record?id=1774375311.534654&hash=b5fd52ec...
  channel=SIP/070196100003-000473e4
```

---

## #91 — SIP /number, NOANSWER (caller hung up after 8s)

```
EVENT 1a: call_id=1774375626.534741 event=1 src_type=2 src_num=070196201 short_src_num=201
  dst_type=1 dst_num=998948867473 pbxdstnum=+998948867473 timestamp=1774375626
  treeName=Исходящая treeNumber=000-1734445 channel=SIP/070196100003-0004740c user_id=070196

EVENT 1b: call_id=1774375626.534741 event=1 timestamp=1774375627
  (identical except timestamp +1s)

EVENT 2: call_id=1774375626.534741 event=2 status=NOANSWER
  call_start_timestamp=1774375626 timestamp=1774375634
  last_called=201 call_record_link=https://sipuni.com/api/crm/record?id=1774375626.534741&hash=bd728195...
  channel=SIP/070196100003-0004740c
```

Identical pattern to #90 (callee rejected). Cannot distinguish who hung up.

---

## #94 — External /external, NOANSWER (3233→7473 via 201)

```
EVENT 1: call_id=1774376114.534794 event=1 src_type=2 src_num=070196201 short_src_num=201
  dst_type=1 dst_num=998948867473 short_dst_num=998948867473 pbxdstnum=998948867473
  timestamp=1774376127 treeName=Исходящая treeNumber=000-1734445
  channel=Local/998946963233@vatsout-000022c5;1 user_id=070196

EVENT 2: call_id=1774376114.534794 event=2 status=NOANSWER
  call_start_timestamp=1774376127 call_answer_timestamp=1774376127 timestamp=1774376133
  last_called=201 call_record_link=https://sipuni.com/api/crm/record?id=1774376114.534794&hash=ddc97745...
  channel=SIP/070196998783337125-00047428
```

Key differences from SIP /number:
- Single event 1 (no duplicate)
- call_answer_timestamp = call_start_timestamp (WRONG — not actually answered)
- channel contains FROM number: Local/998946963233@vatsout
- pbxdstnum has no + prefix

---

## #96 — External /external, ANSWER (3233→7473 via 201, 7s talk)

```
EVENT 1: call_id=1774376474.534867 event=1 src_type=2 src_num=070196201 short_src_num=201
  dst_type=1 dst_num=998948867473 short_dst_num=998948867473 pbxdstnum=998948867473
  timestamp=1774376482 treeName=Исходящая treeNumber=000-1734445
  channel=Local/998946963233@vatsout-000022c6;1 user_id=070196

EVENT 3: call_id=1774376474.534867 event=3 timestamp=1774376494
  channel=SIP/070196998783337125-00047454

EVENT 2: call_id=1774376474.534867 event=2 status=ANSWER
  call_start_timestamp=1774376482 call_answer_timestamp=1774376481 timestamp=1774376501
  last_called=201 call_record_link=https://sipuni.com/api/crm/record?id=1774376474.534867&hash=13820d20...
  channel=SIP/070196998783337125-00047453
```

Key: call_answer_timestamp (481) is BEFORE call_start (482) — broken.
Event 3 timestamp (494) is the reliable answer time.
Real talk: 501 - 494 = 7 seconds.

---

## #98 — External /external, caller hung up first leg (zero events)

API created row with phone_1=+998946963233, phone_2=+998948867473.
First phone rang and was picked up. Caller hung up before second phone answered.

```
--- NO WEBHOOK EVENTS ---
```

Sipuni only fires webhooks when call reaches destination PBX channel.

---

## #92, #93 — SIP /number via Mini App, MicroSIP offline (zero events)

API requests returned HTTP 200 (Sipuni accepted) but MicroSIP wasn't registered.

```
--- NO WEBHOOK EVENTS ---
```

Sipuni can't reach the SIP extension → no call → no webhooks.

---

## Inbound NOANSWER (real production calls #82, #87, #95)

Example from #87 (real webhook URL captured):

```
EVENT 2: call_id=1774373436.533962 event=2 src_type=1 src_num=951218115 short_src_num=951218115
  dst_type=1 dst_num=998783337125_id265648 short_dst_num=998783337125_id265648
  pbxdstnum=998783337125 timestamp=1774373455 status=NOANSWER
  call_start_timestamp=1774373436 treeName=Входящая treeNumber=000-1734442
  channel=SIP/047593496447128812-00047440 user_id=070196
  call_record_link=https://sipuni.com/api/crm/record?id=...
```

Key: for inbound, src_num is the external caller (no cabinet prefix).
dst_num has _id suffix (stripped by our normalizer).
No event 1 captured in logs (may have been outside our log window).
